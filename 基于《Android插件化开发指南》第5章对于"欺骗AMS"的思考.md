基于《Android插件化开发指南》第5章对于"欺骗AMS"的思考

&nbsp;&nbsp;&nbsp;&nbsp;在我们初学Android阶段经常会遇到一个异常：

	  Unable to find explicit activity class xxx; have you declared this 
	activity in your AndroidManifest.xml?
	
&nbsp;&nbsp;&nbsp;&nbsp;异常信息提示的很清楚，原来我们在startActivity一个新的Activity时，忘记在AndroidManifest.xml文件中进行注册了。这条异常信息我们可以在Android源码中的Instrumentation类中的checkStartActivityResult()方法中找到出处。为什么要注册呢？因为我们知道，对于Activity，framework通过解析AndroidManifest.xml文件，然后通过反射的方式创建一个Activity，不注册当然就无法去创建一个Activity实例。那么我们是否可以通过某种方式来启动一个未注册的Activity呢？

&nbsp;&nbsp;&nbsp;&nbsp;在《Android插件化开发指南》一书中，为我们提供了一种"欺骗AMS"从而可以启动一个未注册Activity的方式，下面直接贴代码（建议大家先阅读相关资料，对Activity的启动流程有个大致的了解），基于的源码是Android9.0（api28）。

在MainActivity中，通过点击按钮，跳转到一个新的Activity：

	fun goSecond(view: View) {
		val intent = Intent(this, NoRegisterAct::class.java)
    	startActivity(intent)
    }
    
并且在onCreate中调用：

	AMSHookHelper.hookAm()
下面是AMSHookHelper类的实现：

	public class AMSHookHelper {

       public static final String EXTRA_TARGET_INTENT  = "target_intent";

	    @SuppressLint("PrivateApi")
	    public static void hookAm() {

	        try {
	        	// 通过反射拿到ActivityManager中的Singleton对象
	            Class<?> amClass = Class.forName("android.app.ActivityManager");
	            Field singleField = amClass.getDeclaredField("IActivityManagerSingleton");
	            singleField.setAccessible(true);
	            Object rawSingleton = singleField.get(null);
	
			    // 通过反射拿到Singleton中的mInstance
	            Class<?> sClass = Class.forName("android.util.Singleton");
	            Field instanceField = sClass.getDeclaredField("mInstance");
	            instanceField.setAccessible(true);
	            Object rawInstance = instanceField.get(rawSingleton);
	
				// 防止多次重复hook，在"基于《Android插件化开发指南》第4章对于ActivityManager hook的思考"一文已介绍
	            if (Proxy.isProxyClass(rawInstance.getClass())) {
	                return;
	            }
	
				// 拿到动态代理对象
	            Class<?>[] interfaces = new Class[]{Class.forName("android.app.IActivityManager")};
	            Object proxy = Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), interfaces, new AMSHandler(rawInstance));
	
				// 用动态代理对象来替换Singleton中的mInstance
	            instanceField.set(rawSingleton, proxy);
	
	
	        } catch (ClassNotFoundException | NoSuchFieldException | IllegalAccessException e) {
	            e.printStackTrace();
	        }
	
	    }
    }

这个地方的核心就是通过反射和动态代理将Singleton中的mInstance属性hook，用一个代理对象类替换它。实际上这个mInstace就是指向ActivityManagerService的实例。下面贴出AMSHandler的源码:

	public class AMSHandler implements InvocationHandler {

        private Object mBase;

	    public AMSHandler(Object base) {
	        this.mBase = base;
	    }

	    @Override
	    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	
			 // hookActivityManagerService的startActivity方法
	        if ("startActivity".equals(method.getName())) {
	
	            Intent rawIntent;
	            int index = 0;
	
	            // 遍历，从args中找到Intent
	            for (int i = 0; i < args.length; i++) {
	                if (args[i] instanceof Intent) {
	                    index = i;
	                    break;
	                }
	            }
	            
	            rawIntent = (Intent) args[index];
	
	            // 创建新的替身Intent
	            Intent newIntent = new Intent();
	
	            String stubPackage = rawIntent.getComponent().getPackageName();
	            // 这个StubActivity我们在AndroidManifest.xml中已注册
	            String cls = StubActivity.class.getName();
	
	            // 将componentName设置到替身Intent中
	            ComponentName componentName = new ComponentName(stubPackage, cls);
	            newIntent.setComponent(componentName);
	
	            // 将原始Intent作为extra放置到替身Intent中
	            newIntent.putExtra(AMSHookHelper.EXTRA_TARGET_INTENT, rawIntent);
	
	            // 用替身Intent替换原始Intent
	            args[index] = newIntent;
	        }
	
	        return method.invoke(mBase, args);
	    }
	}

