# 第一章 面向对象的六大原则

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

&emsp;&emsp;在Android中，我们通常会获取许多系统级的服务，如 WindowManagerService、ActivityManagerService 以及 LayoutInflater，这些类会注册在系统之中。我们通过 Context 的 getSystemService(key) 方法进行获取使用。在 LayoutInflater 的使用中，通过 LayoutInflater.from(context)获取 LayoutInflater 服务，在 from 方法中是通过 context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) 获取到单例，再调用其 inflate() 等方法加载布局控件。

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

#### 5.3 Android 源码中的工厂方法模式实现

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
* **AbstractProductA**：抽象产品类A
* **AbstractProductB**：抽象产品类B
* **ConcreteProductA1**：具体产品实现类A1
* **ConcreteProductA2**：具体产品实现类A2
* **ConcreteProductB1**：具体产品实现类B1
* **ConcreteProductB2**：具体产品实现类B2
* **AbstractFactory**：抽象工厂类，AB
* **ConcreteFactory1**：具体工厂实现类，A1B1
* **ConcreteFactory2**：具体工厂实现类，A2B2

-----------------------------------------------------

#### 6.2 实现方式

1. AbstractFactory：

```

public abstract class CarFactory {
    /**
     * 生产轮胎
     */
    public abstract ITire createTire();

    /**
     * 生产发动机
     */
    public abstract IEngine createEngine();
}

```

2. AbstractProduct 和 ConcreteProduct：

```

public interface ITire {
    void tire();
}

public class NormalTire implements ITire {
    @override
    public void tire() {
        // 普通轮胎
    }
}

public class SUVTire implements ITire {
    @override
    public void tire() {
        // SUV轮胎
    }
}

public interface IEngine {
    void engine();
}

public class NormalEngine implements IEngine {
    @override
    public void engine() {
        // 普通发动机
    }
}

public class SUVEngine implements IEngine {
    @override
    public void engine() {
        // SUV发动机
    }
}

```

3. ConcreteFactory：

```

public class Q3Factory extends CarFactory {

    /**
     * 生产轮胎
     */
    public abstract ITire createTire() {
        return new NormalTire();
    }

    /**
     * 生产发动机
     */
    public abstract IEngine createEngine() {
        return new NormalEngine();
    }

}

public class Q7Factory extends CarFactory {

    /**
     * 生产轮胎
     */
    public abstract ITire createTire() {
        return new SUVTire();
    }

    /**
     * 生产发动机
     */
    public abstract IEngine createEngine() {
        return new SUVEngine();
    }

}

```

4. Client：

```

public class Client {
    public static void main(String[] args) {
        CarFactory q3factory = new Q3Factory();
        factory.createTire(); // 普通轮胎
        factory.createEngine(); // 普通发动机
        CarFactory q7factory = new Q7Factory();
        factory.createTire(); // SUV轮胎
        factory.createEngine(); // SUV轮胎
    }
}

```

-----------------------------------------------------

#### 6.3 Android 源码中的抽象工厂方法模式

##### 6.3.1 在 Framework 角度看 Activity 和 Service，二者都属于具体的工厂。

##### 6.3.2 MediaPlayer

1. MediaPlayerFactory：Android 底层会调用其方法 createPlayer 创建不同的 MediaPlayer，每一种创建完后，会调用 registerFactory_1 将 Player 注册到 MediaPlayerFactory 中。

2. MediaPlayerFactory 的本质是管理 4 种不同的 MediaPlayer 类，每种 MediaPlayer 由各自具体的 Factory 创建。

3. 四种具体工厂类继承自 MediaPlayerBase，通过不同的实现，创建不同的 MediaPlayer。

-----------------------------------------------------

#### 6.4 总结

##### 优点

* 分离接口和实现，使用抽象工厂来创建需要的对象，不用知道具体实现是谁，面向产品的接口编程，从具体的接口实现中解耦。基于接口和实现的分离，使该模式在切换产品类时更加灵活。

##### 缺点

* 类文件爆炸性增加；不容易拓展新的产品类，增加一个产品类就会修改抽象工厂，导致每个具体工厂也要修改。

-----------------------------------------------------





## 第七章 时势造英雄——策略模式

#### 7.1 策略模式介绍

**定义：**定义一系列算法，并将每种算法封装起来，使用它们还可以互相替换。让算法独立于使用它的客户而独立变化。

**场景：**

