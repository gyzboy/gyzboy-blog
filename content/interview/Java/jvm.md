---
title: "JVM"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}
## Q:jvm的内存模型是啥样的?
![image](/jvm_memory.png)
![image](/object_alloc.webp)
1. 程序计数器:在jvm中，它就是程序控制流的指示器，循环，跳转，异常处理，线程的恢复等工作都需要依赖程序计数器去完成,此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemotyError情况的区域
2. 虚拟机栈:在每个方法执行的时候，jvm都会同步创建一个栈帧去存储局部变量表，操作数栈，动态连接，方法返回值等信息。一个方法的生命周期就贯彻了一个栈帧从入栈到出栈的全部过程。局部变量表应该是我们接触的最多的，里面存储了java的8大基本数据类型（byte、short、char、int、float、long、double、boolean）、对象引用(reference类型，不是对象本身，是指向对象的引用)和returnAddress类型（指向一条字节码指令的地址），
3. 本地方法栈:native方法栈
4. 堆:所有线程共享的一块内存区域，用于存放几乎所有的对象实例和数组，TLAB是线程私有的，在堆空间中分配，对象会首先存放在这个线程私有的TLAB中，可以提升线程分配的效率
5. 方法区:方法区也是所有线程共享的区域，储存虚拟机加载的类信息，常量，静态变量，编译后的代码。运行时常量池就是方法区的一部分，代表运行时每个class文件中的常量表。包括几种常量：编译时的数字常量、方法或者域的引用,Java语言并不要求常量一定只有编译期才能产生，也就是可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法

## Q:类的加载过程?
![image](/class_load.png)
加载、验证、准备、初始化、卸载这五个阶段的顺序是确定的，是依次有序的。但是解析阶段有可能会在初始化之后才会进行，这是为了支持java动态绑定的特性
- 加载
    * 通过一个类的全限定名获取定义此类的二进制字节流给了开发人员很大的灵活性,比如可以通过ZIP、网络、动态代理等技术加载指定二进制流
- 准备
    * 正式为类变量（static变量）分配内存并设置初始值,此时变量所使用的内存，将在**方法区**分配,并且初始值为默认值

## Q:Class.forName() 和ClassLoader.loadClass()区别？实际开发你用那种，为什么？
- Class.forName() 和ClassLoader.loadClass()区别？
    - Class.forName() 默认执行类加载过程中的连接与初始化动作，一旦执行初始化动作，静态变量就会被初始化为程序员设置的值，如果有静态代码块，静态代码块也会被执行
    - ClassLoader.loadClass() 默认只执行类加载过程中的加载动作，后面的动作都不会执行

## Q:说一下对象的创建过程？变量创建过程种放在虚拟机哪里？
- 说一下对象的创建过程？比如：Dog dog= new Dog()；
    - 当虚拟机执行到new指令时，它先在常量池中查找“Dog”，看能否定位到Dog类的符号引用；如果能，说明这个类已经被加载到方法区了，则继续执行。如果没有，就让Class Loader先执行类的加载。
    - 然后，虚拟机开始为该对象分配内存，对象所需要的内存大小在类加载完成后就已经确定了。这时候只要在堆中按需求分配空间即可。具体分配内存时有两种方式，第一种，内存绝对规整，那么只要在被占用内存和空闲内存间放置指针即可，每次分配空间时只要把指针向空闲内存空间移动相应距离即可，当某对象被GC回收后，则需要进行某些对象内存的迁移。第二种，空闲内存和非空闲内存夹杂在一起，那么就需要用一个列表来记录堆内存的使用情况，然后按需分配内存。
    - 对于多线程的情况，如何确保一个线程分配了对象内存但尚未修改内存管理指针时，其他线程又分配该块内存而覆盖的情况？有一种方法，就是让每一个线程在堆中先预分配一小块内存（TLAB本地线程分配缓冲），每个线程只在自己的内存中分配内存。但对象本身按其访问属性是可以线程共享访问的。
    - 内存分配到后，虚拟机将分配的内存空间都初始化为零值(不包括对象头)。实例变量按变量类型初始化相应的默认值（数值型为0，boolan为false），所以实例变量不赋初值也能使用。接着设置对象头信息，比如对象的哈希值，GC分代年龄等。
    - 从虚拟机角度，此时一个新的对象已经创建完成了。但从我们程序运行的角度，新建对象才刚刚开始，对象的构造方法还没有执行。只有执行完构造方法，按构造方法进行初始化后，对象才是彻底创建完成了。构造函数的执行还涉及到调用父类构造器，如果没有显式声明调用父类构造器，则自动添加默认构造器。
    - new运算符可以返回堆中这个对象的引用
