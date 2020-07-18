## 七、走进注解驱动变成

#### 1.Enable实现

​		![Enable注解实现](E:\learn\reading-notes\spring-boot核心原理\Enable注解实现.png)

Spring对于注解属性的抽象：在Java反射编程模型中，注解之间无法继承，也不能实现接口，不过Java语言默认将所有注解实现Annotation接口，被标注的对象用API AnnotatedElement表达。通过AnnotatedElement#getAnnotation(Class)方法返回指定类型的注解对象，获取注解属性则需要显示地调用对应的属性方法。