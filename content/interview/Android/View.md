---
title: "View"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}
## Q:View.getContext()一定会返回activity对象吗?
并不是,有这么几种情况:
1. View.inflate的时候手动传的不是activity
2. 当使用AppCompatActivity时,context是TintWrapContext

## Q:View.inflate和LayoutInflater.inflate方法区别？
View.inflate是对LayoutInflater的封装,其内部就是调用的LayoutInflater.inflate
创建view的方法是LayoutInflater的createViewFromTag方法。会依次调用mFactory2、mFactory和mPrivateFactory三者之一的onCreateView方法去创建一个View。如果不存在Factory，则调用LayoutInflater自身的onCreateView或者createView来实例化View

## Q:Inflate的三个参数(int resource, ViewGroup root, boolean attachToRoot),组合设置后分别有什么效果?
1. 如果root为null，attachToRoot将失去作用，则resource布局文件最外层设置任何值都没有意义，仅仅是解析布局文件的子View。
2. 如果root不为null，attachToRoot设为true，则会给布局文件指定一个父布局，即root。（merge作为父布局标签为什么需要attachToRoot设为true的原因）,不需要调用addView
3. 如果root不为null，attachToRoot设为false，则会将布局文件最外层的所有layout属性进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。需要调用addView
4. 在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true；如果root为null，attachToRoot参数为false。

## Q:子线程中可以更新UI吗?为什么?
可以,线程检查是在ViewRootImpl的checkThread()中。ViewRootImpl的初始化是在Activity的onResume()方法之后。因此，如果有子线程在onResume之前更新UI是可以成功的。当然还有一种Hook ViewRootImpl的mThread的方法也可以更新UI

## Q:View坐标体系是什么?
![image](/view坐标.png)

## Q:为什么setTranslation会移动View的位置?
因为view的getX或者getY中会通过mTop + getTranslation来确定最终的X或者Y

## Q:为什么View移动的位置跟mScrollX，mScrollY是相反的?
因为view在draw的时候,会对mScrollX，mScrollY进行取反

## Q:都有什么获取view区域的方法?怎么用?
![image](/获取view区域的各方法.png)
前置条件:
View 长宽：
width:600
height:200

View 四个顶点：
mLeft = -200
mTop = 200
mRight = 400
mBottom = 400;

1. 子布局可以超过父布局展示 clipChildren=false

getLocationInWindow:View距离Window左上角坐标，因为View超出了Window，因此获取的坐标为[x,y]=[-100,400]

getLocationOnScreen:View距离屏幕左上角的坐标，在getLocationInWindow 基础上加上Window 的偏移。[x,y]=[-100, 400] + [200, 100] = [100, 500]

getGlobalVisibleRect:View的可见部分在Window里的区域，View的真实区域：白色 + 红色 部分，只是白色部分超出了Window，不会展示，可见区域是红色部分。rect=[0, 400, 500, 600] 左上右下

getLocalVisibleRect:View可见部分相对于自身的区域，也就是说自身的哪些区域可见。在getGlobalVisibleRect基础上，不断查找。rect=[100, 0, 600, 200] 

getHitRect:获取View有效的点击区域，以四个顶点为基础，考虑matrix，得出结果如下：rect=[-200,200,400, 400]

getDrawingRect:获取View的绘制区域，以四个顶点为基础，考虑scroll值，得出结果如下：rect=[0,0,600,200]

2. 子布局不可以超过父布局展示 clipChildren=true

getGlobalVisibleRect:可以看出，红色部分为可见区域，那么该区域相对Window左上角的距离为：rect=[100,400,500,600]

getLocalVisibleRect:红色部分在View自身里的区域:rect=[200,0,600,200]

## Q:View的绘制流程分几步，从哪开始？哪个过程结束以后能看到view？
从ViewRoot的performTraversals开始，经过measure，layout,draw 三个流程。draw流程结束以后就可以在屏幕上看到view了。