- 变量创建过程种放在虚拟机哪里？
    - 变量是实例变量、局部变量或静态变量的不同将引用放在不同的地方：
        - 如果dog局部变量，dog变量在栈帧的局部变量表，这个对象的引用就放在栈帧。
        - 如果dog是实例变量，dog变量在堆中，对象的引用就放在堆。
        - 如果dog是静态变量，dog变量在方法区，对象的引用就放在方法区。

## Q:什么情况下会触发类的初始化?
* 遇到 new、getstatic、putstatic 或 invokestatic 这 4 条字节码指令；
* 使用 java.lang.reflect 包的方法对类进行反射调用的时候；（这里可能就会出现面试题，反射的缺点是什么）
* 当初始化一个类的时候，发现其父类还没有进行初始化的时候，需要先触发其父类的初始化；
* 当虚拟机启动时，用户需要指定一个要执行的主类，虚拟机会先初始化这个类；
* 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果 REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有初始化。
* 使用java8新加入的default默认方法，如果这个接口的实现类发生了初始化，那么该接口要在其之前被初始化。

## Q:什么是双亲委派模型?有什么优势?
双亲委派模型就是说一个类加载器收到了类加载的请求，不会自己先加载，而是把它交给自己的父类去加载，层层迭代,优势就是所有的类都会交给顶层父类加载器去实现,这样就实现了jvm中类的唯一性
* 避免重复加载，如果已经加载，就不需要再次加载
* 安全，如果你定义String类来代替系统的String类，这样会导致风险，但是在双亲委托模型中，String类在java虚拟机启动时就被加载了，你自定义的String类是不会被加载的
```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException{
            // 首先检查类是否被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {//先用父加载器加载
                        c = parent.loadClass(name, false);
                    } else {//没有父加载器了用启动加载器加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    //如果父类抛出ClassNotFoundException异常
                    //则说明父类不能加载该类
                }

                if (c == null) {
                    //如果父类无法加载，则调用自身的findClass进行加
                    c = findClass(name);
                }
            }
            return c;
    }
```

## Q:如何自定义一个类加载器?
* 定义一个类并继承ClassLoader
* 重写findClass方法，并且在findClass方法中调用defineClass方法
    - byte[] data = loadClassData(name);//将文件内容转换为字节流
    - defineClass(name, data, 0, data.length)//将字节流转换为class实例

## Q:如何使用自定义类加载器加载文件?
1. CustomClassLoader customClassLoader = new CustomClassLoader(filePath);//传入class文件路径
2. Class<?> aClass = customClassLoader.findClass("com.gyz.Hello");//生成指定包名class文件
3. 通过反射调用方法

## Q:为什么会出现破坏双亲委派的模型？是解决了什么问题？

## Q:基本数据类型一定存储在栈中吗？
- 首先说明，"java中的基本数据类型一定存储在栈中的吗？”这句话肯定是错误的。
- 基本数据类型是放在栈中还是放在堆中，这取决于基本类型在何处声明，下面对数据类型在内存中的存储问题来解释一下：
  - 一：在方法中声明的变量，即该变量是局部变量，每当程序调用方法时，系统都会为该方法建立一个方法栈，其所在方法中声明的变量就放在方法栈中，当方法结束系统会释放方法栈，其对应在该方法中声明的变量随着栈的销毁而结束，这就局部变量只能在方法中有效的原因
    - 在方法中声明的变量可以是基本类型的变量，也可以是引用类型的变量。当声明是基本类型的变量的时，其变量名及值（变量名及值是两个概念）是放在JAVA虚拟机栈中。当声明的是引用变量时，所声明的变量（该变量实际上是在方法中存储的是内存地址值）是放在JAVA虚拟机的栈中，该变量所指向的对象是放在堆类存中的。
  - 二：在类中声明的变量是成员变量，也叫全局变量，放在堆中的（因为全局变量不会随着某个方法执行结束而销毁）。同样在类中声明的变量即可是基本类型的变量，也可是引用类型的变量
    - 当声明的是基本类型的变量其变量名及其值放在堆内存中的。引用类型时，其声明的变量仍然会存储一个内存地址值，该内存地址值指向所引用的对象。引用变量名和对应的对象仍然存储在相应的堆中

