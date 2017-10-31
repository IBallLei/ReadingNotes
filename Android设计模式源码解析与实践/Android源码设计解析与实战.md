## 第一章 面向对象的六大原则

#### 1.1 单一原则——SRP

**定义：** 就一个类而言，应该仅有一个引起它变化的原因。

#### 1.2 开闭原则——OCP

**定义：** 软件中的对象应该对拓展是开放的，对于修改是封闭的。

#### 1.3 里氏替换原则——LSP

**定义：** 所有引用基类的地方必须能透明的使用其子类的对象。

#### 1.4 依赖倒置原则——DIP

**定义：** 模块间的依赖通过抽象发生，实现类之间不发生直接的依赖关系，其依赖关系是通过接口或抽象类产生的。

#### 1.5 接口隔离原则——ISP

**定义1：** 客户端不应该依赖他不需要的接口。
**定义2：** 类间的依赖依赖关系应该建立在最小的接口上。

#### 1.6 迪米特原则（最少只是原则）——LOD

**定义：** 一个对象应该对其他对象有最少的了解。

-----------------------------------------------------





## 第二章 应用最广的模式——单例模式

#### 2.1 单例模式介绍

**定义：** 确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

**场景：** 避免产生多个对象消耗过多的资源，或某种类型对象只应该有一个实例。

*****************************************************

#### 2.2 实现方式

**2.2.1 懒汉式：**

```
private static SingletonDemo instance;
private SingletonDemo() {
}
public static synchronize SingletonDemo getInstance() {
    if (instance == null) {
        instance = new SingletonDemo();
    }
    return instance;
}
```
* 优点：用到时，才创建实例。
* 缺点：第一次加载时，创建实例，反应稍慢；线程同步造成性能开销。
* 不建议使用

**2.2.2 DCL（Double）：**
```
private volatile static SingletonDemo instance;

private SingletonDemo() {
}

public static SingletonDemo getInstance() {
 if (instance == null) {
     synchronized (SingletonDemo.class) {
         if (instance == null) {
             instance = new SingletonDemo();
         }
     }
 }
 return instance;
}
```
* 优点：既能够在需要时创建实例，又能保证线程安全，且初始化后不进行同步锁。
* 缺点：第一次加载稍慢，双重检查锁定失效，《Java并发编程实践》提到，优化丑陋，不赞成，应使用下面那种。
* 由于java的编译器润徐乱序执行，创建实例时，容易产生先分配内存后初始化成员变量，可消耗性能通过volatile关键字提高正确性。

**2.2.3 静态内部类单例模式：**

```
private static SingletonDemo instance;

private SingletonDemo() {
}

public static SingletonDemo getInstance() {
 return SingletonHolder.instance;
}

private static class SingletonHolder{
    private static final SingletonDemo instance = new SingletonDemo();
}
```
* 优点：当第一次加载这个类的时候不会创建实例，只用调用getInstance时才初始化实例，线程安全，代码简洁。
* 推荐使用的方式

**2.2.4 枚举**

```
public enum SingletonEnum {
    INSTANCE;
    public void doSomething() {
    }
}
```
* 优点：防止反序列化造成多个单例对象，线程安全，一直单例。
* 缺点：Android中不建议使用枚举。
* 以上几种防止反序列化的方法是：加上readResolve方法，返回单例。

**2.2.5 使用容器实现单例模式**

```
public class SingletonManager {
    private static Map<String, Object> singletonMap = new HashMap<>();
    
    private SingletonManager(){
        
    }
    
    public static void registerService(String key, Object instance) {
        if(!singletonMap.contains(key)) {
            singletonMap.put(key, instance);
        }
    }
    
    public static Object getService(String key) {
        return singletonMap.get(key);
    }
}
```
* 优点：统一管理多个单例类，统一接口调用获取，降低耦合。

**2.2.6 小结**

&emsp;&emsp;核心原理就是构造函数私有化，通过静态方法获取唯一实例，保证线程安全，防止反序列化。用哪种取决项目本身。

*****************************************************

#### 2.3 Android源码中的单例模式:LayoutInflater

