基于《Android插件化开发指南》第4章对于ActivityManager hook的思考

   最近在读包建强老师的《Android插件化开发指南》一书，在读到第4章对于ActivityManager hook时，有点小启发。先看hook的代码：
   
	   public class HookHelper {
	       public static void hookActivityManager() {
	
	
	        try {
	
	            Class<?> amClass = Class.forName("android.app.ActivityManager");
	            Field gDefaultField = amClass.getDeclaredField("IActivityManagerSingleton");
	            gDefaultField.setAccessible(true);
	            // 这个iActivityManagerSingleton对象实际上就是一个Singleton对象
	            Object iActivityManagerSingleton = gDefaultField.get(null);
	
	
	            // 反射Singleton类
	            @SuppressLint("PrivateApi")
	            Class<?> singleClass = Class.forName("android.util.Singleton");
	            Field instanceField = singleClass.getDeclaredField("mInstance");
	            instanceField.setAccessible(true);
	            Object rawIActivityManager = instanceField.get(iActivityManagerSingleton);
	            
	            // 注释留空处
	
	          	
	            @SuppressLint("PrivateApi")
	            Class<?> iActivityManagerInterface = Class.forName("android.app.IActivityManager");
	            Object newProxyInstance = Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class[]{iActivityManagerInterface},
	                    new HookHandler(rawIActivityManager));
	
	            // 将IActivityManagerSingleton中的mInstance字段替换成proxy对象
	            instanceField.set(iActivityManagerSingleton, newProxyInstance);
	
	
	        } catch (ClassNotFoundException | NoSuchFieldException | IllegalAccessException e) {
	            e.printStackTrace();
	        }
	
	
	    }
	 }
	    
	 public class HookHandler implements InvocationHandler {

		private Object mBase;
		
		public HookHandler(Object base){
		    this.mBase = base;
		}
		@Override
		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		
		    Log.e("TAG", mBase.getClass().getCanonicalName() + "-> before invoke....");
		    Object result = method.invoke(mBase, args);
		    Log.e("TAG", mBase.getClass().getCanonicalName() + "-> method name -> " + method.getName());
		    Log.e("TAG", mBase.getClass().getCanonicalName() + "-> after invoke....");
		
		    return result;
		}
	 }
	 
基础知识点就是反射和动态代理（不熟悉的童鞋可参看我的之前一篇文章）的使用。紧接着，我在MainActivity中调用了hookActivityManager方法，然后运行app，查看Log：

哇塞，hook成功了，好强大！但是当我第二次再打开app时，发现了一个问题：

// 贴图

发现会调用两次。第三次打开app呢？

// 贴图

会调用三次。

// 贴图

这是怎么回事呢？因为我们是在MainActivity中调用的，每次打开app都会进行一次hook，这样当第二次hook时，实际上是对第一次hook的代理对象又一次进行代理封装，此时相当于有两层代理对象了。同样地，第三次打开app时就会有三层代理对象了。所以，InvocationHandler的invoke方法会输出多次。我们可以试图来解决这个问题。

在前面代码的注释留空处添加：

	if (Proxy.isProxyClass(rawIActivityManager.getClass())) {
	    return;
	}

这样就会hook一次。




   