## Q:泛型是什么?
泛型就是为了参数化类型，或者说可以将类型当作参数传递给一个类或者是方法,好处就是它不再需要对取出来的结果进行强制转换了
出于规范的目的，Java 还是建议我们用单个大写字母来代表类型参数。常见的如：
```java
T 代表一般的任何类。
E 代表 Element 的意思，或者 Exception 异常的意思。
K 代表 Key 的意思。
V 代表 Value 的意思，通常与 K 一起配合使用。
S 代表 Subtype 的意思，文章后面部分会讲解示意
```

## Q:泛型通配符?
- <?>被称作无限定的通配符。
- <? extends T>被称作有上限的通配符。
- <? super T>被称作有下限的通配符。

## Q:泛型擦除是什么?存在什么问题?有哪些补救方案?应用场景是哪些？
```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();
		
System.out.println(l1.getClass() == l2.getClass());
// true
```
在泛型类被类型擦除的时候，之前泛型类中的类型参数部分如果没有指定上限，如 <T>则会被转译成普通的 Object 类型，如果指定了上限如 <T extends String>则类型参数就被替换成类型上限
类型擦除，是泛型能够与之前的 java 版本代码兼容共存的原因,用反射的手段就绕过了正常开发中编译器不允许的操作限制

## Q：泛型和反射有何联系？使用反射来获取泛型信息？getType和getGenericType有何区别？
- 通过反射中使用泛型，可以避免使用反射生成的对象需要强制类型转换
- 泛型的好处众多，最主要的一点就是避免类型转换，防止出现ClassCastException，即类型转换异常。
    ```
    public class ObjectFactory {
        public static <T> T getInstance(Class<T> cls) {
            try {
                // 返回使用该Class对象创建的实例
                return cls.newInstance();
            } catch (InstantiationException | IllegalAccessException e) {
                e.printStackTrace();
                return null;
            }
        }
    
    }
    ```
    - 在上面程序的getInstance\(\)方法中传入一个Class&lt;T&gt;参数，这是一个泛型化的Class对象，调用该Class对象的newInstance\(\)方法将返回一个T对象。
    ```
    String instance = ObjectFactory.getInstance(String.class);
    ```
    - 通过传入`String.class`便知道T代表String，所以返回的对象是String类型的，避免强制类型转换。当然Class类引入泛型的好处不止这一点，在以后的实际应用中会更加能体会到。
- 直接使用Field的getType\(\)方法只能获取普通类型的Field的数据类型：对于增加了泛型参数的类型的 Field，应该使用 getGenericType\(\) 方法来取得其类型
        ```
        public class GenericTest{
            private Map<String , Integer> score;
            
            public static void main(String[] args)
                throws Exception{
                Class<GenericTest> clazz = GenericTest.class;
                Field f = clazz.getDeclaredField("score");
                // 直接使用getType()取出Field类型只对普通类型的Field有效
                Class<?> a = f.getType();
                // 下面将看到仅输出java.util.Map
                System.out.println("score的类型是:" + a);
                // 获得Field实例f的泛型类型
                Type gType = f.getGenericType();
                // 如果gType类型是ParameterizedType对象
                if(gType instanceof ParameterizedType){
                    // 强制类型转换
                    ParameterizedType pType = (ParameterizedType)gType;
                    // 获取原始类型
                    Type rType = pType.getRawType();
                    System.out.println("原始类型是：" + rType);
                    // 取得泛型类型的泛型参数
                    Type[] tArgs = pType.getActualTypeArguments();
                    System.out.println("泛型类型是:");
                    for (int i = 0; i < tArgs.length; i++) {
                        System.out.println("第" + i + "个泛型类型是：" + tArgs[i]);
                    }
                } else{
                    System.out.println("获取泛型类型出错！");
                }
            }
        }
        ```
        - 输出结果：
            > score 的类型是: interface java.util.Map  
            > 原始类型是: interface java.util.Map  
            > 泛型类型是:  
            > 第 0 个泛型类型是: class java.lang.String  
            > 第 1 个泛型类型是：class java.lang.Integer