&emsp;&emsp;在Android中，我们通常会获取许多系统级的服务，如WindowManagerService、ActivityManagerService以及LayoutInflater，这些类会注册在系统之中。我们通过Context的getSystemService(key)方法进行获取使用。在LayoutInflater的使用中，通过LayoutInflater.from(context)获取LayoutInflater服务，在from方法中是通过context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)获取到单例，再调用其inflate()等方法加载布局控件。

&emsp;&emsp;通过Android代码分析获取单例的整个流程：

1. 首先是通过Context的调用获取服务的方法，但是这时发现Context是抽象类。
2. 我们知道，Context的创建实在Activity创建由ActivityThread的主函数main中创建。
3. 在ActivityThread通过binder机制与ActivityManagerService进行通信，最终调用ActivityThread的handleLaunchActivity()。
4. 在handleLaunchActivity()方法中，先后进行了，创建Activity对象，创建ContextImpl，绑定Context到Activity，调用Activity的onCreate方法。
5. 通过ContextImpl代码：
    * 在静态代码块中初始化中注册各种继承自内部类ServiceFatcher的服务类，存放到静态的map中
    * 然后通过key获取对应的fatcher类，然后调用其getService()获取服务对象
    * 当第一次获取时，会创建实例然后存放到缓存列表中，之后调用直接获取缓存。
6. 运用了容器管理的单例模式。

*****************************************************

#### 2.4 深入理解LayoutInflater

1. LayoutInflater是个抽象类：通过初始化服务流程可知，反射拿到Policy的实现类创建了其实现类PhoneLayoutInfalter对象。

2. PhoneLayoutInfalter只是复写了onCreateView()，作用是将控件名拼接成完整路径名。

3. 由Activity分析，Activity套Window套decorView套contentView，通过LayoutInflater的inflate()将我们的布局加载到里面。

4. 在加载中进行以下操作：
    1. 获取布局的解析器，通过解析器拿到根元素
    2. 如果是merge的话，解析内部所有子控件添加到父控件中
    3. 如果不是，创建根控件，再添加所有子控件到根控件中
    4. 返回根视图

5. createView()：拼接好控件完整路径，通过类加载器，反射构造出控件，将构造方法缓存，便于下次创建。

6. rInflate()：遍历解析创建树状视图，每个元素递归调用rInflate()。构建完毕setContentView显示。

*****************************************************

#### 2.5 总结

##### 优点：

1. 只生成一个实例，减少内存开支。
2. 只生成一个实例，减少系统性能开销。
3. 避免对资源的多重占用。
4. 设置全局访问点，优化和共享资源访问。

##### 缺点：

1. 没有接口，拓展困难，只能改代码来实现。
2. 持有Context时，容易导致内存泄漏，如果用最好是ApplicationContext。

*****************************************************





## 第三章 自由拓展你的项目——Builder模式

#### 3.1 Builder模式介绍

**定义：**将一个复杂对象的构建与它的表示分离，使同样的构建过程可以创建不同的表示。

**场景：**

1. 相同的方法，不同的执行顺序，产生不同的事件结果时。

2. 多个部件或零件，都可以装配到一个对象中，但是产生的结果不相同时。

3. 产品类非常复杂，或产品类中的调用顺序不同产生不同的作用时。

4. 当初始化一个对象非常复杂，如参数多，且很多参数都有默认值时。

**角色：**

* **Product**：产品类
* **Builder**：建造抽象类
* **ConcreteBuilder**：建造实现类
* **Director**：组装过程类

-----------------------------------------------------

#### 3.2 实现方式

1. **Product**：创建Bean的实体产品类

2. **Builder**：创建定义建造抽象方法的抽象建造类，定义构建方法和创建方法

3. **ConcreteBuilder**
：创建继承自抽象类的具体实现的抽象类，持有产品类的引用，在创建方法中返回产品类

4. **Director**：创建时传入 Builder 实现类，在构建方法里调用 Builder 的构建方法传入参数，最终调用 Builder 的创建方法，返回实体产品对象。

    >现实中，Director 通常会被省略，直接在 Builder 类中进行组装，然后再组装的方法中返回自身。

    >形成链式调用：new Builder().setA("a").setB("b").create();

    >这样做不仅结构简单，而且也能对 Product 进行精密控制。

