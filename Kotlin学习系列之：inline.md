1. inline: Kotlin中的一个关键字，用来修饰function，那么这个function就被称作inline function（内联函数）。最初接触内联函数这个概念还是当初在学校学习C语言时候提到的，Java中是没有这一概念的，如今Kotlin又引入这一特性。那么inline function有什么优势或者说特点呢？
2. 特点：当你调用一个inline function的时候，编译器不会产生一次函数调用，而是会在每次调用时，将inline function中的代码直接嵌入到调用处。
3. 那么在Kotlin中如何去声明一个inline function呢？
		
		inline fun xxxx(){
		}
    很简单，直接在function定义之前添加一个inline关键字修饰，就可以将一个普通的function变成一个inline function了，至于函数调用也和普通函数一样
 4. 透过现象看本质：我们在第2条中阐述过inline function的特点，那么事实是否真是如此呢？我们来看代码：
InlineTest2.kt

		fun main(args: Array<String>) {  
		    sayHello("David")  
		}  
		  
	    fun sayHello(name: String) {  
		    println("Hi, $name")  
		}
	代码很简单，就不多说了。我们直接运行，会输出：Hi, David.下面我们进入找InlineTest2.kt对应的class文件。
	![enter image description here](https://img-blog.csdnimg.cn/2019020110215765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)
	接下来我们做一件事：通过javap -c命令反编译这个InlineTest2Kt.class，会得到如下结果：
![enter image description here](https://img-blog.csdnimg.cn/20190201102245740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)
	我们会看到Method invocation。最后，我们将sayHello这个function使用inline修饰，将其变成inline function，再次build，再来反编译这个InlineTest2Kt.class，会看到如下
![enter image description here](https://img-blog.csdnimg.cn/20190201102756361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)
	我们可以看到，sayHello的代码会直接出现在main function中，没有我们之前看到的Method invocation。
	至此，我们就可以证明之前的结论了。

5. 我们再回过头来看这个sayHello函数，我们会看到编译器会给我们一个警告：
    ![enter image description here](https://img-blog.csdnimg.cn/20190201102828626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)
	翻译过来就是，我们将sayHello函数声明成inline是没有意义的，inline需要和接收函数类型参数的function一起使用才是最好的。为什么编译器要这么去要求呢？我们用代码去证明：这里的函数类型参数我们可以简单理解成lambda（尽管我们现在的系列还有介绍过lambda，我会在后面的系列补充）。
	在我们的InlineTest2.kt文件中添加一个method函数：
	
		fun method(print: (String) -> Unit) {  
		    print("hello world")  
		}

		fun main(args: Array<String>) {  
			 method { }
		}
	method接收一个lambda表达式作为参数，然后运行程序，会打印：hello world。结果不重要，重要的是：你应该能猜到我接下来要干啥，对，将反编译贯彻到底。
![enter image description here](https://img-blog.csdnimg.cn/20190201102859641.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)
	可以看到，lambda表达式在字节码层面上还是创建一个匿名内部类的对象，也就是说只能语法糖，写代码时更加方便，本质上并没有什么改变。但是如果我们将method方法变成inline？
![enter image description here](https://img-blog.csdnimg.cn/20190201102926523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)

	惊奇的发现，这个匿名内部类的对象没有了，是直接把这个"hello world"字符串嵌入到了调用处(main方法中)。这下我们就能明白编译器的用心良苦了。inline和接收函数类型参数的function才是绝配。

6. inline和reified关键字
这块实际上设计到Kotlin中泛型的相关知识。
		
		inline fun <reified T> isA(value: Any) = value is T
	大家可以先把inline和reified两个关键字去掉来看，实际上就是Kotlin中泛型方法的声明方式，和Java中的泛型方法类似的。接下来我们重点讨论函数体的实现：
	
		value is T

	可以看到，泛型T被用来类型比较，我们都知道由于泛型的类型擦除，T在运行时实际上和Object类型一样，并没有具体的类型信息。所以这条语句在普通的泛型方法里面是会编译报错的，现在再把inline和reified关键字加回来：

		fun main(args: Array<String>) {  
		    println(isA<String>("abc"))  		// true
		    println(isA<String>(123)) 		// false
		}
	从现象可以看出reified声明的类型参数不会被擦除。但是并不是没有限制的：
	
	- 依旧不能创建泛化类型的实例
	- 不能将类、属性以及非内联函数标记成reified
	- 不能在需要reified的地方传递一个non-reified的类型参数
	- 不能够调用泛型类的伴生对象中的方法

	
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMTA4NjgyOSwyOTkwNzQyNjksLTIwOD
g3NDY2MTJdfQ==
-->