## Q:为什么说反射的效率低?
1. Method#invoke方法的参数是Object[],所以在调用时会对参数做封装和解封操作,会产生大量临时对象
2. 需要检查方法可见性
3. 需要校验参数
4. 反射方法难以内联
5. JIT难以优化

## Q:HashCode为什么使用31作为乘数?
1. 是一个奇质数，如果选择偶数会导致乘积运算时数据溢出。
2. 另外在二进制中，2个5次方是32，那么也就是 31 * i == (i << 5) - i。这主要是说乘积运算可以使用位移提升性能，同时目前的JVM虚拟机也会自动支持此类的优化。
3. 减少碰撞概率

## Q:static变量存储位置是哪里？静态变量的生命周期？静态变量何时销毁？静态引用的对象回收如何理解？
- static变量存储位置
  - 注意是：存储在JVM的方法区中
  - static变量在类加载时被初始化，存储在JVM的方法区中，整个内存中只有一个static变量的拷贝，可以使用类名直接访问，也可以通过类的实例化对象访问，一般不推荐通过实例化对象访问，通俗的讲static变量属于类，不属于对象，任何实例化的对象访问的都是同一个static变量，任何地放都可以通过类名来访问static变量。
- 静态变量的生命周期
  - 当我们启动一个app的时候，系统会创建一个进程，此进程会加载一个Dalvik VM的实例，然后代码就运行在DVM之上，类的加载和卸载，垃圾回收等事情都由DVM负责。也就是说在进程启动的时候，类被加载，静态变量被分配内存。
- 静态变量何时销毁
  - 类在什么时候被卸载？在进程结束的时候。
  - 说明：一般情况下，所有的类都是默认的ClassLoader加载的，只要ClassLoader存在，类就不会被卸载，而默认的ClassLoader生命周期是与进程一致的
- 静态引用的对象回收
  - 只要静态变量没有被销毁也没有置null，其对象一直被保持引用，也即引用计数不可能是0，因此不会被垃圾回收。因此，单例对象在运行时不会被回收

## Q:什么是绑定？静态和动态绑定如何区别？动态绑定编译原理是什么？动态绑定运行原理是什么？
- 什么是绑定？
    - 把一个方法与其所在的类/对象 关联起来叫做方法的绑定。绑定分为静态绑定（前期绑定）和动态绑定（后期绑定）。
- 静态和动态绑定如何区别？
    - 静态绑定（前期绑定）是指：
        - 在程序运行前就已经知道方法是属于那个类的，在编译的时候就可以连接到类的中，定位到这个方法。
        - 在Java中，final、private、static修饰的方法以及构造函数都是静态绑定的，不需程序运行，不需具体的实例对象就可以知道这个方法的具体内容。
    - 动态绑定（后期绑定）是指：
        - 在程序运行过程中，根据具体的实例对象才能具体确定是哪个方法。
        - 动态绑定是多态性得以实现的重要因素，它通过方法表来实现：每个类被加载到虚拟机时，在方法区保存元数据，其中，包括一个叫做 方法表（method table）的东西，表中记录了这个类定义的方法的指针，每个表项指向一个具体的方法代码。如果这个类重写了父类中的某个方法，则对应表项指向新的代码实现处。从父类继承来的方法位于子类定义的方法的前面。