-----------------------------------------------------

#### 3.3 Android中的建造者模式：AlertDialog.Builder

##### 3.3.1 AlertDialog.Builder 的使用代码如下：

```
AlertDialog alertDialog = new AlertDialog.Builder(mActivity)
                .setIcon(R.drawable.icon)
                .setTitle("title")
                .setMessage("msg")
                .create();
alertDialog.show();
```

##### 3.3.2 AlertDialog.Builder 的使用的内部代码解析：

1. AlertDialog 的中所有设置参数的set方法，都是调用成员变量 AlertController 的同名方法，并且在 AlertDialog 创建的时候创建 AlertController 对象。

2. AlertDialog.Builder 作为 AlertDialog 的内部类，同样内部也有一个成员变量 AlertController.AlertParams P 在构建的set方法中，将参数赋值给 P 的成员属性，并返回 Builder 本身，形成链式调用。

3. 调用 AlertDialog.Builder 的 create() 方法完成 AlertDialog 对象的创建，并且返回。

    1. 创建 AlertDialog 对象
    
    2. 调用 Builder 中 P 的 apply()方法传入 AlertDialog 中的 AlertController ，在该方法中，完成构建参数的传递。

4. 调用 AlertDialog.show() 方法显示 AlertDialog 窗口内容。

    1. 调用 onCreate() 方法完成页面控件的解析创建的初始化，在内部调用了 alertController.installContent()方法。
    
        1. 通过 setContentView() 设置了窗口的布局。
    
        2. 调用 setupView() 方法：
        
            1. 获取初始化内容区域
            2. 初始化按钮
            3. 初始化title
            4. 初始化自定义区域，如果有的话
            5. 等等，最后初始化背景
    
    2. 调用 onStart() 方法。
    
    3. 获取 DecorView。
    
    4. 获取布局参数。
    
    5. 将 DecorView 添加到 WindowManager 中，并发送一个显示 dialog 的消息。

-----------------------------------------------------

#### 3.4 深入理解 WindowManager（以及 WMS —— WindowManagerService）

##### 3.4.1 将 Dialog 添加到手机窗口显示的过程

1. 与 LayoutInflate 相同，首先通过 Context.getSystemService() 在 ContextImpl 中初始化时注册在装有各类服务的 map 中，获取 WindowManager 的服务实例 WindowManagerImpl。

2. 通过 Dialog 的构造函数传入 context，然后通过 context.getSystemService() 方法获取到 WindowManager 。

3. 通过 PolicyManager.makeNewWindow(context) 获取到 Window 对象。

4. 给 Window 设置回调，传入 Dialog 对象本身。

5. 调用 window.setWindowManager(windowManager), 将 Window 与 Manager 关联。在其方法内部通过传入的 windowManager 调用 createLocalWindowManager() 方法，传入 window 本身，用来创建返回 WindowManagerImpl 对象，并赋值给 Window 中的成员变量。

    1. 创建 WindowManagerImpl 时，调用构造函数多传入一个 parentWindow ，进行关联。

    2. WindowManagerImpl 中的其他方法，是通过内部成员变量 WindowManagerGlobal 的代理，调用同名的相应方法（addView(), removeView()等）。

6. 在以上准备好后，调用 WindowManager 的 addView() 方法，也就是 WindowManagerGlobal 的方法。将要显示的 View 加到 Window 里面。

    1. 构建 ViewRootImpl 对象

    2. 给需要添加的 View 设置布局属性 setLayoutParam

    3. 将 View 添加到 Views 列表中，将 root 添加到 roots 集合中，将布局参数加到布局集合中。

    4. 调用 root.setView() 将 View 显示到手机窗口中。

##### 3.4.2 ViewRootImpl 的创建与通信的桥梁（只讨论 WindowManager 模块）

1. ViewRootImpl：继承自 Handler 是 native 层与 java 层通信的桥梁

