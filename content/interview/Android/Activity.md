---
title: "Activity"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}
## Q:简述下Activity相关生命周期
onCreate->onRestart->onStart->onRestoreInstance->onResume->onPause->onStop->onRestrt->onDestroy  onSaveInstance onRestoreInstance

- onCreate:Activity生命周期的第一个方法，也是我们在android开发中接触的最多的生命周期方法。它本身的作用是进行Activity的一些初始化工作，比如使用setContentView加载布局，对一些控件和变量进行初始化等，此时activity还在后台,不可见

- onStart:对用户可见(这个可见指的是进程的可见),在此时包含activity进入前台与用户互动之前的最后准备工作

- onResume:系统会在 Activity 开始与用户互动之前调用此回调。此时，该 Activity 位于 Activity 堆栈的顶部，并会捕获所有用户输入

- onPause:Activity 失去焦点并进入“已暂停”状态

- onStop:当 Activity 对用户不再可见时，系统会调用 onStop()

- onRestart:当处于“已停止”状态的 Activity 即将重启时，系统就会调用此回调。onRestart() 会从 Activity 停止时的状态恢复 Activity。
此回调后面总是跟着 onStart()

- onDestroy:系统会在销毁 Activity 之前调用此回调

## Q:onStart调用后,用户就真正可以看见activity了吗?
不能,onStart的可见指的是进程优先级中的可见,用户肉眼的可见需要在onResume方法执行之后再调用Activity.makeVisible()方法，我们才能真正用肉眼看到我们的DecoreView

## Q:Activity生命周期中,任务栈是怎么变化的
新Activity在onCreate时就会被移到任务栈的栈顶
## Q:onSaveInstance和onRestoreInstance的调用时机,系统会默认干些什么
- onSaveInstanceState:当用户显式关闭 Activity 时，或者在其他情况下调用 finish() 时，系统不会调用 onSaveInstanceState()。只有发生诸如配置变化等可能恢复activity的操作时,系统才会调用onSaveInstance,在onStop之前(在API28之后onSaveInstanceState()方法的执行放在了onStop()之后)默认保存view的一些瞬时信息

- onRestoreInstance:在onStart之后调用,不用判断bundle是否为空,您应始终调用 onRestoreInstanceState() 的父类实现，以便默认实现可以恢复视图层次结构的状态。


## Q:为什么不建议在onPause中处理耗时操作
因为系统在上一个activity的onPause执行结束后才会调用下一个activity的创建工作,在onPause中执行耗时操作会影响下一个actviity的启动

## Q:什么情况下activity会调用onPause
1. 来自用户的终端操作,比如来电、用户导航到另一个 Activity，或设备屏幕关闭
2. 有多个应用在多窗口模式下运行。无论何时，都只有一个应用（窗口）可以拥有焦点，因此系统会暂停所有其他应用
3. 有新的半透明 Activity（例如对话框）处于开启状态。只要 Activity 仍然部分可见但并未处于焦点之中，它便会一直暂停

## Q:activity在onDestroy中做了什么？
清理资源,

## Q:哪些是在activity清单文件配置中的必要元素,哪些是发布后不建议修改元素,为什么
清单文件中activity的唯一的必要属性是 android:name，该属性用于指定 Activity 的类名称

## Q:intent-filter过滤器是什么，有什么作用,其中各标签代表什么含义
Intent Filter（意图过滤器）其实就是用来匹配隐式Intent的，当一个意图对象被一个意图过滤器进行匹配测试时，只有三个方面会被参考到：动作、数据（URI以及数据类型）和类别
## Q:intent-filter是如何响应,然后启动activity的
PMS中的queryIntentActivities去判断启动哪个activity

## Q:activity中的权限控制是怎么样的,详细解释下
uses-permission标签声明访问activity所需要的权限
binder中的pid和uid进行权限控制
## Q:用户离开activity时,activity一定会被销毁吗？
不会,只有永久性离开时才会销毁activity

## Q:activity实例的销毁时机,系统会直接销毁activity吗
1. 当用户按下返回按钮或您的 Activity 通过调用 finish() 方法发出销毁信号时
2. 系统因系统限制（例如配置变更或内存压力）而销毁 Activity

## Q:setResult调用时机
在finish之前调用,当按下back键时生命周期调用
ActivityB.finish()->ActivityB.onBackPressed()->ActivityA.onActivityResult()->ActivityA.restart()