- 动态绑定编译原理
    - 我们假设 Father ft=new Son();  ft.say();  Son继承自Father，重写了say()。
    - 编译：我们知道，向上转型时，用父类引用执行子类对象，并可以用父类引用调用子类中重写了的同名方法。但是不能调用子类中新增的方法，为什么呢？
    - 因为在代码的编译阶段，编译器通过 声明对象的类型（即引用本身的类型） 在方法区中该类型的方法表中查找匹配的方法（最佳匹配法：参数类型最接近的被调用），如果有则编译通过。（这里是根据声明的对象类型来查找的，所以此处是查找 Father类的方法表，而Father类方法表中是没有子类新增的方法的，所以不能调用。）
    - 编译阶段是确保方法的存在性，保证程序能顺利、安全运行。
- 动态绑定运行原理是什么？
    - 运行：我们又知道，ft.say()调用的是Son中的say()，这不就与上面说的，查找Father类的方法表的匹配方法矛盾了吗？不，这里就是动态绑定机制的真正体现。
    - 上面编译阶段在 声明对象类型 的方法表中查找方法，只是为了安全地通过编译（也为了检验方法是否是存在的）。而在实际运行这条语句时，在执行 Father ft=new Son(); 这一句时创建了一个Son实例对象，然后在 ft.say() 调用方法时，JVM会把刚才的son对象压入操作数栈，用它来进行调用。而用实例对象进行方法调用的过程就是动态绑定：根据实例对象所属的类型去查找它的方法表，找到匹配的方法进行调用。我们知道，子类中如果重写了父类的方法，则方法表中同名表项会指向子类的方法代码；若无重写，则按照父类中的方法表顺序保存在子类方法表中。故此：动态绑定根据对象的类型的方法表查找方法是一定会匹配（因为编译时在父类方法表中以及查找并匹配成功了，说明方法是存在的。这也解释了为何向上转型时父类引用不能调用子类新增的方法：在父类方法表中必须先对这个方法的存在性进行检验，如果在运行时才检验就容易出危险——可能子类中也没有这个方法）.
- 两者之间区分
    - 程序在JVM运行过程中，会把类的类型信息、static属性和方法、final常量等元数据加载到方法区，这些在类被加载时就已经知道，不需对象的创建就能访问的，就是静态绑定的内容；需要等对象创建出来，使用时根据堆中的实例对象的类型才进行取用的就是动态绑定的内容。

## Q:Java内部类和静态内部类的区别
* 内部类:总是隐式的持有外部类的引用,内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据
* 静态内部类:不持有外部类的引用

## Q:为什么内部类调用的局部变量必须是final修饰的？局部变量对垃圾回收机制有什么样的影响？
- 为什么内部类调用的外部变量必须是final修饰的？
    - 简单解答：一方面，由于方法中的局部变量的生命周期很短，一旦方法结束变量就要被销毁，为了保证在内部类中能找到外部局部变量，通过final关键字可得到一个外部变量的引用；另一方面，通过final关键字也不会在内部类去做修改该变量的值，保护了数据的一致性。 
    - 详细一点可以这样说：因为生命周期的原因。方法中的局部变量，方法结束后这个变量就要释放掉，final保证这个变量始终指向一个对象。首先，内部类和外部类其实是处于同一个级别，内部类不会因为定义在方法中就会随着方法的执行完毕而跟随者被销毁。问题就来了，如果外部类的方法中的变量不定义final，那么当外部类方法执行完毕的时候，这个局部变量肯定也就被GC了，然而内部类的某个方法还没有执行完，这个时候他所引用的外部变量已经找不到了。如果定义为final，java会将这个变量复制一份作为成员变量内置于内部类中，这样的话，由于final所修饰的值始终无法改变，所以这个变量所指向的内存区域就不会变。
    - 为了解决：局部变量的生命周期与局部内部类的对象的生命周期的不一致性问题
    - 注意：在java 1.8中，可以不用final修饰，但是千万不要被误导，因为你不用final修饰，在匿名内部类中修改它的值还是会导致编译报错。因为java 1.8其实会自动给它加上final
- 局部变量对垃圾回收机制有什么样的影响？
    - 先说出结论：局部变量表中的变量是很重要的垃圾回收根节点，被局部变量表中变量直接或者间接引用的对象都不会被回收