* 针对同一类型的多种处理方式，仅仅是行为有所差别时。
* 需要安全的封装多种同一类型的操作时。
* 出现同一抽象类有多个子类，而且有需要使用 if-else 或者 switch 来选择具体子类时。

**角色：**

* **Context**：用来操作策略的上下文环境
* **Strategy**：策略的抽象
* **ConcreteStrategyA/B**：具体的策略实现

-----------------------------------------------------

#### 7.2 实现方式

##### 算法替换，简化逻辑和结构，增加了可读性，稳定性，拓展性。

```

public interface CalculateStrategy {
    int calculatePrice(int km);
}

public class BusStrategy implements CalculateStrategy {
    @override
    public int calculatePrice(int km) {
        // 计算过程
        return busCalculatePrice;
    }
}

public class SubwayStrategy implements CalculateStrategy {
    @override
    public int calculatePrice(int km) {
        // 计算过程
        return subwayCalculatePrice;
    }
}

public class TranficCalculate {
    public static void main(String[] args) {
        TranficCalculate calculate = new TranficCalculate();
        calculate.setStrategy(new BusStrategy());
        int price = calculate.calculatePrice(km);
    }
    
    CalculateStrategy strategy;
    public void setStrategy(CalculateStrategy strategy) {
        this.strategy = strategy;
    }
    public int calculatePrice(int km) {
        return strategy.calculatePrice(km);
    }
}

```

-----------------------------------------------------

#### 7.3 Android 源码中的策略模式

##### 7.3.1 时间插值器和估值器

**时间插值器作用：**通过时间流逝的百分比，计算出当前属性值改变的百分比。

**估值器作用：**通过时间插值器获得的属性百分比，计算出当前属性值改变的值，设置给 View。

##### 7.3.2 动画中的时间插值器

当调用 View.startAnimation(Animation a) 启动动画：

1. 初始化动画开始时间：a.setStartTime();

2. 对 View 设置动画：setAnimation(a);

3. 刷新父类缓存：invalidateParentCaches();

    1. 父类中，会调用 dispatchDraw() 对 View 进行重绘，最终调用 drawChild(),内部调用 child.draw();

    2. 在 View 的 draw 方法中，会查看是否清除动画，获取动画，绘制动画（drawAnimation）等操作。

    3. 绘制动画：

        1. 获取是否初始化过动画，未初始化时去初始化，然后获取 Transformation 对象，储存动画信息。

        2. 通过 Animation 的 getTransformation()，获取动画的相关值。

            1. 计算时间流逝百分比，判断是否动画已完成

            2. 通过插值器获取属性改变的百分比：getInterpolation() 获取插值器

            3. 应用动画效果：applyInterpolation()，通过不同动画的这个方法，改变了 View 的属性。

            4. 如果动画执行完，出发完成回调或者重复执行等操作

        3. 根据具体实现，判断当前动画类型，是否要进行位置大小调整，然后刷新不同区域值。

        4. 获取重汇区域，重新计算有效区域，更新这块区域。

4. 刷新本身及子 View：invalidate();

-----------------------------------------------------

#### 7.4 深入属性动画

>属性动画在 Android 3.0 版本以上使用，兼容低版本可以使用 NineOldAnimations 兼容库。

##### 7.4.1 属性动画的总体设计

Animator 通过 PropertyValuesHolder 来更新目标属性，如果没有设置目标属性的 Property 对象，会通过反射调用目标属性的 setter 方法更新属性值；否则，就会调用 Property 的 set 方法来设置属性值。属性值通过 KeyFrameSet 利用插值器和估值器的计算在动画执行中，不断计算当前属性值，然后更新属性值形成动画。

##### 7.4.2 属性动画的核心类

* ValueAnimator：Animation 的子类，实现动画的整个逻辑。
* ObjectAnimator：ValueAnimation 的子类，对象属性动画操作类，通过使用动画的形式操作对象的属性。
* TimeInterpolator：时间插值器，根据时间流逝的百分比计算当前的属性值改变的百分比。系统预设插值器：线性插值器，加速减速插值器，减速插值器。
* TypeEvaluator：类型估值算法，根据当前属性值改变的百分比来计算改变后的属性值。系统预设：针对整型属性，浮点型属性，Color 属性。
* Property：属性对象，主要定义了属性的 set 和 get 方法。
* PropertyValuesHolder：持有目标属性 Property，setter 和 getter 方法以及关键帧集合和类。
* KeyFrameSet：储存一个动画的关键帧集合。
* AnimationProxy：在 Android 3.0 以下版本 View 使用属性动画的辅助类。

