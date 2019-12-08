#  Android组件化与热修复
#### 1. Java中的类加载器
##### 1.1 系统类加载
- 引导类加载器（Bootstrap ClassLoader）：用C/C++代码实现，用于加载指定的JDK核心库，JAVA_HOME/jre/lib
- 拓展类加载器（Extensions ClassLoader）：用于加载Java的拓展类, JAVA_HOME/jre/lib/ext
- 应用类加载器（Application ClassLoader）：又称作系统类加载器
##### 1.2 自定义类加载器
> 用户自定义类加载器，是通过继承java.lang.ClassLoader类的方式来来实现自己的类加载器
> 
##### 1.3 类加载器子系统
- 加载：查找并加载Class文件，然后保存到方法区，并且在Java堆内存中会创建一个Class对象来代表这个.class文件
- 链接：验证、准备以及解析
> 验证：确保被导入类型的正确性
> 准备：为类的静态字段分配内存，并将其初始化为默认值
> 解析：根据运行时常量池的符号引用来动态决定具体执行过程，符号引用转换成直接引用
- 初始化：将类变量（static）初始化为正确初始值

##### 1.4 相关补充说明
- 除了Bootstrap ClassLoader，Extension ClassLoader和Application ClassLoader以及Custom ClassLoader都继承自java.lang.ClassLoader类

#### 2. JVM运行时数据区域
- 程序计数器：线程私有，如果当前线程执行的方法不是Native方法，则程序计数器保存的就是正在执行的字节码指令地址，如果是Native方法则程序计数器的值为空（Undefined）
- Java虚拟机栈：线程私有
- 本地方法栈
- Java堆：所有线程共享的运行时内存区域
- 方法区：所有线程共享的运行时内存区域

#### 3. Android中的ClassLoader
- BootClassLoader：
- PathClassLoader：通常用来加载已经安装的apk的dex文件（安装的apk的dex文件会存储在/data/dalvik-cache中）
- DexClassLoader：

#### 4. 初始化顺序：
- 静态代码块和静态域初始化在clinit中的先后关系就是两者在源码中的先后关系
- 非静态代码块和非静态域初始化在init中的先后关系就是两者在源码中的先后关系，这点是和静态代码块和静态域是一致的
- final static域优化原理：这种优化仅仅体现在final static原生类型和String类型域（非引用类型），如果是引用类型，实际上不会得到任何优化。这种优化会体现在：
	- sget指令（非final使用）和const/4指令（final使用）相比，它的解释要繁重的多
	
#### 5. 获取Class对象的几种方式：
- 通过对象的getClass方法，这个方法是定义在Object类中的
- 通过Class.forName()方法，需要传入一个字符串：类名的全限定名
- 通过类的class属性
- 对于原生数据类型而言，它们的包装类都有一个TYPE属性，来代表对应的class对象

#### 6. 可以作为GC Roots的对象
- 虚拟机栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈（即一般说的native方法）中引用的对象

#### 7. Java虚拟机在以下情况下将结束生命周期：
- 执行了System.exit()方法
- 程序正常执行退出
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统错误而导致Java虚拟机进程终止

#### 8. Java程序对于类的使用分为两种方式
- 主动使用
-  被动使用

#### 9. 所有的Java虚拟机实现都必须在每个类或接口被Java程序“首次主动使用”时才初始化他们

#### 10. 主动使用：
- 创建类的实例
- 访问某个类或接口的静态变量，或者对该静态变量赋值（对于静态变量的set、get，反应到指令层面上就是getstatic、putstatic）
- 调用类的静态方法 (反应到指令层面上就是invokestatic)
-  反射，如Class.forName("");
- 初始化一个类的子类
- Java虚拟机启动时被标明为启动类的类	（包含main方法）
- jdk1.7之后，java提供动态语言的支持
#### 11.
  
/**  
 * 一、所有的jvm实现都必须在每个类或接口被java程序“首次主动使用”时才初始化他们  
  * 二、触发首次主动使用：  
  * 1. 创建一个类的实例  
  * 2. 访问一个类的静态字段，或者给该静态字段赋值  
  * 3. 调用类的静态方法  
  * 4. 发射  
  * 5. 初始化一个类的子类  
  * 6. jvm启动时被标明为启动类的类 main方法所在的类  
  *  
 * 该程序所说明的问题：  
  * 1. 初始化一个类的子类会触发该类被初始化，如这里的Animal类，我们在触发其子类Dog的初始化时，会导致Animal类自身初始化。换句话说，  
  * 当一个类被初始化时，要求其父类全部都已经初始化完毕  
  * 2. 对于静态字段来说，只有直接定义了该字段的类才会被初始化  
  * 3. 同时也再次强调了静态代码块的作用：在类被初始化时为类的静态字段赋值。换句话说，当一个类被初始化时，会触发该类的静态代码块的调用  
  *  
 * 以上所说明的点都是在类加载子系统的第三阶段所完成的。这三个阶段分别是：  
  * 1. 加载：将class文件读取载入内存  
  * 2. 连接  
  * - 验证： 验证所导入类的正确性，这个要是拓展的话，就是class文件本身的格式  
  * - 准备： 为类的静态字段分配内存，并初始化为默认值  
  * - 解析： 根据运行时常量池的符号引用来动态决定具体执行过程，将符号引用转换成直接引用  
  * 3. 初始化  
  */  
  
public class MyTest1 {  
  
    public static void main(String[] args) {  
        System.out.println(Dog.color);  
    }  
}  
  
class Animal{  
  
    public static String name = "animal";  
  
    static {  
        System.out.println("in Animal's static block.");  
    }  
}  
  
class Dog extends Animal{  
  
    public static String color = "white";  
  
    static {  
        System.out.println("in Dog's static block.");  
    }  
}
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgxMDY2MzMzOCwtMTE5MjkwMTAwMSwtMT
IwNDY1MDk3OCw1MTAwNDM2NDAsLTE5NDQ2ODUyNDksMjg0NTkx
NTU4LDEyMzQ1MzYwNjcsLTE4NjQ3NTkzNSw2NDY5NjA4MDMsLT
QwMTIzMTQ2NCwtMjEzMjIxOTA3MiwtNjUyNzgzNTk5LDE0Mzgy
NDA1MTQsLTExMjY2NzkwODBdfQ==
-->