2. 构造函数中，会通过 WindowManagerGlobal.getWindowSession，这是建立通信的地方。

    1. 通过 IWindowManager.Stub.asInterface(ServiceManager.getService("window")) 获取 WMS（IBinder）并转化成 WindowManager。

        1. ServiceManager.getService() 中先去缓存集合中去取"window"中的 WMS。

        2. 如果没有缓存，再调用 getIServiceManager().getService("window") 并返回。

    2. 调用 WMS.openSession() 方法，创建一个 session 并返回

3. 保存当前线程，创建时，是在UI线程创建的。

4. 再 Root.setView() 之后并不能直接显示，WMS 管理的不是 Window 其实是 View（DecorView），其中：

    1. 主要是调用 requestLayout()，请求布局：发送 DO_TRAVERSAL 消息调用 performTraversals() 方法进行测量绘制，内部最后会调用performDraw()进行绘制。

        1. 判断是 CPU 绘制，还是 GPU 绘制。
        2. 获取绘制表面的 Surface 对象。
        3. 通过 Surface 获取并锁住 Canvas 对象。
        4. 从 DecorView 开始绘制整个视图树。
        5. Surface 解锁 Canvas 对象，并通知 SurfaceFlinger 更新视图。

    2. 绘制完整个窗后视图后，通过 session 向 WMS 发送显示请求。

-----------------------------------------------------

#### 3.5 总结

##### 优点：

1. 良好的封装性，隐藏构建细节。
2. 建造者独立，容易拓展。

##### 缺点：

1. 增加类，占用内存。

-----------------------------------------------------





## 第四章 使程序运行高效——原型模式

#### 4.1 原型模式介绍

**定义：**用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。

**场景：**

1. 类的初始化需要消耗非常多的资源。

2. 通过 new 产生一个对象时，需要非常繁琐的数据准备和访问权限。

3. 一个对象需要提供给其他对象访问，且各个调用的对象需要修改其属性值，即保护性拷贝。

> 需要注意：

**角色：**

* **Client**：客户端用户
* **Prototype**：抽象类或借口
* **ConcretePrototype**：原型实现类

-----------------------------------------------------

#### 4.2 实现方式

1. ConcretePrototype:

```

public class A implements Cloneable {
    // 成员变量

    // 构造函数

    @Override
    protected A clone() {
        try {
            A a = (A) super.clone();
            a.x = this.x
            return a
        } catch() {}
        return null;
    }

    // get和set方法

    // show方法显示成员变量值
}

```

2. client端：创建 A 对象，给其赋值，然后克隆一个对象，对其赋其他值，调用 show 方法，显示不同的属性值。

-----------------------------------------------------

#### 4.3 浅拷贝和深拷贝

**浅拷贝：**并不是将对象的所有字段全部重新构造一遍，而是将指针指向同一地址。当其中拷贝的属性内容发生变化，有可能导致原型对象也发生变化。

**深拷贝：**完善浅拷贝的方法，将成员属性也重新拷贝一份，导致指针指向不同的地址，从而不会对原型对象造成改变。

-----------------------------------------------------

#### 4.4 Android 源码中的原型模式实现

##### ArrayList：

1. ArrayList 实现了 Cloneable 接口，并实现了克隆方法，对持有的成员变量 Object 数组进行了克隆，并赋给克隆对象的成员变量，实现内部元素的克隆。

##### Intent 来分析 Android 源码中的原型模式

1. 用 Intent 克隆出一个对象，然后启动一个 Activity ，在新的页面获取得到的数据与原型对象的数据相同。

2. 从 Intent 的源码中发现，Intent 实现的 clone() 中，重新创建了一个对象当做克隆对象，并将原型对象传入到新建对象的构造方法中。在构造方法里，将传入对象的成员变量，重新赋给新建的对象，集合和对象类型的成员变量，则如果原型对象有值，则重新创建该变量类型的对象，再赋给成员变量。

3. 可以看出，如果构造函数成本比较高的时候，那么使用克隆函数效率更高，否则可以使用构造函数。如 Intent ，将原型对象当做参数，用构造函数重新创建一个对象，并将值重新拷贝一遍。

-----------------------------------------------------

#### 4.5 Intent 的查找与匹配