##### 7.4.3 基本使用

* **改变一个对象的 translationY 属性，在 Y 轴上平移一段距离，在默认时间内完成：**

```
ObjectAnimator.ofFloat(myObject, "translationY", -myObject.getHeight()).start();
```

* **改变一个 View 的背景色属性：**

```
ValueAnimator colorAnim = ObjectAnimator.ofInt(this, "backgroundColor", 0xFFFF8080, 0xFF8080FF);
colorAnim.setDuration(3000); // 3秒
colorAnim.setEvaluator(new ArgbEvaluator()); // 颜色估值器
colorAnim.setRepeatCount(ValueAnimator.INFINITE); // 无限循环
colorAnim.setRepeatMode(ValueAnimator.REVERSE); // 反转
colorAnim.start();
```

* **动画集合，5秒内对 View 旋转，平移，缩放，透明度都进行改变：**

```
AnimatorSet set = new AnimatorSet();
set.playTogethor(
    ObjectAnimator.ofFloat(myView, "rotationX", 0, 360),
    ObjectAnimator.ofFloat(myView, "rotationY", 0, 180),
    ObjectAnimator.ofFloat(myView, "rotation", 0, -90),
    ObjectAnimator.ofFloat(myView, "translationX", 0, 90),
    ObjectAnimator.ofFloat(myView, "translationY", 0, 90),
    ObjectAnimator.ofFloat(myView, "scaleX", 1, 1.5f),
    ObjectAnimator.ofFloat(myView, "scaleY", 1, 0.5f),
    ObjectAnimator.ofFloat(myView, "alpha", 1, 0.25f, 1)
    );
set.setDuration(5000).start();
```

* **调用属性动画特有的 animate() 方法，两秒，y 轴旋转720°，平移到（100，100）的位置：**

```
Button buttion = (Button) findViewById(R.id.buttion);
animate(buttion).setDuration(2000).rotationYBy(720).x(100).y(100);
```

##### 7.4.4 流程图

1. **ValueAnimation：**

    开始 —— 设置动画时间，目标对象，属性值 —— 启动动画 —— 是否延时执行

    1. 否 ——

    2. 是 —— 将动画放入队列 —— 通过 handler 发送延时消息来延后执行 —— 

    执行动画 —— 根据双器算出属性值 —— 是否设置了 Property ——

    1. 是 —— 通过 Property 的 set 方法设置属性值 —— 

    2. 否 —— 通过反射调用属性的 set 方法更新属性值 ——

    动画是否结束 —— 

    1. 否 —— 根据双器算出属性值 —— 。。。

    2. 是 —— 结束动画


2. **ObjectAnimation：**

    开始 —— 设置动画时间，目标对象，属性值 —— 启动动画 —— 是否延时执行

    1. 否 ——

    2. 是 —— 将动画放入队列 —— 通过 handler 发送延时消息来延后执行 —— 

    执行动画 —— 根据双器算出属性值 —— 系统版本是否小于 API 11 —— 

    1. 通过 Matrix 实现动画效果 —— 

    2. 通过 Android 3.0 以后的 setter 方法实现动画 —— 

    动画是否结束 —— 

    1. 否 —— 根据双器算出属性值 —— 。。。

    2. 是 —— 结束动画

##### 7.4.4 核心原理分析

从 ObjectAnimation.ofFloat() 分析属性动画原理：

1. 构建属性动画：new ObjectAnimation();

2. 设置属性值：anim.setFloatValues(values); Values 是一个值，是目标值；两个值，一个初始值，一个目标值。通过判断 Values 的值和 mProperty 的值，选择传入不同的值到 setValues() 方法中。

    1. 在设置属性值的方法中，用到 PropertyValuesHolder.ofFloat() ，核心类，作用是保存属性的名称和 setter、getter 方法以及目标值。

        1. 构建 FloatPropertyValuesHolder

        2. 构造函数中，设置目标属性值 setFloatValues(values)，方法内调用了 KeyFrameSet.ofFloat(values)：

        3. 调用父类的设置值的方法

        4. 获取关键帧：mFloatKeyFrameSet

3. 返回 anim。

4. 设置完属性值之后，调用 start() 方法，开始执行动画：判断 looper 是否为空；设置基本状态；启动动画；将动画加到队列；是否延时；发送启动动画的消息。