## Q:View什么时候发生绘制?
* 由于onCreate会先于handleResumeActivity执行，我们在onCreate中调用了setContentView，也只是生成DecorView并给这个DecorView的内容设置了布局而已，此时还并没有把这个DecorView添加到Window中，同样，WMS中也还没有这个Window，所以此时并不能做任何事情（绘制、接收点击事件等），虽然调用了requestLayout和invalidate，并不会真正触发布局和重绘（因为还没有与ViewRootImpl进行绑定）
* Activity与Window产生联系，是在调用activity#attach方法中，生成了一个PhoneWindow，并把这个activity对象自身，设置给了Window的Callback回调（Activity实现了Window的Callback接口）
* Window与WindowManagerService产生联系，是在handleResumeActivity中，先执行了onResume方法，通过调用WindowManagerImpl#addView方法将这个Activity对应的DecorView添加到了这个Window中，addView方法是一个IPC操作，将这个Window也添加到了WindowManagerService中；
* ViewRootImpl与Window产生联系，是在WindowManagerImpl#addView方法中，这个过程中会new一个ViewRootImpl与DecorView相对应，保存在WindowManagerGloable中；
* 总的来说，就是setContentView生成了DecorView及其视图，在onResume之后才把这个视图添加进了Window和WMS中，具备了交互能力

## Q:ViewGroup有onMeasure方法吗？为什么?
没有，这个方法是交给子类自己实现的。不同的viewgroup子类 肯定布局都不一样，那onMeasure索性就全部交给他们自己实现好了

## Q:setContentView之后发生了什么？
![image](/setContentView.png)

## Q:setContentView可以在其他生命周期内执行吗?为什么?
可以,因为Activity.setContentView只是api。实际操作是去交给了PhoneWindow去做的，可以在onContentChanged中进行Activity View成员变量的初始化操作(findViewById等)。因为setContentView是同步操作，所以直接在setContentView之后findViewById也没什么问题

## Q:view、drawable、bitmap之间的关系?
bitmap： 仅仅就是一个位图 你可以理解为一张图片在内存中的映射。 就这么简单。这个很多人都知道

view： view最大的作用是2个  一个是draw 也就是canvas的draw方法，还有一个作用 就是测量大小。

drawable： 他其实本身和bitmap没有关系, 你可以把他理解为是一个绘制工具，和view的第一个作用是一摸一样的，你能用view的canvas 画出来的东西 你用drawable 一样可以画出来，  不一样的是drawable 仅仅能绘制，但是不能测量自己的大小，但是view可以。
换句话说 drawable 承担了view的一半作用

## Q:有了view，为什么还要设计drawable?
假设你要自定义 一组view，注意是一组，不是一个， 那你就可以把这一组view中 共同的部分 抽成一个drawable ，这样view就可以复用 这个drawble了，不用写重复的 canvas.draw 方法 

## Q:如何设计一个曝光系统?需要注意些什么?
1. 要确定什么样的算有效曝光（在屏幕停留时间超过一个值如2秒）可以通过attachToWindow、detachToWindow记录时间
2. 监听到每个view移入和移出屏幕的事件   View.getLoaclRect

## Q:有用过LayoutInflater.Factory吗？都有哪些应用场景?
系统通过Factory提供了一种hook的方法，方便开发者拦截LayoutInflater创建View的过程。应用场景包括
1. 在XML布局中自定义标签名称；
2. 全局替换系统控件为自定义View； 
3. 替换app中字体；
4. 全局换肤


## Q:SurfaceView是什么?有哪些应用场景?

## Q:一个viewGroup 绘制了4-5个view，当其中一个子view。例如背景变了。就会导致整个viewGroup的刷新。。请问有没有办法？仅仅只让对应的view更改UI。避免让viewGroup重新测量 绘制?
调用view的invalidate方法，执行view的重新绘制

## Q:在绘制阶段，ViewGroup 不光要绘制自身，还需循环绘制其一众子 View，这个绘制策略默认为顺序绘制，即 [0 ~ childCount)。这个默认的策略，有办法调整吗？修改了之后，事件分发需要特殊处理吗？还是需要特殊处理?getChildAt会有相应变化吗?
可以调整,
1. 通过setChildrenDrawingOrderEnabled(true)来开启自定义顺序；
2. 重写getChildDrawingOrder方法来决定什么时候要返回哪个子View；
其中RecyclerView还可以通过一个setChildDrawingOrderCallback方法来动态指定顺序，而不用重写RecyclerView

