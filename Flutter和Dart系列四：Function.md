Flutter和Dart系列四：Function

1. Dart语言是一门真正面向对象的语言，函数也是一个对象，并且类型为Function，这点和Js是类似的。
2. 定义Function：

		返回类型 name (参数列表){
		
	
		}
	
	例如：
		
		bool isAdult(int age){
			return age >= 18;
		}
		
		void main() {
  			print(isAdult(28));	// true
		}
		
	对于isAdult函数而言：
	- 可以省略前面的返回值类型：因为返回值类型可以通过return的值类型推断出来，但是最好还是不要省略，这里说明一下只是想表达从语法角度上是允许的。
	
			isAdult(int age) {
				return age >= 18;
			}
			
			void main() {
	  			print(isAdult(28));	// true
			}
			
	- 使用=> 改写函数体形式：=>，箭头语法，当函数体只有一条语句时，可以使用箭头语法进行简化。=> expr,相当于{return expr}。那么，isAdult函数就变成了：

			bool isAdult(int age) => age >= 18;
			
		这点实际上和Kotlin对于一条语句的函数体的简化是相似的。
	  	
	  		fun method() = "hello world"
	注意：在=> 和；之间只能出现的是一个表达式，而不能是一条语句。
			
3. 讨论完了返回值，函数体，接下来就是参数了。在Kotlin中有个概念叫做有名参数（named parameters），在dart中也有，虽然稍微有点不同。我们可以在调用函数传递参数时，指定参数名：

		void sayHello({String name, String msg}) => print("$name: $msg");
	
	
	我们使用{}将形参包含起来，那么我们在调用时：
		
		void main() {
	  		sayHello(name: "david", msg: "你好啊"); // david: 你好啊
		}
		
	这样是不是很直观？我们还可以这么调用：
		
		void main(){
			sayHello(msg: "你好啊", name: "david",);  // david: 你好啊
			sayHello(msg: "hello");			//null: hello
  			sayHello(name: "david");     //david: null
			sayHello();			// null: null
		}
	换句话说，{}中的参数是可传可不传的，故它叫做可选的有名参数（optional named parameters）.
	这种方式在日常Flutter开发很常见。
	
4. 可选的位置参数(optional position parameters).

	语法格式为：
	[参数列表]
	
		String say(String from, String msg, [String device, String time]) {
	  		
	  		var result = "$from says $msg";
	
			if (device != null) {
			  result = "$result with a $device";
			}
			
			if (time != null) {
			  result = "$result at $time";
			}
			
			return result;
		}
	
		void main(){
			print(say("david", "hello", "mobile", "2019.2.29.20:08:08"));
			// david says hello with a mobile at 2019.2.29.20:08:08
			print(say("david", "hello", "2019.2.29.20:08:08", "mobile"));
			// david says hello with a 2019.2.29.20:08:08 at mobile
			print(say("david", "hello"));
			// david says hello
		}
	
	可以看出：
	- 同样作为可选的参数，故在调用时可以不传
	- 是通过位置来确定实参和形参的对应关系的

	
5. 默认参数值（Default parameter values）: 对应可选的有名参数和可选的位置参数而言，我们可以指定形参的默认参数值.

		String say(String from, String msg, [String device = "mobile", String time = "2019.2.23.18:30:00"]) {
		  
		  var result = "$from says $msg";
		
		  if (device != null) {
		    result = "$result with a $device";
		  }
		
		  if (time != null) {
		    result = "$result at $time";
		  }
		
		  return result;
		}

		void main(){
			print(say("david", "hello", "mobile", "2019.2.29.20:08:08"));
			// david says hello with a mobile at 2019.2.29.20:08:08
			print(say("david", "hello", "2019.2.29.20:08:08", "mobile"));
			// david says hello with a 2019.2.29.20:08:08 at mobile
			print(say("david", "hello"));
			//david says hello with a mobile at 2019.2.23.18:30:00
		}
	
	即如果为可选参数指定了实参，那么在函数中就使用的是实参的值；如果没有传递，那么就使用默认的参数值。
	
6. 函数是作为一级类的对象来看待的。

		void printItem(int item) {
	  		print(item);
		}
	
		void main(){
		  	var list = [1, 2, 3];
  			list.forEach(printItem);
  			// 1
  			// 2
  			// 3
		}
	
	对于前端同学这块很好理解，而对于Android同学而言，这块就需要时间去接受一下了。
	
7. 匿名函数（Anonymous Functions）.
	语法： ([type] param){}
	例如上面的List遍历：
	
		void main(){
			var list = [1, 2, 3];
			
			list.forEach((item) {
			   print(item);
			});

		}	
		
	有点像Java和Kotlin中lambda表达式。
	
			
	