5. 在 handler 中，收到消息：获取延时和要播放的动画列表；

    1. Animation_start：获取还未执行的动画列表，start 中添加的就是这个列表；复制一份等待执行的动画列表；循环取出动画，如果延时为 0：启动动画（startAnimation()），不为 0：添加到延时列表。

    2. startAnimation()：初始化动画，对系统版本进行判断；将动画添加到 sAnimation 集合中；回调动画开始的 Hook

    3. Animation_frame：获取正在执行的动画数量，遍历准备播放和延时播放的动画，如果到了播放的时间，则播放动画，然后遍历立即执行 sAnimation 里表中的动画，最后，如果动画列表的动画没有放完，再发一个 Action_frame 的消息，使它继续循环 Animation_frame 这个处理流程，知道全部动画放完。

-----------------------------------------------------

#### 7.5 策略模式实战

**图片加载不仅需要顺序加载，同事需要逆序加载：**

加载策略接口：

```

public interface Policy {
    public int compare(Request r1, Request r2); 
}

```

顺序加载策略：

```

public class SerialPolicy implements Policy {
    public int compare(Request r1, Request r2) {
        return r1.serialNum - r2.serialNum;
    } 
}

```

逆序加载策略：

```

public class ReversePolicy implements Policy {
    public int compare(Request r1, Request r2) {
        return r2.serialNum - r1.serialNum;
    } 
}

```

给 Request 增加实现 Comparable 接口：

```

public class ReversePolicy implements Comparable {
    SerialPolicy mSerialPolicy = new SerialPolicy();
    public int compareTo(Request r) {
        return mSerialPolicy.compare(this, r);
    } 
}

```

将请求加到队列中：

```

public void displayImage(...) {
    Request r = new Request();
    // 加载配置对象
    r.config = ...
    // 设置加载策略
    r.setLoadPolicy(mConfig.loadPolicy);
    // 添加到优先级队列，会进行排序
    mImageQueue.add(r);
}

```

-----------------------------------------------------

#### 7.6 总结

* 优点：结构清晰、直观；耦合度相对低、拓展方便；封装彻底、数据安全。

* 缺点：增加子类。

-----------------------------------------------------





## 第八章 随遇而安——状态模式

#### 8.1 状态模式介绍

**定义：**当一个对象内在状态改变时允许改变其行为，这个对象看起来像是改变了其类。

**场景：**

* 一个对象的行为取决于他的状态，并且它必须在运行时根据状态改变他的行为。
* 代码中包含大量与对象状态有关的条件语句。（if-else、switch）

**角色：**

* **Context**：用来操作策略的上下文环境
* **State**：状态的抽象类
* **ConcreteStateA/B**：具体的状态实现

-----------------------------------------------------

#### 8.2 实现方式

```

public interface TVState {
    void nextChannel();
    void prevChannel();
}

public class PowerOffState implements TVState {

    void nextChannel() {
        // 无效果
    }

    void prevChannel() {
        // 无效果
    }
}

public class PowerOnState implements TVState {

    void nextChannel() {
        // 下一个频道
    }

    void prevChannel() {
        // 上一个频道
    }
}

public interface PowerController {
    void powerOn();
    void powerOff();
}

public class TVController implements PowerController {
    TVState mTVState;

    public void setTVState(TVState tvState) {
        mTVState = tvState;
    }

    void powerOn() {
        setTVState(new PowerOnState())
        // 开机
    }

    void powerOff() {
        setTVState(new PowerOffState())
        // 关机
    }

    public void turnUp() {
        mTVState.turnUp();
    }

    public void turnDown() {
        mTVState.turnDown();
    }

    public void nextChannel() {
        mTVState.nextChannel();
    }

    public void prevChannel() {
        mTVState.prevChannel();
    }
}

public class Client {
    public static void main(String[] args) {
        TVController mTVController = new TVController();

        mTVController.powerOn();

        mTVController.nextChannel();

        mTVController.prevChannel();

        mTVController.powerOff();
    }
}
```

-----------------------------------------------------

#### 8.3 WiFi 管理中的状态模式

##### WiFi 设置页面：WiFiSetting 一个 Fragment

1. 构造中，创建监听 WiFi 状态的广播；在 onResume 中注册监听；onActivityCreated 中获取 WiFiManager，初始化空列表；

2. SwitchBar：WiFi 的开关。

3. WiFiEnable：创建一个广播来监听 WiFi 状态的改变，并且实现了按钮的状态改变监听，当状态改变时被广播接收器接收到，通过 handleWiFiStateChanged 函数修改按钮状态。