我们已经对相关源码进行了注释，核心思想就是将一个新的替身Intent来替换原来的Intent，并且将我们事先已注册的StubActivity来设置到新的替身Intent中，这就意味着欺骗AMS：我们要启动的并不是未注册NoRegisterAct，而是一个已经注册的StubActivity，这样就不会抛出我们文章开头提到的异常了。
既然已经替换了，但是StubActivity并不是我们所期望的Activity，自然而然我们还需要替换回来。
重写MainActivity的attachBaseContext()方法：

	override fun attachBaseContext(newBase: Context?) {
	    super.attachBaseContext(newBase)
	    HookHelper.attachBaseContext()
	}
	
下面是HookHelper类的实现：

	@SuppressLint("PrivateApi")
    public static void attachBaseContext() {


        try {
            //反射得到ActivityThread的实例
            Class<?> ahClass = Class.forName("android.app.ActivityThread");
            Field ahField = ahClass.getDeclaredField("sCurrentActivityThread");
            ahField.setAccessible(true);
            Object sCurrentActivityThread = ahField.get(null);

			// 反射得到ActivityThread中的mInstrumentation
            Field instrumentField = ahClass.getDeclaredField("mInstrumentation");
            instrumentField.setAccessible(true);
            Instrumentation rawInstrumentation = (Instrumentation) instrumentField.get(sCurrentActivityThread);

            // 替换mInstrumentation原来指向对象，这里用的是一个静态代理
            instrumentField.set(sCurrentActivityThread, new EvilInstrumentation(rawInstrumentation));


        } catch (ClassNotFoundException | NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }

    }
    
接着看EvilInstrumentation类的实现：

	public class EvilInstrumentation extends Instrumentation {

	    private Instrumentation mBase;
	
	    public EvilInstrumentation(Instrumentation base) {
	        this.mBase = base;
	    }
	    
	    @Override
	    public Activity newActivity(ClassLoader cl, String className, Intent intent) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
	        Log.e("EvilInstrumentation", "newActivity()");
	
	        Intent rawIntent = intent.getParcelableExtra(AMSHookHelper.EXTRA_TARGET_INTENT);
		
	        if (rawIntent == null) {
	            return mBase.newActivity(cl, className, intent);
	        }
	
	        String newClassName = rawIntent.getComponent().getClassName();
	        return mBase.newActivity(cl, newClassName, rawIntent);
	    }
	
	}  
	
在newActivity方法中我们实现了Intent还原，这样我们启动的就是所期望的NoRegisterAct了。

	class NoRegisterAct : Activity() {

	    override fun onCreate(savedInstanceState: Bundle?) {
	        super.onCreate(savedInstanceState)
	        val textView = TextView(this)
	        textView.text = "I'm in no register act"
	        setContentView(textView)
	    }
	}	
	
//贴图![enter image description here](https://img-blog.csdnimg.cn/20190307140545818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)

ok，这样整个欺瞒的过程也就结束了，我们已经可以实现启动未注册的Activity了，很开心。本来应该到此就结束了，但是我对源码中创建替身Intent的方式（即AMSHandler的invoke方法的实现中）有点怀疑：它是直接new了一个新的Intent对象，那么万一rawIntent中携带有相关参数呢，那么我们在目标Activity不是取不到了吗？但是转念一想，不应该，因为在newActivity方法中将它还原成rawIntent。嗯，我们来试一试，我们在启动的时候：

	val intent = Intent(this, NoRegisterAct::class.java)
    intent.putExtra("data", "hello world")
    startActivity(intent)
    
然后再到NoRegisterAct中尝试去取：

	 override fun onCreate(savedInstanceState: Bundle?) {
	    super.onCreate(savedInstanceState)
	    val textView = TextView(this)
	    val data = intent.getStringExtra("data")
	        
        textView.text = if (!TextUtils.isEmpty(data)) {
            	data
        	} else {
            	"I'm in no register act"
        	}

        setContentView(textView)
    }    
屏幕上显示的是I'm in no register act，而不是hello world，这就意味着我们确实没有取到。有点懵，不是替换了吗，为什么没有？带着疑问，我先尝试使用clone的方式来得到一个替身Intent，这样替身Intent中也带有了相关数据：

	// 创建新的替身Intent
	//  Intent newIntent = new Intent();   
	Intent newIntent = (Intent) rawIntent.clone();
	
