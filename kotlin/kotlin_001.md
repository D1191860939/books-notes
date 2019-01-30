#  Android组件化与热修复
#### 1. Java中的类加载器
##### 1.1 系统类加载
- 引导类加载器（Bootstrap ClassLoader）：用C/C++代码实现，用于加载指定的JDK核心库
- 拓展类加载器（Extensions ClassLoader）：用于加载Java的拓展类
- 应用类加载器（Application ClassLoader）：又称作系统类加载器
##### 1.2 自定义类加载器
> 用户自定义类加载器，是通过继承java.lang.ClassLoader类的方式来来实现自己的类加载器
> 
##### 1.3 类加载器子系统
- 加载：查找并加载Class文件
- 链接：验证、准备以及解析
> 验证：确保被导入类型的正确性
> 准备：为类的静态字段分配字段，并用默认值初始化这些字段
> 解析：根据运行时常量池的符号引用来动态决定具体执行过程
- 初始化：将类变量初始化为正确初始值

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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MjI5NTM5NDEsLTE4MzkwMTU1NTldfQ
==
-->