4. WiFi 状态修改后，会导致 SwitchBar 按钮状态的修改，又会触发 onCheckedChanged() 回调函数，调用 WiFiManager.setWifiEnabled(isChecked) 来修改 WiFi 状态。

5. setWifiEnabled 中实际是调用 WiFiService 的方法来改变WiFi状态。（WiFiService 同 AMS、WMS 一样，启动时一起被注入到 ServiceManager 中）

6. WiFiService 中，通过 WiFiController 用 SmHandler 发送消息，在 SMHandler 中，处理消息，并且返回处理的状态，然后执行状态转换。

7. 设置新的 WiFi 状态（状态模式雏形）

##### WWiFi 状态模式：

1. State 基类：enter，exit，processMessage 三个函数，在进入状态，退出状态和处理消息时调用。

2. WifiStateMachine：都早函数中，通过调用 addState() 方法定义各个状态切换的层级

3. 不同的 State 实现类的三个相同方法实现不同，来处理不同状态下的逻辑。

-----------------------------------------------------

#### 8.4 状态模式实战

在 App 中，通常登录和未登录状态下，相同的实现产生的逻辑不同，比如登录状态下的转发按钮会调用转发业务逻辑，而未登录状态下的转发按钮会跳到登录页面，通过状态模式实现这种效果。

-----------------------------------------------------

#### 8.5 总结

* 优点：将繁琐的状态判断转化成清晰的状态类族，也保证了可拓展性和可维护性。

* 缺点：增加系统类和对象的个数。

-----------------------------------------------------





## 第九章 使编程更有灵活性——责任链模式

#### 9.1 责任链模式介绍

**定义：**使多个对象都有机会处理请求，从而避免了请求的发送者和接受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有对象处理它为止。

**场景：**

* 多个对象可以处理同一请求，但具体由哪个对象处理则在运行时动态决定。
* 在请求处理者在不明确的情况下，向多个对象中的一个提交请求。
* 需要动态指定一组对象处理请求。

**角色：**

* **Handler**：抽象处理者
* **ConcreteHandler**：具体处理者

-----------------------------------------------------

#### 9.2 实现方式

**公司报销系统：**

```

/**
 * 抽象领导者
 */
public abstract class Leader {
    protect Leader nextHandler;

    public void handleRequtest(int money){
        if(money <= limit()) {
            handle(money);
        } else {
            if(null != nextHandler) {
                nextHandler.handleRequtest(money);
            }
        }
    }

    public abstract int limit();

    public abstract void handle(int money);
}

/**
 * 组长
 */
public class GroupLeader extends Leader {

    public abstract int limit() {
        return 1000;
    }

    public abstract void handle(int money) {
        // 报销1000以下款项
    }
}

/**
 * 主管
 */
public class Director extends Leader {

    public abstract int limit() {
        return 5000;
    }

    public abstract void handle(int money) {
        // 报销6000以下款项
    }
}

/**
 * 经理
 */
public class Manager extends Leader {

    public abstract int limit() {
        return 10000;
    }

    public abstract void handle(int money) {
        // 报销10000以下款项
    }
}

/**
 * 老板
 */
public class Boss extends Leader {

    public abstract int limit() {
        return Integer.MAX_VALUE;
    }

    public abstract void handle(int money) {
        // 报销款项
    }
}

public class XiaoMing {
    public static void main(String[] args) {
        Leader mGroupLeader = new GroupLeader();
        Leader mDirector = new Director();
        Leader mManager = new Manager();
        Leader mBoss = new Boss();

        mGroupLeader.nextHandler = mDirector;
        mDirector.nextHandler = mManager;
        mManager.nextHandler = mBoss;

        int money = 50000;
        mGroupLeader.handleRquest(money);
    }
}
```

-----------------------------------------------------

#### 9.3 Android 源码中责任链模式实现

事件的分发处理，Android 开发艺术探索中的比较好理解。

-----------------------------------------------------

#### 9.4 责任链模式实战

**Android 中有序广播接收者和责任链相思：**

``` 
public class FirstReceiver extends BroadcastReceiver {
    @override
    public void onReceive(Context context, Intent intent) {
        int limit = intent.getIntExtra("limit", -1000);

        if(limit == 1000) {
            // 相关处理
            // 停止传播广播
            abortBroadcast();
        } else {
            // 发送给下一个接受者
            setResultExtras();
        }
    }
}
```

