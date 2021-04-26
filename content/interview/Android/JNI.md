---
title: JNI
weight: -20
---

{{< toc >}}

## JNI加载流程
* Gradle调用外部构建脚本CMklist.txt
* CMake按照脚本中的命令将C++源文件native-lib.cpp编译成native-lib.so动态库,并把它编译到APK中
* 运行时首先加载native-lib.so,然后调用相应的native方法

## JNI环境配置
在Gradle配置文件俩个地方配置externalNativeBuild {}，一个在defaultConfig里面一个在defaultConfig外面
* 在defaultConfig外面的externalNativeBuild {}代码块用cmake制定了CMakeList.txt的路径
* 在defaultConfig里面的externalNativeBuild {}代码块用cmake主要是填写CMake命令参数

## CmakeList文件解析
* #配置加载头文件
    include_directories(./src/main/cpp/include)
    file(GLOB main_src "src/main/cpp/*.cpp") //定义main_src变量
* add_library(native-lib SHARED native-lib.cpp) 可以定义多个库，CMake会构建他们，Gradle将自动把共享库打入APK中
    >- 第一个参数：设置这个库的名字
    >- 第二个参数：设置库的类型，SHARE 动态库.so后缀 ,STATIC 静态库 .a后缀
    >- 第三个参数：源文件的相对路径
* find_library(log-lib log) 找到一个NDK库，并且把这个库的路径储存在一个变量中
    >- 第一个参数：变量名
    >- 第二个参数：使用的NDK原生库名称
* target_link_libraries(native-lib ${log-lib})关联库，将指定的库关联起来
    >- 第一个参数：目标库
    >- 第二个参数：将目标库链接到日志库包含在NDK中。
* set_target_properties(库名称 PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI}/xxx.so)

## JNI作用？
* 调用java函数：使用JNIEnv调用java中的代码
* 操作java对象：java对象传入jni层是jstring，可以使用JNIEnv来操作这个对象

## JNI中注册方法的方式?
* 静态注册
    >- Java+包名+类名+方法名
* 动态注册
    * 加载动态库
        >- java层通过System.loadLibrary()方法可以加载一个动态库,此时虚拟机就会调用jni库中的JNI_OnLoad()函数
    * 注册函数
        >- JNI_OnLoad()函数经常用来做一些初始化操作，动态注册就是在这里进行的,动态注册通过_JNIEnv的RegisterNatives()函数来完成
        ```c++
        jint RegisterNatives(JNIEnv *env, jclass clazz, const JNINativeMethod *methods, jint nMethods);
        //第一个参数：JNIEnv *指针
        //第二个参数：代表一个java类
        //第三个参数：代表JNINativeMethod结构体数组，JNINativeMethod定义了java层函数和native层函数的映射关系
        //第四个参数:代表第三个参数methods数组的大小
        //return 0代表成功，负值代表失败
        ```
    * 例子
    ```c++
    jint native_text(JNIEnv *env, jobject jobject1, jstring msg) {
        return 0;
    }
    jint native_staic_text(JNIEnv *env, jobject jclass1, jstring meg) {
        return 0;
    }
    static const JNINativeMethod nativeMethod[] = {
        {"text",        "(Ljava/lang/String;)I", (void *) native_text},
        {"static_text", "(Ljava/lang/String;)I", (void *) native_staic_text}
    };  
    static int registNativeMethod(JNIEnv *env) {
        int result = -1;
        jclass class_text = env->FindClass("com.text.ndk1.TextJni");//java层调用System.loadLibrary的地方
        if (env->RegisterNatives(class_text, nativeMethod,
                             sizeof(nativeMethod) / sizeof(nativeMethod[0])) == JNI_OK) {
            result = 0;
        }
        return result;
    }
    jint JNI_OnLoad(JavaVM *vm, void *reserved) {
        JNIEnv *env = NULL;
        int result = -1;
        if (vm->GetEnv((void **) &env, JNI_VERSION_1_1) == JNI_OK) {
            if (registNativeMethod(env) == JNI_OK) {
                result = JNI_VERSION_1_6;
            }
            return result;
        }
    }
    ```
## JNI访问java对象
```c++
//获取class对象
jclass jclass_student = env->GetObjectClass(jobject1);
//从class中获取变量
jfieldID jfieldId = env->GetFieldID(jclass_student, "name", "Ljava/lang/String;");
//获取class对象中的print方法
jmethodID print = env->GetMethodID(jclass_student, "print", "()V");
//调用java对象中的print方法
env->CallVoidMethod(jobject1, print);
```