## Q:setResult可能会出现什么问题?如何解决？
会导致singleTask、singleInstance不起效,B设置为singleInstance,A以startActivity启动B,B会在新的task中,以startActivityForResult启动B,B不会在新的task中

## Q:在有setResult时,activity的生命周期是什么样的?
1. ActivityB 的onPause执行
2. ActivityA 的onActivityResult -> onRestart -> onResume
3. ActivityB 的onStop执行

## Q:activityA启动activityB时的生命周期?为什么要这样设计?
1. Activity A 的 onPause() 方法执行。
2. Activity B 的 onCreate()、onStart() 和 onResume() 方法依次执行（Activity B 现在具有用户焦点）。
3. 然后，如果 Activity A 在屏幕上不再显示，其 onStop() 方法执行。

这样设计是为了用户的使用体验,当onPause执行后,当前activity就会进入后台进程,资源就优先分配给后来展示的前台进程

## Q:activity状态改变
1. 配置改变,比如横竖屏切换:重建生命周期
2. 多窗口:onPause->onResume相互切换
3. Activity 或对话框显示在前台:当被覆盖的 Activity 的同一实例返回到前台时，系统会对该 Activity 调用 onRestart()、onStart() 和 onResume()。如果被覆盖的 Activity 的新实例进入后台，则系统不会调用 onRestart()，而只会调用 onStart() 和 onResume()。
4. 用户点按“返回”按钮:Activity 将依次经历 onPause()、onStop() 和 onDestroy() 回调。活动不仅会被销毁，还会从返回堆栈中移除

## Q:activity还有其他什么生命周期的函数,分别调用时机是什么时刻
* onPostCreate方法发生在onRestoreInstanceState之后，onResume之前，他代表着界面数据已经完全恢复，就差显示出来与用户交互了。在onStart方法被调用时这些操作尚未完成。

* onPostResume是在Resume方法被完全执行之后的回调。

* onContentChange是在setContentView之后的回调。


## Q:activity启动方式
- standard:默认值。系统在启动该 Activity 的任务中创建 Activity 的新实例
- singleTop:如果当前任务的顶部已存在 Activity 的实例，则系统会通过调用其 onNewIntent() 方法来将 intent 转送给该实例，而不是创建 Activity 的新实例,适合启动同类型的 Activity，例如接收通知启动的内容显示页面
- singleTask:任务栈中只有一个activity实例,默认带有clearTop效果,适合作为程序入口
- singleInstance:任务栈中有且只有一个activity实例,适合需要与程序分离开的页面，例如闹铃的响铃界面

## Q:taskAffinity是做什么用的?有什么具体的应用场景?
官方翻译过来叫做任务亲和性,可以理解为activityTask任务栈的一个别名,常与singleTask或者allowTaskReparenting属性共同使用,默认为应用包名,比如A设置启动方式为singleTask,设置taskAffinity为"com.xxx.a",则从A启动的B,任务栈也就变成了com.xxx.a

## Q:如果一个A Activity（standard）启动B Activity（singleInstance），这个时候用户点击了手机最近访问列表，然后在再点击该App所在的界面（卡片），然后这个时候点击返回键,会发生什么?为什么?
会返回桌面,因为从手机最近访问列表进入app所在卡片相当于以singleInstance方式启动了B activity,这是B所在的任务栈只有B一个activity

## Q:任务，任务栈，前台任务栈，后台任务栈，返回栈分别是什么？
![image](/activityStack.webp)

## Q:有多少种操作activity任务堆栈的方式?操作的结果是什么？
如果用户离开任务较长时间，系统会清除任务中除根 Activity 以外的所有 Activity。当用户再次返回到该任务时，只有根 Activity 会恢复
可以使用一些 Activity 属性来修改此行为：

- alwaysRetainTaskState
如果在任务的根 Activity 中将该属性设为 "true"，则不会发生上述默认行为。即使经过很长一段时间后，任务仍会在其堆栈中保留所有 Activity。
- clearTaskOnLaunch
如果在任务的根 Activity 中将该属性设为 "true"，那么只要用户离开任务再返回，堆栈就会被清除到只剩根 Activity。也就是说，它与 alwaysRetainTaskState 正好相反。用户始终会返回到任务的初始状态，即便只是短暂离开任务也是如此。
- finishOnTaskLaunch
该属性与 clearTaskOnLaunch 类似，但它只会作用于单个 Activity 而非整个任务。它还可导致任何 Activity 消失，包括根 Activity。如果将该属性设为 "true"，则 Activity 仅在当前会话中归属于任务。如果用户离开任务再返回，则该任务将不再存在

