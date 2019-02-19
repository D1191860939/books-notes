1. 从SystemServer的main方法出发：它直接创建了一个SystemServer的对象，并且调用了它的run方法
2. 在SystemServer的run方法中：
- 通过System.loadLibrary方法加载动态so库：System.loadLibrary("android_servers");
- 启动了三类服务：
	- startBootstrapServices()：
			
			mActivityManagerService = mSystemServiceManager.startService(ActivityManagerService.Lifecycle.class).getService();
	- startCoreServices()
	- startOtherServices()
3. 我们仔细看一下startBootstrapServices()方法的实现：

		public SystemService startService(String className) {
			    final Class<SystemService> serviceClass;
		        try {
		            serviceClass = (Class<SystemService>)Class.forName(className);
		        return startService(serviceClass);
		    }
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjM4Mzk1NjQyLC0yMzAwMzk4MjBdfQ==
-->