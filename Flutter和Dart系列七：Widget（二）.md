在此之前，我们使用的都是StatelessWidget，即不带状态的，是不变的。在第三篇我们说过，widget有两种类型：StatelessWidget和StatefulWidget。今天我们就来感受一下StatefulWidget的使用。

1. 上demo

		void main() {
		  runApp(MaterialApp(
		    home: MaterialHome(),
		  ));
		}
		
		class MaterialHome extends StatelessWidget {
		  @override
		  Widget build(BuildContext context) {
		    return Scaffold(
		      appBar: AppBar(
		        leading: IconButton(icon: Icon(Icons.menu), onPressed: null),
		        title: Text("I'm title"),
		        actions: <Widget>[
		          IconButton(
		            icon: Icon(Icons.search),
		            onPressed: onSearchClick,
		          )
		        ],
		      ),
		      body: Column(
		        children: <Widget>[
		        	Text("I'm content and in center"), 
		        	Counter()
		        ],
		      ),
		      floatingActionButton: FloatingActionButton(
		        onPressed: onFabClick,
		        child: Icon(Icons.add),
		      ),
		    );
		  }
		
		  void onSearchClick() {
		    print("search button clicked");
		  }
		
		  void onFabClick() {
		    print("fab  clicked");
		  }
		}
		
		class Counter extends StatefulWidget {
		  @override
		  _CounterState createState() => _CounterState();
		}
		
		class _CounterState extends State<Counter> {
		  int _counter = 0;
		
		  void _increment() {
		    setState(() {
		      _counter++;
		    });
		  }
		
		  @override
		  Widget build(BuildContext context) {
		    return Row(
		      children: <Widget>[
		        RaisedButton(
		          onPressed: _increment,
		          child: Text('Increment'),
		        ),
		        Text('Count:$_counter')
		      ],
		    );
		  }
		}
先看效果，运行起来，我们可以看到每次当我点击button时，右边Text的值都会加1.

	再来分析代码，基于我们在上一篇的代码，我们在MaterialHome的body里添加了一个Counter，下面来分析Counter的实现：
 - 继承自StatefulWidget，并在createState()方法中返回了一个_CounterState对象
 - 在_CounterState的build方法里返回了一个Row，我们说过，这个相当于是orientation=horizontal的LienarLayout。里面放置了一个RaisedButton,关于RaisedButton，我们看下注释：
 
 > A raised button is based on a [Material] widget whose [Material.elevation]
 increases when the button is pressed.
 
 就是说RaisedButton的特点是当它被按下时，它的elevation（阴影）会变大。
 
 - 为RaisedButton设置了一个点击事件回调：\_increment().调用了setState方法，并且在它的回调中将我们定义的_counter自增。我们在这里仅仅是改变了数据，并没有去执行刷新UI的操作，但是UI的的确确是变化了的。是不是让你想起了Android中的Databinding，没错，它这里也是一个mvvm模式。我们再来看setState的文档说明（doc很长，我们截取重要的贴了出来）：

 > Calling [setState] notifies the framework that the internal state of this object has changed in a way that might impact the user interface in this subtree, which causes the framework to schedule a [build] for this [State] object.
 
 说的很清楚，调用setState方法会通知framework，widget的状态改变了，就会导致framework去调用这个State的build方法，就实现了UI刷新，这样页面显示的数字就会每点击一次自增一次。
 
2. 实际上现在界面很难看，我们现在想把它实现居中，借此机会学习Column、Row的两个属性

	

3.  
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