##### 4.5.1 App信息表的构建

1. **PackageServiceManager：**系统启动的时候注册各种服务包括 PSM ，PSM 启动之后，会扫描已经安装的 apk 目录，然后解析每个 apk 的 AndroidManifest.xml 文件，解析出里面的信息（包括 Activity 和 Service 等注册信息），扫描解析后，就构成了整个 apk 信息树。下面是解析过程：

    1. 在 PSM 构造函数中，获取 data 目录；加载 Framework 资源；加载核心库；获取系统应用目录，扫描系统应用目录；扫描第三方应用。

    2. 扫描方法 scanDirLI() 内部实现：

        1. 获取目录下所有文件
        2. 循环解析目录下所有 apk 文件（不是 apk 文件的跳过）；循环中，解析 apk 文件方法 scanPackageLI()

    3. 解析 apk 文件方法 scanPackageLI()：

        1. 创建解析包：PackageParser

        2. 用解析包解析扫描到的文件，获得包信息对象 PackageParser.Package apk。

            1. 如果该文件是文件夹，则解析文件夹下的所有 apk 文件，parseClusterPackage()。

            2. 如果是单个 apk 文件，则解析单个文件，parseMonolithicPackage()。

                1. 构建 AssetManager 对象

                2. 构建资源 Resource 

                3. 用 assetManager 对象，获取 AndroidManifest 解析器。assmg.openXmlResourceParser()

                4. 解析获取到的 AndroidManifest.xml 文件。parsePackage()

                    1. 解析包名，构建 Package 对象。

                    2. 循环解析内部元素：

                        1. 解析到 application 元素时，再次解析 application 内部的 Activity 和 Service 等组件。parseApplication()，在方法内部，获取包名，应用名，应用图标等其他属性，以及四大组件的信息，根据不同的标签调用不同的解析方法。

                        2. 解析到 users-permission 权限元素时，再次解析内部权限。 parseUsersPermission()

        3. 解析 apk 得到的信息中的 Activity 和 Service 等组件。scanPackageLI()，内部调用 scanPackageDirtyLI() 进行扫描。

            1. 将 apk 中解析到的到的 PackageParser.Activity 等组件全部取出，并分别存到 PMS 中的集合 mActivitys，mServices，mReceivers中；

2. 通过 PackageServiceManager 获得到的 apk 的信息全部存到系统当中，当通过 Intent 启动组件的时候，就会去这个信息表中查找，符合要求的组件就会被启动。这样就将各个组件连接在一起。

##### 4.5.2 精准匹配

1. 通过 Intent 启动 Activity 时，最终都是调用 startActivityForResult() 。

2. 启动 Activity：调用 mInstrumentation.execStartActivity()。

    1. 将 Intent 中的数据迁移到粘贴板上，调用 Intent 的方法 prepareToLeaveProcess() 准备离开当前线程。

    2. 调用 AMS 的 startActivity() 方法，里面有调用了 ActivityStackSupervisor 的 startActivityMayWait()，该方法最后调用了 PMS 的 resolveIntent() ，内部有调用了 PMS 自身的 queryIntentActivities() 返回一个符合匹配的 ActivityInfo 列表 List<ResolveInfo>，每个元素是记录一个 Activity 的信息。

        1. 在 queryIntentActivities() 中，先获取 Intent 的 Component 对象。

        2. 精确查找时（显式启动），component 不为空，通过 getActivityInfo(component) 获取 ActivityInfo，然后将 ai 赋值给构建的 ResolveInfo 的成员变量，将 ri 加到构建的集合中，返回。

        3. 当隐式启动时，component 为空：Intent 获取包名，通过包名获取 Package 对象，调用 mActivities.queryIntentForPackage() 返回查询结果。

    3. 检测结果，并且返回给调用端。

3. 发送启动请求：mMainThread.sendActivityResult()。

-----------------------------------------------------

#### 4.6 原型模式实战

##### 问题：
    1. 有 User 类持有 Address 类变量。

    2. 提供外部使用的UserSession，内部 User 对象私有，并且 setUser() 方法包内私有，无法直接对 User 进行修改

    3. 通过 getUserSession() 方法获取 User 并对其成员变量进行修改，导致 User 改变。

