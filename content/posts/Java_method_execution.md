---
title: "Java 方法执行"
date: 2024-06-09T20:44:02+08:00
draft: false
---

- 从源码到cpu执行的过程梳理
- common method execution scenarios such as main method, constructor, static method, polymorphism, reflection, lambda expression
- beyond java

# how source code execute to cpu

JVM load class file and interpret/execute the bytecode.

class file contains byte sequence, which is platform neutral. 
- Java source code was compiled to platform neutral byte code, wellknowed as class file. java class file is with strict format, define every thing of java class. 

- class loading is the process of loading class file into java virtual machine at runtime, the method area, basicly.

- main method is the entry point of java application, jvm then continue to load and execute from that.



# method invocation scenarios

main方法

new 关键字构造对象执行时分为两个指令, new 和 invokespecial <init>

>检查 new 指令参数指向的符号引用对应类是否被加载
分配内存，内存大小在类加载时确定。
Java内存分配线程安全
初始化内存，设置字段#零值
对象头设置
new 关键字对应第二条指令 `invokespecial <init>` [[<init>]]。Java 类编译时生成`<init>`收敛方法，依次包含对父类无参构造器，子类无参构造器，字段初始化等

invoke方法调用指令的一般执行过程  

> invoke指令后面的参数(非方法参数) 方法的符号引用，指向常量池中方法名称和描述符常量。这些符号引用是在编译时由编译器根据接收者的静态类型和参数的静态类型，确定方法版本，生成确定的invoke指令，即invoke参数为符号引用。
> 一些指令的符号引用在类加载解析阶段或第一次使用时变为直接引用，直接指向方法的内存结构。
> 对于invokevirtual，要在运行时判断接收者的实际类型进行动态分派。

对于 private 方法, 构造器方法, super方法, final方法, static 方法这几类的符号引用均可以在类加载即转换为唯一的直接引用。
其中private, 构造器, super方法调用对应的字节码指令为invokespecial。
static invokestatic
final方法的字节码invokevirtual

Java中成员方法默认为虚方法(与c++虚函数类似)，虚方法调用机制为动态分派，运行时根据方法接收者的实际类型动态判断符号引用的指向方法的唯一版本。

与动态分派(dynamic dispatch)对应静态绑定(或静态解析、)，


反射方式进行方法调用，同普通方法调用，将参数传递给方法区的方法对象，不同的地方在于普通方法调用依赖编译器必须知道参数的静态类型（not实际类型）静态解析或动态分派完成方法分派，而反射提供类似动态语言的特性，只要target有该Method方法即可完成调用，否则抛出NoSuchMethodException。  
反射实现方式有两种，native code 和 bytecode-based（动态字节码）。  
从java 18开始，反射的底层实现变为MethodHandle，提供更现代化的反射实现。  

lambda expression  





- javap -verbose 查看class文件字节码内容
- [] method reflection metadata
- https://stackoverflow.com/questions/5698614/java-final-methods-vs-c-nonvirtual-functions
- [反射性能开销原理及优化](https://github.com/xfhy/Android-Notes/blob/master/Blogs/Java/%E5%9F%BA%E7%A1%80/%E5%8F%8D%E5%B0%84%E6%80%A7%E8%83%BD%E5%BC%80%E9%94%80%E5%8E%9F%E7%90%86%E5%8F%8A%E4%BC%98%E5%8C%96.md)



a class byte sequence describe the original class structure, using well defined differrent data types for different parts of class, such as class hirachechy, field, method signature and the method code, etc.

method code is compiled to byte code. method invoke code was compiled to versatile byte code instruction, the instruction may follow the symbolic reference, which is used for jvm engine to find the target method.

when jvm interprete to execute the byte code, different 

- Java source code was compiled to platform neutral byte code, wellknowed as class file. java class file is with strict format, define every thing of java class. 

- class loading is the process of loading class file into java virtual machine at runtime, the method area, basicly.

- main method is the entry point of java application, jvm then continue to load and execute from that.