## Q:可以从后台启动activity吗?怎么做?
可以,
原生Android ROM
Android 原生 ROM 都能正常地从后台启动 Activity 界面，无论是 Android 9(直接启动) 还是 10 版本(借助全屏通知)。
定制化ROM
1. 检测后台弹出界面权限：
通过反射 AppOpsManager 相关方法检测对应 opCode 的权限；
opCode = 10021(小米机型)；
其它机型可以尝试遍历得到 opCode；
2. 通过moveTaskToFront将应用切到前台

# Q:用户在调用finish时发生了什么?
通过该应用进程ActivityThread的Handler发消息到主线程的消息队列中，然后会回调onpause，onstop，ondestroy（）。然后这个Activity的其他生命周期不会再被调用了。还会销毁该Activity的界面。触摸事件到了该应用进程主线程消息队列后（假如从该Activity返回后去到的Activity也是在同一进程的），也不会分派给这个Activity了

## Q:在不同的生命周期内调用finish的生命周期是怎么样的?
- 在onCreate中：onCreate->onDestroy
- 在onStart中：onCreate->onStart->onStop->onDestroy
- 在onResume中：onCreate->onStart->onResume->onPause->onStop->onDestroy


## Q:finish调用后会立刻调用onStop、onDestroy吗？不会的话,为什么?
1. 不一定会立刻调用
调用finish后会将当前activity加入到ActivityStackSuperVisor的mStopptingActivities集合中,在下一个要启动的activity执行onResume时
会通过idleHandler去调用ActivityStackSuperVisor中处于mStopptingActivities里的activity,所以当要启动的activity在主线程做了很多耗时操作时,就会导致上一个activity的onStop、onDestroy无法立即执行
2. 延迟10s是因为idleHandler的默认延时时长是10s,超过10s就会主动调用上一个activity的onStop

## Q:如果App还存在缓存进程，这个时候启动App，应用Application的onCreate方法会执行吗？
不会,因为从缓存进程启动App，系统已经缓存了很多信息，很多数据并不会被销毁，onCreate中初始化的那些内容还在，方便用户下次快速启动

## Q:启动一个Dialog,activity生命周期是如何变化的?为什么?
生命周期回调都是 AMS 通过 Binder 通知应用进程调用的；而弹出 Dialog、Toast、PopupWindow 本质上都直接是通过 WindowManager.addView 显示的（没有经过 AMS），所以不会对生命周期有任何影响。


## Q:onSaveInstance是如何实现保持应用状态的?如果在里面传递大量数据会发生什么?
1. 在ActivityThread中会创建一个用于保持状态信息的bundle
2. 在Activity被回收时，会触发一个SaveState的事件。
3. 跟其他的事件一样，SaveState事件从Activity->Window->View传递到最大的View，然后遍历View树保存状态
4. 状态保存在一个SparseArray中，以View的ID作为key。
5. 自定义View可以重载onSaveInstanceState()来保存自己的状态，参考TextView的实现方法
6. 在activity进入stop时,会像ActivityManager发出申请,用来保存activity状态,此时就涉及到Binder 传输做一个跨进程的通信将 Bundle 的数据传递给 ActivityManager

传递大数据会报异常



## Q:onResume执行了就可以认为界面对用户可见了?这种理解对吗?
这种理解是错误的,onResume只是系统执行了makeVisiable,而何时将界面绘制完成需要系统具体的绘制后才会知道,当系统回调onWindowFocusChanged=true时才证明界面真正的对用户可见了

## Q:一个activity从创建到用户可见,经历了什么操作,涉及到了哪些生命周期?
onCreate - onStart - onResume - measure - layout -measure - layout - draw - onWindowFocusChanged
1. 在onCreate阶段通过setContentView，布局被包装为PhoneWindow内部的DecorView对象
2. 在ActivityThread.handleResumeActivity阶段，执行完performResume后通过WindowManagerImpl的addView，DecorView被 setView 到 ViewRootImpl
3. ViewRootImpl.setView 方法里通过 requestLayout - scheduleTraversals 向 Choreographer 请求安排绘制任务
4. Choreographer收到VSYNC信号回调到ViewRootImpl的performTraversals对DecorView进行measure、layout、draw，其中由于首次performTraversals会涉及到初始化EGL，所以最终会执行两次该方法，因此decorView会经历：measure - layout - measure - layout draw


