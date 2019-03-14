
<!-- TOC -->

- [ANTLR](#antlr)
    - [语法导入](#语法导入)
    - [非贪婪模式](#非贪婪模式)
    - [标签](#标签)

<!-- /TOC -->

# ANTLR

## 语法导入
可以把语法分为两部分：语法分析器的语法和词法分析器的语法。

eg， 词法规则可以被多重语法共享，单独抽出来：
```
lexer grammar CommonLexerRules;
ID : [a-zA-Z]+;
INT : [0-9]+;
NEWLINE : '\r' ? '\n';
WS : [ \t]+ -> skip;
```

语法规则中可以通过imports引入:

```
grammar LibExpr;
import CommonLexerRules;

prog : stat+ ;

stat : expr NEWLINE
     | ID '=' expr NEWLINE
     | NEWLINE;

expr : expr ('*' | '/') expr
    | expr ('+' | '-') expr
    | INT
    | ID
    | '(' expr ')'
    ;
```

## 非贪婪模式
(...)?、(...)*、(...)+的EBNF自规则是贪婪的，它们会消费尽可能多的输入文本。`.*`能匹配到输入文本的末尾。可以通过在后缀添加'?'的方式使任意以?、*、+结尾的子规则变成非贪婪的。非贪婪模式可以用于词法规则和语法规则。尽量少用非贪婪模式，它增加了识别的复杂度。

ANTLR 4词法规则的匹配过程：

- 首先能匹配能够识别最多输入字符的词法规则。
eg:
```
INT : [0-9]+;
DOT : '.';
```
- 如果多余一条词法规则能匹配相同的输入序列，在语法文件中靠前的有更高的优先级。

- 非贪婪自规则能够匹配满足该规则的最少的字符序列。
输入 '<<foo>>'匹配的是STRING END
```
STRING : '<<' ~'\n'*? '>>';
END: '>>';
```

- 在词法规则中非贪婪匹配子规则之后的所有决策都遵循"最先匹配原则"。

1.对于模式".*?('a'|ab)", 如果输入的是a, 则第一个备选子规则匹配。如果输入的是ab，则第一个备选子规则匹配到‘a’整条rule结束。不会匹配到第二条子规则我。
2.对于模式.*?('ab'|'a')，如果输入的是a，则第一个备选子规则匹配。如果输入的是ab，则第二个自规则匹配。
这样设计是为了性能考虑。

## 标签
ANTLR 支持在分支末尾加标签和在rule中间加标签
- mulDiv标签会生成MulDivContext，visitor和listener可以访问。
- op标签或让op成为Context的一个field，可以在程序中访问。
```
expr:  expr op=('*' | '/') expr     # mulDiv
     | expr op=('+' | '-') expr     # addSub
     | INT                       # int
     | ID                        # id
     | '(' expr ')'              # parens
     ;
```

