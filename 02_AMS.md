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
5. ActivityManagerService.Lifecycle类：

		public static final class Lifecycle extends SystemService {
			private final ActivityManagerService mService;
			 public Lifecycle(Context context) {
				 super(context);
				 mService = new ActivityManagerService(context);
			 }
			 @Override
			 public void onStart() {
				 mService.start();
			 }
			 public ActivityManagerService getService() {
				 return mService;
			 }
		 }

6. ActivityStack：用来管理系统所有Activity的各种状态
7. ActivityState：通过枚举定义了Activity的所有状态，在ActivityStack类的内部
8. ActivityStackSupervisor：中有多个ActivityStack实例
9. ActivityRecord用来记录一个Activity的所有信息；一个或多个ActivityRecord会组成一个TaskRecord，TaskRecord用来记录Activity栈；ActivityStack包含了一个或多个TaskRecord
10. 










<!--stackedit_data:
eyJoaXN0b3J5IjpbMjEwMDkzMTgzMSw0NDQ1NTU3NDYsOTg1MD
Y0MjI4LC0xNTYwOTc2MTczLDE0MzU2MjgyOTMsLTIzMDAzOTgy
MF19
-->