## Q:什么情况下startActivity需要设置Flag=NEW_TASK?为什么?不设置的话会发生什么?
1. 非activity情况下启动新的activity，比如application和service,
2. 用一个非 Activity 的 Context 去启动一个新的 Activity 的话，新的 Activity 并不知道自己应该放到哪个 Activity 栈中。而设置上 FLAG_ACTIVITY_NEW_TASK 标记，就会直接创建一个 Activity 栈来管理它了。实际上，这样的启动方式就是以 singleTask 模式启动的。
3. Instrumentation.execStartActivity 执行跳转之前，我们有一个判断条件，当这个条件不成立的时候，会直接抛出运行时异常。

## Q:activity从启动到显示,都经历了什么?
![image](/activityToViewShow.webp)

## Q:activity在屏幕旋转后如何保持下载不中断?
Fragment设置setRetaineInstance保持下载操作


## Q:startActivities是干什么用的?activity栈是如何变化的?
进栈顺序为A->B->C，使用startActivities会直接跳转到C,返回时会依次返回C->B-A

## Q:Activity启动模式与FLAG结合?
* FLAG_ACTIVITY_SINGLE_TOP 
设置此flag时，当被启动的activity已经位于当前栈的顶部时，则不会新建Activity。

* FLAG_ACTIVITY_NEW_TASK 
此flag是会让Activity在新的栈中启动。但是单独使用此flag时会有较多意想不到的情况发送
使用场景：在非Activity中启动Activity需要强制加上此flag
单独使用此flag的情形： 
  - Activity的taskAffinity属性的Task栈是否存在 
  - 如果存在，要看Activity是否存已经存在于该Task 
  - 如果已经存在于该taskAffinity的Task，要看其是不是其rootActivity
  - 如果是其rootActivity，还要看启动该Activity的Intent是否跟当前intent相等

* FLAG_ACTIVITY_CLEAR_TASK 
此flag需要与FLAG_ACTIVITY_NEW_TASK结合使用。使用时会在Activity启动之前将task中其它Activity销毁(无论其它Activity设置了何种启动模式)

* FLAG_ACTIVITY_CLEAR_TOP
使用此flag时，如果当前task中存在待启动Activity的实例，则会清空此task中待启动activity及以上的activity

## Q:Flag使用具体使用?
1. 测试步骤:MainActivity和SercondActivity处在不同task中,使用FLAG_ACTIVITY_NEW_TASK启动SecondActivity
MainActivity-->SecondActivity-->MainActivity-->SecondActivity
结果:当第二次尝试进入SecondActivity中时，会发现没有任何变化，仍然停留在MainActivity中
原因分析:因为此时SecondActivity实例已经存在，但是它所在的task的栈顶是ActivityTest；而单独的添加FLAG_ACTIVITY_NEW_TASK又不会"删除task中位于SecondActivity之上的Activity实例"，所以就没有发生跳转(onNewIntent也没有回调)。这种情况下只会将整个栈移动到前台，且栈中的状态不会改变。

2. 测试步骤:MainActivity和SercondActivity处在同一个or不同task中,使用FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_CLEAR_TOP启动SecondActivity
结果:两个SecondActivity为不同的实例
原因分析:在第二次启动SecondActivity时，会将SecondActivity及以上的activity清空，然后finish并re-create SecondActivity

3. 测试步骤:SecondActivity与MainActivity在同一个task中,使用FLAG_ACTIVITY_SINGLE_TOP和FLAG_ACTIVITY_CLEAR_TOP启动SecondActivity
MainActivity-->SecondActivity-->MainActivity-->SecondActivity
结果:两个SecondActivity为相同的实例
原因分析:第二次启动SecondActivity时，在当前的task中已经存在SecondActivity的实例，所以第二次启动时，SecondActivity不会被重建，而只会回调onNewIntent方法

4. 测试步骤:SecondActivity和MainActivity在不同task中,使用FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_CLEAR_TASK启动SecondActivity
结果:两个SecondActivity为不同实例
原因分析:clearTask会清除task上其他activity

5. 测试步骤:SecondActivity使用singleInstance启动,MainActivity-->SecondActivity-->MainActivity-->SecondActivity
结果:两个activity在不同task中,第二次SecondActivity与第一次是同一个,会调用onNewIntent方法

## Q:子线程可以startActivity吗?会有什么问题?
可以,暂时没有发现问题