自定义DrawingOrder，只是改变事件分发和子View绘制的顺序，ViewGroup内部mChildren数组的排序是不会变的，所以不要认为通过修改DrawingOrder也能同时改变getChildAt方法返回的View对象。

## Q:通过ViewTreeObserver都可以监听什么回调?原理是什么?
视图即将绘制,视图布局发生变化,原理就是时间发生时会系统发出回调

## Q:你知道detachViewFromParent/attachViewToParent 这一组方法在哪些控件中被使用中？detachViewFromParent/attachViewToParent 与 removeView/addView 有什么区别呢？detachViewFromParent/attachViewToParent在什么场景下非常适合使用？
addView和removeView方法，操作容器内的子视图数组，触发视图重绘制，触发子视图attach和detached窗体回调。
addViewInLayout和removeViewInLayou方法，与上面一样，只是不会重绘视图。
attachViewToParent和detachViewFromParent方法，只会操作容器内的子视图数组,recyclerview中

## Q:requestLayout和invalidate的区别?都会执行到哪些生命周期?
![image](/requestLayout&invalidate.jpeg)
* 调用requestlayout，会导致自己的View设置两个标志位，并且向上调用父类的requestlayout并且设置标志位，不断向上传递，最后上传到ViewRootImp执行performTraversals执行measure,layout(在三大流程中检查标志位)（不一定执行onDraw,没有设置PFLAG_DIRTY_MASK标志位）
* 调用Invalidate方法给该View设置PFLAG_DIRTY标志位，然后不断计算在父容器中的区域向上传递(设置标志位)，最终传递到RootViewImp执行scheduleTraversals然后执行performTraversals，然后measure,layout,draw三个方法，由于没有设置PFLAG_FORCE_LAYOUT标志位，只能执行draw方法，前面两个不执行(不是不执行，只是不执行特有的方法)

## Q:ViewStub是什么?有什么应用场景?原理是什么?
ViewStub是一个不可见的，宽高为0的View，可用于在程序运行的时候延迟  加载布局资源的（用于实现布局资源的“懒加载”),当使ViewStub可见或者调用inflate方法，可以使布局资源被加载！
初始化时setWillNotDraw(true),onMeasure时设置宽高为0，onDraw中不进行任何绘制
* inflateViewNoAdd：获取到布局渲染器将真正需要展示的布局文件渲染成View并且给返回。
* replaceSelfWithView：将ViewStub从布局文件结构中移除，同时把渲染好的View添加到ViewStub之前所处的位置
* 之后把渲染好的View的弱引用给存储起来。方便在setVisibility()方法中使用。


## Q:系统可以在子线程中访问UI吗？为什么这样设计?
不可以,这样设计是因为Android的UI控件不是线程安全的，如果在多线程中并发访问可能会导致UI控件处于不可预期的状态,UI不搞成加锁的是因为
①首先加上锁机制会让UI访问的逻辑变得复杂  
②锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行。 所以最简单且高效的方法就是采用单线程模型来处理UI操作。

## Q:为什么Toast和showDialog可以在子线程显示?需要做些什么?
Toast\Dialog本质是通过window显示和绘制的（操作的是window），而主线程不能更新UI 是因为ViewRootImpl的checkThread方法在Activity维护的View树的行为
Toast中TN类使用Handler是为了用队列和时间控制排队显示Toast,所以需要Looper.prepare和Looper.loop


## Q:View.post为何可以获取宽高信息?
如果在 performTraversals 前调用 View.post，则会将消息进行保存，之后在 dispatchAttachedToWindow 的时候通过 ViewRootImpl 中的 Handler 进行调用。
如果在 performTraversals 以后调用 View.post，则直接通过 ViewRootImpl 中的 Handler 进行调用。

