1. Dart是一门面向对象的语言。和Java一样，它有个超级父类Object，所有的类都直接或间接继承该类。
2. 定义一个类：

		class Point {  
		  num x = 0, y = 1;  
		}
- 编写main方法来使用这个类：
	   
		void main() {  
		  var point = Point();  //创建了一个Point类型的对象，可以加new关键字，也可以不加
		  print("${point.x}, ${point.y}");  //0, 1
		}
	 和Kotlin类似，我们可以使用?.来表示可空类型
	
		Point point2;  
		print('${point2?.x}, ${point2?.y}'); //这里不会造成空指针异常，而是会打印null, null
	对于成员的访问，这里说明一下：它的设计理念是和Kotlin类似的，在Java中，通常将域（Field）和它的访问器（Setter/Getter）联合起来称作是属性（Property），而在Dart和Kotlin中，属性是作为一级语言特性，完全替代了域和访问器的概念。具体的理解可以参看链接 **backing_field**
	
- 访问控制：Dart中是没有public、protected、和private关键字的。换句话说，是没有访问控制修饰符的。如果成员（属性或者是方法）是_开头，那么就表明它是在该library可见的。
	
- 如何得到某个对象的类型？在Object类中有个runtimeType方法，它会返回一个Type类型的对象，有点类似Class对象。
		  
		print("${point.runtimeType}");	// Point
3. 构造器（Constructor).

	   class Point {  
		  num x, y;  
	    
		  Point(num x, num y){  //参数类型可以省略
		    this.x = x;  
		    this.y = y;  
		  }  
		}
- 同样使用this关键字指代当前对象。但是在Dart中除了出现这种命名冲突，建议使用this关键字，其他地方都不建议使用它
	
- 实际上这么写构造器有点繁琐，可以改造一下：
	
		class Point {  
		  num x, y;  
		    
		  Point(this.x, this.y);
		}
- 还记得我们在系列四一节介绍的optional named parameters吗？和构造器联合起来使用就成了如下形式：
	
		class Point {  
	      num x, y;  
	  
	      Point({this.x, this.y});  
	   }
	 这种方式在Flutter中很常见的。
	  
-  有名构造器(Named constructors)：
	
		class Point {  
		  num x, y;  
	  
		  Point(this.x, this.y);  
	      //named constructor
		  Point.origin(){  
		    x = 0;  
		    y = 0;  
		  }  
		}
	在使用处：
		
		var point = Point.origin();  // 通过有名构造器创建Point对象
		print('${point.x}, ${point.y}');	// 0, 0

 - 初始化列表（initializer list）
   
	    Point.fromJson(Map<String, num> json)  
		    : x = json['x'],  
		      y = json['y'] {  
		  print('fromJson...');  
		}
	我们可以在构造器后面添加相关参数初始化语句，并且它的执行顺序是在构造器体执行之前。
 - 重定向构造器（Redirecting constructors）
    
	    class Point {  
		  num x, y;  
		  Point(this.x, this.y);  
		  Point.alongXAxis(num x): this(x, 0);
	   }
	这种方式应该很熟悉吧。
- 工厂构造器（factory constructor）

		class Logger{  
		  final String name;  
		  bool mute = false;  
		    
		  static final Map<String, Logger> _cache = new Map<String, Logger>();  
	    
		  // 在factory constructor中是不能够访问this关键字的，换句话说，这个就是静态工厂  
		  factory Logger(String name){  
		    if(_cache.containsKey(name)) {  
		      return _cache[name];  
		    }else{  
		      final logger = Logger.internal(name);  
		      _cache[name] = logger;  
		      return logger;  
		    }  
		  }  
		    
		  Logger.internal(this.name);  
		    
		  void log(String msg){  
		    if(!mute)
		      print(msg);   
		  }  
		}
	
4. 继承（inheritance）.使用extends关键字表示类的继承，shi'yong。
5. 多态
6. 抽象类和接口






























d. 下面几个注意点是和Java类似的：
	  i.  当我们没有为类显式地定义构造器时，编译器会为我们声明一个默认的无参构造，在这个无参构造中它会调用父类的无参构造.
	  ii. 构造器是不能够被继承的。
	  
	  e. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUxMDAwMDQ5NiwtMjIzNzY5MzMwXX0=
-->