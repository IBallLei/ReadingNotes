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

1. 与 LayoutInflate 相同，首先通过 Context.getSystemService() 在 ContextImpl 中初始化时注册在装有各类服务的 map 中，获取 WindowManager 的服务实例 WindowManagerImpl。

2. 通过 Dialog 的构造函数传入 context，然后通过 context.getSystemService() 方法获取到 WindowManager 

3. 通过 PolicyManager.makeNewWindow(context) 获取到 Window 对象。

4. 给 Window 设置回调，传入 Dialog 对象本身。

5. 调用 window.setWindowManager(windowManager), 将 Window 与 Manager 关联。在其方法内部通过传入的 windowManager 调用 createLocalWindowManager() 方法，传入 window 本身，用来创建返回 WindowManagerImpl 对象，并赋值给 Window 中的成员变量。

    1. 

6. 









