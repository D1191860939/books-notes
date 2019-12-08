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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIzNDUzNjA2NywtMTg2NDc1OTM1LDY0Nj
k2MDgwMywtNDAxMjMxNDY0LC0yMTMyMjE5MDcyLC02NTI3ODM1
OTksMTQzODI0MDUxNCwtMTEyNjY3OTA4MF19
-->