-----------------------------------------------------

#### 9.5 总结

* 优点：请求者与处理者解耦，提高灵活性。

* 缺点：会遍历所有调用者，影响性能。

-----------------------------------------------------





## 第十章 化繁为简的翻译机——解释器模式

#### 10.1 解释器模式介绍

**定义：**给定一个语言，定义一个文法的一种表示，并定义一个解释器，该解释器使用该表示来解释语言中的句子。

**场景：**

* 如果某个简单的语言需要解释执行而且可以将该语言的语句表示为一个抽象的语法树时。
* 在某些特定的领域出现不断重复的问题时，可以将该领域的问题转化成一种语法规则下的领域，然后构建解释器来解释该语句。

**角色：**

* **AbstractExpression**：抽象表达式
* **TerminalExpression**：终结符表达式
* **NonterminalExpression**：非终结符表达式
* **Context**：上下文
* **Client**：客户端

-----------------------------------------------------

#### 10.2 解释器模式简单实现

**利用算数器解释器来算加法：**

```

/**
 * 抽象算法表示
 */
public abstract class ArithmeticExpression {
    public abstract int interpret();
}

/**
 * 数字解释器
 */
public class NumberExpression extends ArithmeticExpression {
    private int num;

    public NumberExpression(int num) {
        this.num = num;
    }

    public int interpret() {
        return num;
    }
}

/**
 * 运算符解释器
 */
public abstract class OperatorExpression extends ArithmeticExpression {
    private ArithmeticExpression exp1;
    private ArithmeticExpression exp2;

    public NumberExpression(ArithmeticExpression exp1,
        ArithmeticExpression exp2) {
        this.exp1 = exp1;
        this.exp2 = exp2;
    }
}

/**
 * 加法运算符解释器
 */
public abstract class AdditionExpression extends OperatorExpression {
    public AdditionExpression(ArithmeticExpression exp1,
        ArithmeticExpression exp2) {
        super(exp1, exp2);
    }

    @override
    public int interpret() {
        return exp1.interpret() + exp2.interpret();
    }
}

public class Calculator {
    private Stack<ArithmeticExpression> mExpStack = new Stack<>();

    public Calculator(String expression) {
        ArithmeticExpression exp1, exp2;

        String[] elements = expression.split(" ");

        for(int i; i < elements.length; i++) {
            switch(elements[i]) {
                cass "+":
                    exp1 = mExpStack.pop();
                    exp2 = new NumberExpression(Integer.valueOf(elements[++i]));
                    mExpStack.push(new AdditionExpression(exp1, exp2));
                    break;

                default:
                    mExpStack.push(new NumberExpression(Integer.valueOf(elements[i])));
                    break;
            }
        }
    }

    public int calculator() {
        return mExpStack.pop().interpret();
    }
}

public class Client {
    public static void main(String[] args) {
        Calculator c = new Calculator("111 + 222 + 333 + 444");
        // 获取加法字符串结果
        int result = c.calculator();
    }
}
```

-----------------------------------------------------

#### 10.3 Android 源码中的解释器模式实现

##### 10.3.1 PackageParser 对 AndroidManifest.xml 文件中的每个组件标签创建了相应的类用于储存相应的信息：

1. PackageParser 按照解释器模式在以内部类方式创建了 Activity, Service, Provider, Permission 等构件对应的类，与 Manifest 中的标签对应。

2. 在 Android 启动解析安装的应用包时，调用 PackageManagerService 的方法 scanPackageLI(File file) 和 scanPackageLI(PackageParser.Package pkg)。第一个方法是将 apk 文件解析获取解析器，然后传入第二个方法，进行解析保存在 PSM。

3. 在 scanPackageLI() 方法中，调用 PackageParser 的解析方法 parsePackage(), 该方法同上有两个不同参数的方法 parsePackage(File file) 和 parsePackage(Resource res)，调用同上，第一个方法调用第二个方法。

4. 在 parsePackage() 方法中，对 AndroidManifest 文件内的各个节点标签就行解析，对 Application 标签调用 parseApplication() 方法，对标签内的 Activity，Service 等标签进行解析和创建。注意：对 Activity 的解析方法 parseActivity() 也对 broadcast 进行解析。

##### 10.3.2 PackageManagerService