##### 解决：

    1. 使 User 集成 Cloneable ，并且实现 clone 方法。

    2. 再用 UserSession 获取 User 实例时，返回原型 User 克隆的对象，来保护原型不被修改。

-----------------------------------------------------

#### 4.7 总结

##### 优点：

* 原型模式是在内存中进行二进制流拷贝，比创建对象性能好。特别循环中。

##### 缺点：

* 优点也是缺点，直接内存拷贝，构造函数不会被执行，实际中有潜在问题。

-----------------------------------------------------





## 第五章 应用最广泛的模式——工厂方法模式

#### 5.1 工厂方法模式介绍

**定义：**定义一个用于创建对象的接口，让子类决定实例化哪个类。

**场景：**在任何需要创建复杂对象的地方，复杂对象适合，直接用 new 可以创建的无需使用工厂方法模式。

**角色：**

* **Client**：客户端用户
* **AbstractFactory**：抽象工厂类
* **ConcreteFactory**：具体工厂实现类
* **AbstractProduct**：抽象产品类
* **ConcreteProduct**：具体产品实现类

-----------------------------------------------------

#### 5.2 实现方式

##### 5.2.1 简单工厂模式：工厂类只有一个，创建对象只有一个

```

public class Factory {
    public static Product createProduct() {
        return new ConcreteProduct();
    }
}

```

##### 5.2.2 反射工厂模式：通过传入 Class 通过泛型和反射，创建需要返回的产品实例

```

public class ConcreteFactory extends Factory {

    @Override
    public <T extends Product> T createProduct(Class<T> clazz) {
        Product p = null;
        try {
            p = (Product) Class.forName(clazz.getName()).newInstance();
        }
        return (T) p;
    }
}

public class Client {
    public static void main(String[] args) {
        Factory factory = new ConcreteFactory();
        Product p = factory.createProduct(ConcreteProduct.class);
        p.method();
    }
}

```

##### 5.2.3 多工厂模式：为多个产品创建多个工厂

```

public class ConcreteFactoryA extends Factory {

    @Override
    public Product createProduct() {
        return new ConcreteProductA();
    }
}

public class ConcreteFactoryB extends Factory {

    @Override
    public Product createProduct() {
        return new ConcreteProductB();
    }
}

```

##### 5.2.4 小民 Audi 汽车生产例子

* **Client**：客户端生产线
* **AudiFactory**：抽象 Audi 工厂类，抽象的生产汽车方法
* **AudiCarFactory**：具体 Audi 车工厂实现类，通过反射工厂方法，实现生产方法
* **AudiCar**：抽象奥迪车类，提供车的一些抽象方法
* **AudiQ3,AudiQ5,AudiQ7**：具体奥迪车实现类

* 通过传入奥迪车实现类到工厂实现类的方法中，获取具体实例。

-----------------------------------------------------

#### 5.3 源码中的工厂方法模式实现

##### 5.3.1 Iterator

1. List 和 Set 的实现接口 Collection 所继承的 Iterator 就一个方法 iterator()，List 和 Set 实现该方法就是构造一个实例并返回。这些实现方法就相当于工厂方法。

2. ArrayList 该方法返回构造的 ArrayListIterator() 实例。

3. HashSet 该方法会返回其成员变量 backingMap.KeySet().iterator() ，该方法中，构建了一个 KeyIterator() 并返回。

##### 5.3.2 Activity 中的 onCreate() 方法

* 通过 setContentView() 方法，传入构造的布局实例，就将不同的布局实例添加到Activity中，最后被添加到每个需要的 Activity 里面，这种模式是工程方法模式。

-----------------------------------------------------

#### 5.4 关于 onCreate 方法

1. 一个应用程序真正的入口是：ActivityThread 的 main() 方法。