## Q:什么是多态?多态有哪些弊端？Java实现多态有哪些必要条件？什么是向上、向下转型?
- 什么是多态？
    - 多态是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。因为在程序运行时才确定具体的类，这样，不用修改源程序代码，就可以让引用变量绑定到各种不同的类实现上，从而导致该引用调用的具体方法随之改变，即不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。
- Java实现多态的必要条件?
    - Java实现多态有三个必要条件：继承、重写、向上转型。
    - 继承：在多态中必须存在有继承关系的子类和父类。
    - 重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
    - 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。
- 向上、向下转型?
   ```
    class Demo_SuperMan {
        public static void main(String[]args){
            Person p=new SuperMan();//父类引用指向子类对象。超人提升为了人
                                    //父类引用指向子类对象，就是向上转型
            System.out.println(p.name); //John
            p.Tsy();//子类Tsy
            //p.Fly();//找不到该方法
            SuperMan sm=(SuperMan)p;//向下转型,看到整个对象的内容
            System.out.println(sm.name);//SuperName
            sm.Fly();//飞出去救
        }
    }
     
    class Person{
        String name="John";
        public void Tsy(){
            System.out.println("Tsy");
        }
    }
     
    class SuperMan extends Person{
        String name="SuperName";
        @Override
        public void Tsy(){
            System.out.println("子类Tsy");
        }
     
        public void Fly(){
            System.out.println("飞出去救人");
        }
    }
    ```

## Q:常见代码块有哪些?执行顺序是什么样子的？
* a:局部代码块
    * 在方法中出现；限定变量生命周期，及早释放，提高内存利用率
* b:构造代码块
    * 在类中方法外出现；多个构造方法方法中相同的代码存放到一起，每次调用构造都执行，并且在构造方法前执行
* c:静态代码块
    * 在类中方法外出现，加了static修饰
    * 在类中方法外出现，并加上static修饰；用于给类进行初始化，在加载的时候就执行，并且只执行一次

## Q:带有类名.变量加载时,加载顺序是怎么样的?
- 当遇到 类名.变量 加载时，只加载变量所在类。
- 当遇到new加载类时，先执行父类，在执行子类。
- 在同一个类中，代码块比构造方法先执行。

## Q:Java数据类型有哪些？什么是值传递？什么是引用传递？如何理解值传递和引用传递，以及它们有何区别？
- Java数据类型有哪些？
    - Java中的数据类型分为两种为基本类型和引用类型。
        - 1、基本类型的变量保存原始值，所以变量就是数据本身。
            - 常见的基本类型：byte,short,int,long,char,float,double,Boolean,returnAddress。
        - 2、引用类型的变量保存引用值，所谓的引用值就是对象所在内存空间的“首地址值”，通过对这个引用值来操作对象。
            - 常见的引用类型：类类型，接口类型和数组。
    - 基本类型(primitive types)
        - primitive types 包括boolean类型以及数值类型（numeric types）。numeric types又分为整型（integer types）和浮点型（floating-point type）。整型有5种：byte short int long char(char本质上是一种特殊的int)。浮点类型有float和double。
    - 引用类型(reference types)
        - ①接口 ②类 ③数组
- 什么是值传递？
    - 在方法的调用过程中，实参把它的实际值传递给形参，此传递过程就是将实参的值复制一份传递到函数中，这样如果在函数中对该值（形参的值）进行了操作将不会影响实参的值。因为是直接复制，所以这种方式在传递大量数据时，运行效率会特别低下。
    - 比如String类，设计成不可变的，所以每次赋值都是重新创建一个新的对象，因此是值传递！
- 什么是引用传递？
    - 引用传递弥补了值传递的不足，如果传递的数据量很大，直接复过去的话，会占用大量的内存空间。
    - 引用传递就是将对象的地址值传递过去，函数接收的是原始值的首地址值。
    - 在方法的执行过程中，形参和实参的内容相同，指向同一块内存地址，也就是说操作的其实都是源数据，所以方法的执行将会影响到实际对象。