1. 与其他服务一样，PMS 也是由 SystemServer 启动。在 SystemServer 的 initAndLoop() 方法中，调用 PMS 的静态 main() 方法，然后内部创建 PSM 保存在 ServiceManager 中。ServiceManager 是 Binder 进程间通信的守护线程，管理 Android 系统中的 Binder 对象。

2. PMS 的作用就是管理应用程序包。

-----------------------------------------------------

#### 10.4 总结

* 优点：灵活的拓展性。

* 缺点：会产生更多的类，复杂的文法会让代码树看起来繁琐，不推荐复杂的文法。

-----------------------------------------------------





## 第十一章 让程序通常执行——命令模式

#### 11.1 命令模式介绍

**定义：**将一个请求封装成一个对象，从而让用户使用不同的请求把客户端参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。

**场景：**

* 需要抽象出待执行的动作，然后以参数的形式提供出来——类似于过程设计中的回调机制，而命令模式正是回调机制的一个面向对象的替代品。
* 在不同时刻指定、排列和执行请求。一个命令对象可以有与初始请求无关的生存期。
* 需要支持取消操作。
* 支持修改日志功能，在系统崩溃时，这些修改可以重新被做一遍。
* 需要支持事务操作。

**角色：**

* **Receiver**：接受者：执行具体逻辑
* **Command**：命令者：所有具体命令类的接口
* **ConcreteCommand**：命令者实现类：用来执行调用接受者的相关方法，加以耦合
* **Invoker**：请求者：调用命令对象执行请求。
* **Client**：客户端

-----------------------------------------------------

#### 11.2 命令模式简单实现

**俄罗斯方块控制按钮实现：**

```

public class TetrisMachine {
    public void toLeft(){
        // 向左操作
    }
    public void toRight(){
        // 向右操作
    }
    public void fastToBottom(){
        // 快速落下
    }
    public void transform(){
        // 改变形状
    }
}

public interface Command {
    void execute();
}

public class LeftCommand implements Command {
    private TetrisMachine machine;
    public LeftCommand(TetrisMachine machine) {
        this.machine = machine;
    }
    public void execute() {
        if(machine!=null) {
            machine.toLeft();
        }
    }
}

public class RightCommand implements Command {
    private TetrisMachine machine;
    public RightCommand(TetrisMachine machine) {
        this.machine = machine;
    }
    public void execute() {
        if(machine!=null) {
            machine.toRight();
        }
    }
}

public class FallCommand implements Command {
    private TetrisMachine machine;
    public FallCommand(TetrisMachine machine) {
        this.machine = machine;
    }
    public void execute() {
        if(machine!=null) {
            machine.fastToBottom();
        }
    }
}

public class TransformCommand implements Command {
    private TetrisMachine machine;
    public TransformCommand(TetrisMachine machine) {
        this.machine = machine;
    }
    public void execute() {
        if(machine!=null) {
            machine.transform();
        }
    }
}

public class Buttons {
    private LeftCommand mLeftCommand;
    private RightCommand mRightCommand;
    private FallCommand mFallCommand;
    private TransformCommand mTransformCommand;

    // 四种命令的 set 方法
    ...

    // 四种按钮命令的请求
    public void toLeft() {
        mLeftCommand.execute();
    }
    ...
}
```

>**设计模式的重要原则之一：**对修改关闭对拓展开放。

-----------------------------------------------------

#### 11.3 Android 源码中的命令模式实现

##### Android 事件机制中底层逻辑对事件转发的处理

    底层 C 的代码不做记录了

-----------------------------------------------------

#### 11.4 Android 事件输入系统介绍

1. InputReader 将事件从硬件节点中读取后转化为一个 Event 事件，然后将事件传给 InputDispatcher 将事件分发给合适的窗口并监听 ANR 的发生；然后创建 InputReader 和 InputDispatcher 的父亲 InputManager 由其创建两者并提供 policy 对事件进行预处理；最后涉及系统的 Service 以及面向用户的几个模块：ActivityManager、WindowManager 和 View 等。

2. 负责管理事件输入的事 InputManagerService 也是系统级服务之一，协调第一步中的四个部分。

-----------------------------------------------------

#### 11.5 命令模式实战

##### 画板：

