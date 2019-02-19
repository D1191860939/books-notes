1. 从SystemServer的main方法出发：它直接创建了一个SystemServer的对象，并且调用了它的run方法
2. 在SystemServer的run方法中：
- 通过System.loadLibrary方法加载动态so库：System.loadLibrary("android_servers");
- 启动了三类服务：
	- startBootstrapServices()：
	- startCoreServices()
	- startOtherServices()
3. 我们仔细看一下startBootstrapServices()方法的实现：

		mActivityManagerService = mSystemServiceManager.startService(ActivityManagerService.Lifecycle.class).getService();
4. SystemServiceManager.startService()方法：

		public SystemService startService(String className) {
			 final Class<SystemService> serviceClass;
		     serviceClass = (Class<SystemService>)Class.forName(className);
		     return startService(serviceClass);
		 }

		public <T extends SystemService> T startService(Class<T> serviceClass){

	        Constructor<T> constructor = serviceClass.getConstructor(Context.class);
	        service = constructor.newInstance(mContext);
	        startService(service);
	        return service;
		}

		public void startService(@NonNull final SystemService service) {
		   mServices.add(service);
		   service.onStart();
		}
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NjA5NzYxNzMsMTQzNTYyODI5MywtMj
MwMDM5ODIwXX0=
-->