顺带看一下Intent的clone方法：

	@Override
	public Object clone() {
	   return new Intent(this);
	}	
	
	/**
     * Copy constructor.
     */
    public Intent(Intent o) {
        this(o, COPY_MODE_ALL);
    }

    private Intent(Intent o, @CopyMode int copyMode) {
        this.mAction = o.mAction;
        this.mData = o.mData;
        this.mType = o.mType;
        this.mPackage = o.mPackage;
        this.mComponent = o.mComponent;

        if (o.mCategories != null) {
            this.mCategories = new ArraySet<>(o.mCategories);
        }

        if (copyMode != COPY_MODE_FILTER) {
            this.mFlags = o.mFlags;
            this.mContentUserHint = o.mContentUserHint;
            this.mLaunchToken = o.mLaunchToken;
            if (o.mSourceBounds != null) {
                this.mSourceBounds = new Rect(o.mSourceBounds);
            }
            if (o.mSelector != null) {
                this.mSelector = new Intent(o.mSelector);
            }

            if (copyMode != COPY_MODE_HISTORY) {
                if (o.mExtras != null) {
                    this.mExtras = new Bundle(o.mExtras);
                }
                if (o.mClipData != null) {
                    this.mClipData = new ClipData(o.mClipData);
                }
            } else {
                if (o.mExtras != null && !o.mExtras.maybeIsEmpty()) {
                    this.mExtras = Bundle.STRIPPED;
                }

                // Also set "stripped" clip data when we ever log mClipData in the (broadcast)
                // history.
            }
        }
    }
 因为我们往Intent中putExtra的时候，就是put到Intent中的mExtras（Bundle）去，这里看到当mExtras不为空时，重新创建了Bundle对象，看的出来是一种深拷贝的方式（实际上不看源码也应该猜得出来）。扯远了，再次运行，发现这回显示的是"hello world"。到这里，我们可以大胆猜测：我们在NoRegisterAct中getIntent返回的是替身Intent，而不是rawIntent。
 
 	override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val textView = TextView(this)
        val data = intent.getStringExtra("data")
        val rawIntent = intent.getParcelableExtra<Intent>(AMSHookHelper.EXTRA_TARGET_INTENT)

        textView.text = if (!TextUtils.isEmpty(data)) {
            "received msg from rawIntent： ${rawIntent.getStringExtra("data")}"
            data
        } else {
            "I'm in no register act"
        }


        setContentView(textView)
    }
	
//贴图![enter image description here](https://img-blog.csdnimg.cn/20190307140545818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hsaDExOTE4NjA5Mzk=,size_16,color_FFFFFF,t_70)

从结果可以证实我们的想法是正确的。那么问题来了，明明我们在newActivity中对Intent进行了替换，为什么没有成功，但是NoRegisterAct却确确实实启动了的。最终在ActivityThread类找到了答案：

在performLaunchActivity方法中：

	private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent){
		ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }
        ...
        
        activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent/*getIntent中返回的intent*/, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
	
	}
	
可以看到，这里调用了我们熟悉的Intrumentation的newActivity方法。也就是说，一旦程序执行到这一行，我们代理类EvilInstrumentation的newActivity就会得到调用，intent还原之后，代理角色需要调用真实角色（被代理角色）的newActivity方法（也就是mBase.newActivity()那行代码），我们看Instrumentation的newActivity的实现：
	
	public Activity newActivity(ClassLoader cl, String className,
            Intent intent)	// 这个intent就是rawIntent
            throws InstantiationException, IllegalAccessException,
            ClassNotFoundException {
        String pkg = intent != null && intent.getComponent() != null
                ? intent.getComponent().getPackageName() : null;
        return getFactory(pkg).instantiateActivity(cl, className, intent);
    }
    
    public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
            @Nullable Intent intent)
            throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        return (Activity) cl.loadClass(className).newInstance();//反射创建Activity对象
    }

 可以看到，我们还原的rawIntent只是在创建Activity对象时起了作用，它作为一个局部变量，仅在方法体中起作用，并且没有传递到其他地方。而我们在Activity中getIntent方法返回的对象，是在performLaunchActivity方法中调用activity.attach()传入的，传入的intent还是我们的替身intent。
 
 现在，过程我们都理清楚了，所以建议：在创建替身Intent之时，使用clone的方式而不是new的方式。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY3NjE5MTE1OF19
-->