```

// 接受者
public interface IBrush {
    void down(Path path, float x, float y);
    void move(Path path, float x, float y);
    void up(Path path, float x, float y);
}

public class NormalBrush implements IBrush{
    public void down(Path path, float x, float y){
        path.moveTo(x, y);
    }
    public void move(Path path, float x, float y){
        path.lineTo(x, y);
    }
    public void up(Path path, float x, float y){
    }
}

public class CircleBrush implements IBrush{
    public void down(Path path, float x, float y){
    }
    public void move(Path path, float x, float y){
        path.addCircle(x, y, 10, Path.Direction.CW);
    }
    public void up(Path path, float x, float y){
    }
}

// 命令者
public interface IDraw {
    void draw(Canvas canvas);
    void undo();
}

public class DrawPath implements IDraw {
    public Path path;
    public Paint paint;
    void draw(Canvas canvas) {
        canvas.drawPath(path, paint);
    }
    void undo() {}
}

// 一大堆请求类，绘制的执行者画布
...
```

-----------------------------------------------------

#### 11.6 总结

* 优点：降低耦合，提高灵活控制性和拓展性。

* 缺点：类的膨胀，大量衍生类创建

-----------------------------------------------------





## 第十二章 解决、解耦的钥匙——观察者模式

#### 12.1 观察者模式介绍

**定义：**定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于他的对象都会得到通知并被自动更新。

**场景：**

* 关联行为场景，是可拆分的，而不是“组合”关系。
* 事件多级出发场景。
* 跨系统的消息交换场景，如消息队列、时间总线的处理机制。

**角色：**

* **Subject**：抽象主题：被观察的角色，提供添加和删除观察者的接口，可将任意观察者放到集合中。
* **ConcreteSubject**：具体主题：将状态存入观察者对象，具体主题状态发生变化通知所有观察者。
* **Observer**：抽象观察者：定义更新接口，用于主题更新时更新自己。
* **ConcreteObserver**：具体观察者：实现更新接口。

-----------------------------------------------------

#### 12.2 观察者模式简单实现

##### 程序员订阅技术周报每周接通知

**程序员观察者：**

```

public class Coder implements Observer {
    public String name;
    public Coder(String name) {
        this.name = name;
    }
    @Override
    public void update(Observable o, Object arg) {
        // 更新代码
    }
    @Override
    public String toString() {
        return "码农：" + name;
    }
}

public class DevTechFrontier extends Observable {
    public void postNewPublication(String content) {
        // 标识状态或者内容发生改变
        setChanged();
        // 通知所有观察者
        notifyObservers(content);
    }
}

public class Client {
    public static void main(String[] args) {
        DevTechFrontier devTechFrontier = new DevTechFrontier();
        Coder coder1 = new Coder("coder1");
        Coder coder2 = new Coder("coder2");
        Coder coder3 = new Coder("coder3");
        Coder coder4 = new Coder("coder4");
        devTechFrontier.addObserver(coder1);
        devTechFrontier.addObserver(coder2);
        devTechFrontier.addObserver(coder3);
        devTechFrontier.addObserver(coder4);
        devTechFrontier.postNewPublication("新的周报发送了！");
    }
}

```

-----------------------------------------------------

#### 12.3 Android 源码分析

##### ListView 添加数据，调用 notifyDataSetChanged() 通知条目更新

1. notifyDataSetChanged() 定义在 BaseAdapter 中，调用了成员变量 DataSetObservable 的 notifyChanged() ，在 BaseAdapter 中有两个方法 registerDataSetObserver() 和 unregisterDataSetObserver() 两个方法来注册和取消观察者。

2. DataSetObservable 的 notifyChanged() 中，循环调用 onChanged() 通知注册的观察者，被观察者状态发生改变。

3. 注册的所有观察者，是在 ListView 的 setAdapter() 方法中。

    1. 如果已经有 adapter，先注销对应的观察者。

    2. 调用父类的 setAdapter() 方法。

    3. 获取数据的数量。

    4. 创建一个 ListView 的成员变量数据集观察者 AdapterDataSetObserver，然后将观察者注册到 adapter 中的 DataSetObservable 中。

4. AdapterDataSetObserver 定义在 ListView 的父类 AbsListView 中，内部实现两个方法：onChanged() 和 onInvalidated()。AdapterDataSetObserver 集成自 AbsListView 的父类 AdapterView 中的 AdapterDataSetObserver。

5. AdapterView 中的 AdapterDataSetObserver：被 Adapter 的 notifyDataSetChanged 被调用时，会调用 Adapter 中的 DataSetObservable 的 notifyChanged() ，会调用 ListView 所有观察者的 onChanged() 方法，其中会调用 ListView 重新布局的函数 requestLayout() 使得刷新界面。

-----------------------------------------------------

#### 12.4 观察者模式的深度拓展