2. **ActivityThread**：

    1. ActivityThread 是一个 final 定义的类，不可被继承。

    2. 一个应用程序对应一个 ActivityThread。

    3. 当一个新的应用进程被系统的 Zygote 进程孵化出来后，会执行 ActivityThread 的 main 方法。

    4. 在 main 方法中做一些常规的准备逻辑，如初始化环境，准备 Looper 和消息队列等。然后调用 attach 方法，将自身绑定在 AMS 中，开始不断读取队列中的消息和分发消息。

3. ActivityThread 中的 attach 方法：

    1. 调用 ActivityManagerNative.getDefult() 获取 IActivityManager 也就是 AMS。

    2. 然后调用 AMS.attachApplication(mActivityThread) 将两者绑定。

    3. 在 attachApplication 方法中，调用了 attachApplicationLocked 方法。

    4. attachApplicationLocked 方法中主要调用了 bindApplication 和 attachApplicationLocked 两个方法。

        1. bindApplication 方法中 主要就是将 ApplicationThread 绑定到 AMS 中。

        2. mActivityStackSupervisor.attachApplicationLocked() 方法中，调用 realStartActivityLocked() 方法，该方法是真正启动 Activity 的方法。

            1. activityRecord.startFreezingScreenLocked 锁定其他没启动的 Activity。

            2. mWindowManager.setAppVisibility() 设置 token 让当前 App 显示到前台。

            3. 搜集启动较慢的 App 信息，检查配置信息，设置相关参数信息。

            4. 如果是桌面 Activity，将其添加到当前 Activity 栈的底部。

            5. 所有参数信息到位后，准备启动 Activity，调用 ApplicationThread 的 scheduleLaunchActivity() 方法。

                1. 构建 ActivityClientRecord 实例，配置相关信息。

                2. 通过 sendMessage() 方法，发送启动 Activity 的信息放入队列，在 ActivityThread 中的内部类 H (继承于 Handler)中进行处理。

    5. 在 ActivityThread 的 H 里面标签是 LAUNCH_ACTIVITY 消息处理中，获取传过来的 ActivityClientRecord 检查其中信息，然后调用 ActivityThread 的 handleLaunchActivity() 方法里面再调用 performLaunchActivity() 方法。
    
    6. performLaunchActivity() 方法中是具体启动Activity的逻辑：

        1. 获取 ActivityInfo，获取 PackageInfo，获取 ComponentName

        2. 构建 Activity，获取类加载器，传入到 mInstrumentation.newActivity() 中，返回 Activity 实例。

        3. 获取 Application，将 Application 和 Context 绑定到 Activity 对象中

        4. 通过 mInstrumentation.callActivityOnCreate() 方法，调用 Activity 的 performCreate() 方法，以及 prePerformCreate() 和 postPerformCreate()。

        5. 最终在 performCreate() 方法中，调用 Activity 的 onCreate() 方法。

-----------------------------------------------------

#### 5.4 工厂方法模式实战

1. 功能：工厂方法模式，统一管理多种数据处理，数据库，SP，文件等。

2. 实现：

    1. 定义数据处理方法接口

    2. 定义各种数据方式处理类，实现数据处理接口，实现处理逻辑。

    3. 定义工厂类，调用方法，利用反射，返回传进来的 Class 的实例。

-----------------------------------------------------

#### 5.5 总结

优点：封装复杂的构建对象逻辑，简洁，解耦，依赖抽象，拓展性好。

缺点：增加新品类和抽象层，会导致类的结构复杂化。

-----------------------------------------------------





## 第六章 创建型设计模式——抽象工厂模式

#### 6.1 抽象工厂模式介绍

**定义：**为创建一组相关或者是相互依赖的对象提供一个接口，而不需要指定他们的具体类。

**场景：**一个对象族有相同的约束时。

>**例子：**Android，iOS，WindowsPhone 都有拨号软件和信息软件，两者都属于 Software 软件范畴，但是操作系统不同，代码实现就是不同的。这时候考虑使用抽象工厂模式来生产不同操作系统的两个软件了。

**角色：**

* **Client**：客户端用户
* **AbstractFactory**：抽象工厂类
* **ConcreteFactory**：具体工厂实现类
* **AbstractProduct**：抽象产品类
* **ConcreteProduct**：具体产品实现类

-----------------------------------------------------

#### 6.2 实现方式
















