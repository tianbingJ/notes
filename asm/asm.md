
## ClassReader
解析一个编译后的类（字节码形式的byte array），调用ClassVisitor对象对应的visitXXX方法，ClassVisitor对象通过ClassReader.accept方法的参数传入.Class可以看做是时间的生产者。

## ClassVisitor
代理所有的方法调用给其他的ClassVisitor对象，可以看做是一个Event Filter。

## ClassWriter
ClassWriter继承了ClassVisitor，它生成类的字节码，可以看做是事件的消费者。

## 使用ClassWriter定义的类
如果产生使用类由使用的环境所决定，如果你正在实现一个编译器，类生成则由AST驱动，生成的类则会保存在硬盘上。如果是使用动态代理，则类的生成可以使用ClassLoader实现。
- 可以把类定义保存在.class文件，方便以后使用。
- 定义一个ClassLoader,使用defineClass方法
```
    class MyClassLoader extends ClassLoader {
          public Class defineClass(String name, byte[] b) {
            return defineClass(name, b, 0, b.length);
          }
}
```
使用类:
```
Class c = myClassLoader.defineClass(cName, b);
cName是类internal name， b是byte array.
```
- 使用ClassLoader的findClass方法
```
    class StubClassLoader extends ClassLoader {
        @Override
        protected Class findClass(String name) throws ClassNotFoundException {
          if (name.endsWith("_Stub")) {
            ClassWriter cw = new ClassWriter(0);
            ...
            byte[] b = cw.toByteArray();
            return defineClass(name, b, 0, b.length);
          }
          return super.findClass(name);
        }
    }
```

## 转换类
组合ClassReader, ClassVisitor, ClassWriter. ClassReader读取一个类的定义，ClassVisitor对类的定义进行修改，ClassWriter生成转换后的类定义。
![transform](../resources/image/asm/class_transform.png)
```
byte[] b1 = ...;
ClassWriter cw = new ClassWriter(0);
// cv forwards all events to cw
ClassVisitor cv = new ClassVisitor(ASM4, cw) { };
ClassReader cr = new ClassReader(b1);
cr.accept(cv, 0);
byte[] b2 = cw.toByteArray(); // b2 represents the same class as b1
```

## 删除类成员
对于返回值是空的成员访问，只要不forword method call，即可删除内容。
如下删除内部类、外部类和源文件信息。
```

      public class RemoveDebugAdapter extends ClassVisitor {
        public RemoveDebugAdapter(ClassVisitor cv) {
          super(ASM4, cv);
        }
        @Override
        public void visitSource(String source, String debug) {
        }
        @Override
        public void visitOuterClass(String owner, String name, String desc) {
        }
        @Override
        public void visitInnerClass(String name, String outerName,
            String innerName, int access) {
        }
}
```
上述方法对于字段和方法不起作用，因为visitField和visitMethod必须有返回值。可以通过返回null来删除部分成员。
下面通过方法名和类型描述符删除某个方法。
```
   public class RemoveMethodAdapter extends ClassVisitor {
        private String mName;
        private String mDesc;
        public RemoveMethodAdapter(
            ClassVisitor cv, String mName, String mDesc) {
            super(ASM4, cv);
            this.mName = mName;
            this.mDesc = mDesc;
        }
        @Override
        public MethodVisitor visitMethod(int access, String name,
            String desc, String signature, String[] exceptions) {
          if (name.equals(mName) && desc.equals(mDesc)) {
            // do not delegate to next visitor -> this removes the method
            return null;
}
          return cv.visitMethod(access, name, desc, signature, exceptions);
        }
}
```
## 添加成员
添加Field可以通过调用visitField方法实现，但是visitField的方法调用必须放在合适的位置。比如不同在调用visit()方法的时候调用visitField，因为这样可能导致调用visitField之后调用visitSource、visitAnnotation等方法，这种访问顺序是不合法的。如果添加在visitField或者visitMethod方法中，会导致有几个字段或者有几个方法，就添加了几个字段。更常用的方式是添加在visitEnd方法中，如下：
```

      public class AddFieldAdapter extends ClassVisitor {
        private int fAcc;
        private String fName;
        private String fDesc;
        private boolean isFieldPresent;
        public AddFieldAdapter(ClassVisitor cv, int fAcc, String fName,
            String fDesc) {
          super(ASM4, cv);
          this.fAcc = fAcc;
          this.fName = fName;
          this.fDesc = fDesc;
        }
        @Override
        public FieldVisitor visitField(int access, String name, String desc,
            String signature, Object value) {
          if (name.equals(fName)) {
            isFieldPresent = true;
}
          return cv.visitField(access, name, desc, signature, value);
        }
        @Override
        public void visitEnd() {
          if (!isFieldPresent) {
            FieldVisitor fv = cv.visitField(fAcc, fName, fDesc, null, null);
            if (fv != null) {
              fv.visitEnd();
            }
}
          cv.visitEnd();
        }
}
```