## JNI中字符串操作
Java默认使用unicode编码,C/C++使用utf编码
* 对于小字符串来说，GetStringRegion 和 GetStringUTFRegion 这两对函数是最佳选择，因为缓冲区可以被编译器提前分配，而且永远不会产生内存溢出的异常。当你需要处理一个字符串的一部分时，使用这对函数也是不错。因为它们提供了一个开始索引和子字符串的长度值。另外，复制少量字符串的消耗 也是非常小的。
* 使用 GetStringCritical 和 ReleaseStringCritical 这对函数时，必须非常小心。一定要确保在持有一个由 GetStringCritical 获取到的指针时，本地代码不会在 JVM 内部分配新对象，或者做任何其它可能导致系统死锁的阻塞性调用。它会触发GC的暂停,之后任何会导致GC的操作都会阻塞
* 获取 Unicode 字符串和长度，使用 GetStringChars 和 GetStringLength 函数。
* 获取 UTF-8 字符串的长度，使用 GetStringUTFLength 函数。
* 创建 Unicode 字符串，使用 NewString 函数。
* 创建 UTF-8 字符串,使用NewStringUTF 函数
* 从 Java 字符串转换成 C/C++ 字符串，使用 GetStringUTFChars 函数。
* 通过 GetStringUTFChars、GetStringChars、GetStringCritical 获取字符串，这些函数内部会分配内存，必须调用相对应的 ReleaseXXXX 函数释放内存。

## JNI中的引用
* 局部引用
>- 通过NewLocalRef创建,会阻止GC回收所引用的对象，不在本地函数中跨函数使用，不能跨线程使用。函数返回后局部引用所引用的对象会被JVM自动释放，或调用DeleteLocalRef释放,本地方法返回到Java层之后，如果Java层没有对返回的局部引用使用的话，局部引用就会被JVM自动释放
>>- 为什么建议手动调用DeleteLocalRef是否释放局部引用
>>>- 1、JNI会将创建的局部引用都存储在一个局部引用表中，如果这个表超过了最大容量限制，就会造成局部引用表溢出，使程序崩溃。经测试，Android上的JNI局部引用表最大数量是512个。当我们在实现一个本地方法时，可能需要创建大量的局部引用，如果没有及时释放，就有可能导致JNI局部引用表的溢出，所以，在不需要局部引用时就立即调用DeleteLocalRef手动删除
>>>- 2、在编写JNI工具函数时，工具函数在程序当中是公用的，被谁调用你是不知道的。所以在工具函数中使用完局部引用后，需要调用DeleteLocalRef删除。
>>>- 3、如果你的本地函数不会返回。比如一个接收消息的函数，里面有一个死循环，用于等待别人发送消息过来while(true) { if (有新的消息) ｛ 处理之。。。。｝ else { 等待新的消息。。。}}。如果在消息循环当中创建的引用你不显示删除，很快将会造成JVM局部引用表溢出。
>>>- 4、局部引用会阻止所引用的对象被GC回收。比如你写的一个本地函数中刚开始需要访问一个大对象，因此一开始就创建了一个对这个对象的引用，但在函数返回前会有一个大量的非常复杂的计算过程，而在这个计算过程当中是不需要前面创建的那个大对象的引用的。但是，在计算的过程当中，如果这个大对象的引用还没有被释放的话，会阻止GC回收这个对象，内存一直占用者，造成资源的浪费。所以这种情况下，在进行复杂计算之前就应该把引用给释放了，以免不必要的资源浪费

>>- 局部引用管理:JNI提供了一系列函数来管理局部引用的生命周期。这些函数包括：EnsureLocalCapacity、NewLocalRef、PushLocalFrame、PopLocalFrame、DeleteLocalRef
>>>- EnsureLocalCapacity  确保函数能创建len个局部引用
>>>- PushLocalFrame 为当前函数中需要用到的局部引用创建了一个引用堆栈,当返回一个局部引用时，JVM会自动将该引用压入当前局部引用栈中
>>>- PopLocalFrame  销毁栈中所有的引用,这样就不用一个个调用DeleteLocalRef去删除引用了

* 全局引用
>- 调用NewGlobalRef基于局部引用创建，会阻GC回收所引用的对象。可以跨方法、跨线程使用。JVM不会自动释放，必须调用DeleteGlobalRef手动释放
* 弱全局引用
>- 调用NewWeakGlobalRef基于局部引用或全局引用创建，不会阻止GC回收所引用的对象，可以跨方法、跨线程使用。引用不会自动释放，在JVM认为应该回收它的时候（比如内存紧张的时候）进行回收而被释放。或调用DeleteWeakGlobalRef手动释放。