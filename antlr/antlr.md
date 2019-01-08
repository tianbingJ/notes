# ANTLR

## 语法导入
可以把语法分为两部分：语法分析器的语法和词法分析器的语法。

eg， 词法规则可以被多重语法共享，单独抽出来：
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