- 如何理解值传递和引用传递，以及它们有何区别？
    - 看下面代码案例
        ```
        private void test1(){
            Demo demo = new Demo();
            demo.change(demo.str, demo.ch);
            Log.d("yc---",demo.str);
            Log.d("yc---", Arrays.toString(demo.ch));
            //打印值
            //yc---: hello
            //yc---: [c, b]
        }
        
        public class Demo {
            String str = new String("hello");
            char[] ch = {'a', 'b'};
            public void change(String str, char[] ch) {
                str = "ok";
                ch[0] = 'c';
            }
        }
        ```
    - 案例过程分析
        - 为对象分配空间
            - ![image](https://upload-images.jianshu.io/upload_images/4432347-56468ab4f7dc848f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
        - 执行change()方法
            - 执行前实参（黑色）和形参（红色）的指向如下：
            - ![image](https://upload-images.jianshu.io/upload_images/4432347-9b599a69970d3bc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
        - 最后打印
            - 因为String是不可变类且为值传递，而ch[]是引用传递，所以方法中的str = "ok",相当于重新创建一个对象并没有改变实参str的值，数组是引用传递，直接改变，所以执行完方法后，指向关系如下：
            - ![image](https://upload-images.jianshu.io/upload_images/4432347-f75afffa63e84718.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 通过上面的分析我们可以得出以下结论：
    - 基本数据类型传值，对形参的修改不会影响实参；
    - 引用类型传引用，形参和实参指向同一个内存地址（同一个对象），所以对参数的修改会影响到实际的对象。
    - String, Integer, Double等immutable的类型特殊处理，可以理解为传值，最后的操作不会修改实参对象。
- 如何理解引用类型的按值传递？
    - 引用类型的按值传递，传递的是对象的地址。只是得到元素的地址值，并没有复制元素。比如数组，就是引用传递，假如说是值传递，那么在方法调用赋值中，将实参的值复制一份传递到函数中将会非常影响效率

# Q:Java中代理有几种方式?各有什么特点?
- 静态代理
   - 编译成class文件,通过接口实现
```java
public interface IUserDao {
    public void save();
}
public class UserDao implements IUserDao{

    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
public class UserDaoProxy implements IUserDao{

    private IUserDao target;
    public UserDaoProxy(IUserDao target) {
        this.target = target;
    }
    
    @Override
    public void save() {
        System.out.println("开启事务");//扩展了额外功能
        target.save();
        System.out.println("提交事务");
    }
}
public class StaticUserProxy {
    @Test
    public void testStaticProxy(){
        //目标对象
        IUserDao target = new UserDao();
        //代理对象
        UserDaoProxy proxy = new UserDaoProxy(target);
        proxy.save();//开启事务、保存数据、提交事务
    }
}
```
- 动态代理
   - 动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中
```java
动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中
特点：
动态代理对象不需要实现接口，但是要求目标对象必须实现接口，否则不能使用动态代理。

JDK中生成代理对象主要涉及的类有

java.lang.reflect Proxy，主要方法为
static Object    newProxyInstance(ClassLoader loader,  //指定当前目标对象使用类加载器

 Class<?>[] interfaces,    //目标对象实现的接口的类型
 InvocationHandler h      //事件处理器
) 
//返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
java.lang.reflect InvocationHandler，主要方法为
 Object    invoke(Object proxy, Method method, Object[] args) 
// 在代理实例上处理方法调用并返回结果。
举例：保存用户功能的动态代理实现

接口类： IUserDao
package com.proxy;

public interface IUserDao {
    public void save();
}
目标对象：UserDao
package com.proxy;

public class UserDao implements IUserDao{

    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
动态代理对象：UserProxyFactory
package com.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyFactory {

    private Object target;// 维护一个目标对象

    public ProxyFactory(Object target) {
        this.target = target;
    }

    // 为目标对象生成代理对象
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                new InvocationHandler() {

                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开启事务");

                        // 执行目标对象方法
                        Object returnValue = method.invoke(target, args);

                        System.out.println("提交事务");
                        return null;
                    }
                });
    }
}
```
## Q:java中方法参数的几种使用情况?
* 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）；
* 一个方法可以改变一个对象参数的状态；
* 一个方法不能让对象参数引用一个新的对象