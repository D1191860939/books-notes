1. 从SystemServer的main方法出发：它直接创建了一个SystemServer的对象，并且调用了它的run方法
2. 在SystemServer的run方法中：
- 通过System.loadLibrary方法加载动态so库：System.loadLibrary("android_servers");
- 启动了三类服务：
	- startBootstrapServices()：会启动ActivityManagerService
	- startCoreServices()
	- startOtherServices()
3. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIzMDAzOTgyMF19
-->