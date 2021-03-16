# JS前端基础理论-----类理论（设计模式发散）  

设计模式分为三大类：

1. 创建型模式

   共五种：

   1. 工厂方法模式
   2. 抽象工厂模式
   3. 单例模式
   4. 建造者模式
   5. 原型模式。

   > 创建型模式是用来创建对象的模式，抽象了实例化的过程，帮助一个系统独立于其他关联对象的创建、组合和表示方式。所有的创建型模式都有两个主要功能：
   >
   > 　　1.将系统所使用的具体类的信息封装起来
   >
   > 　　2.隐藏类的实例是如何被创建和组织的。外界对于这些对象只知道他们共同的接口，而不清楚其具体的实现细节。
   >
   > 正因为以上两点，创建型模式在创建什么（what）、由谁来创建（who）、以及何时创建（when）这些方面，都为设计者提供了尽可能大的灵活性。

2. 结构型模式

   共七种：

   1. 适配器模式
   2. 装饰器模式
   3. 代理模式
   4. 外观模式
   5. 桥接模式
   6. 组合模式
   7. 享元模式。

   > 结构型模式讨论的是类和对象的结构，它采用继承机制来组合接口或实现（类结构型模式），或者通过组合一些对象实现新的功能（对象结构型模式）。这些结构型模式在某些方面具有很大的相似性，但侧重点各有不同。

3. 行为型模式

   共十一种：

   1. 策略模式
   2. 模板方法模式
   3. 观察者模式
   4. 迭代子模式
   5. 责任链模式
   6. 命令模式
   7. 备忘录模式
   8. 状态模式
   9. 访问者模式
   10. 中介者模式
   11. 解释器模式。

   > 行为型设计模式关注的是对象的行为，用来解决对象之间的联系问题。

   ## 二、设计模式的六大原则

   ### 总原则：开闭原则（Open Close Principle）

   开闭原则就是说**对扩展开放，对修改关闭**。在程序需要进行拓展的时候，不能去修改原有的代码，而是要扩展原有代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类等，后面的具体设计中我们会提到这点。

   ### 1、单一职责原则

   不要存在多于一个导致类变更的原因，也就是说每个类应该实现单一的职责，如若不然，就应该把类拆分。

    

   ### 2、里氏替换原则（Liskov Substitution Principle）

   里氏代换原则(Liskov Substitution Principle LSP)面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。—— From Baidu 百科

   历史替换原则中，子类对父类的方法尽量不要重写和重载。因为父类代表了定义好的结构，通过这个规范的接口与外界交互，子类不应该随便破坏它。

    

   ### 3、依赖倒转原则（Dependence Inversion Principle）

   这个是开闭原则的基础，具体内容：面向接口编程，依赖于抽象而不依赖于具体。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互。

    

   ### 4、接口隔离原则（Interface Segregation Principle）

   这个原则的意思是：每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分。使用多个隔离的接口，比使用单个接口（多个接口方法集合到一个的接口）要好。

    

   ### 5、迪米特法则（最少知道原则）（Demeter Principle）

   就是说：一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类。

   最少知道原则的另一个表达方式是：只与直接的朋友通信。类之间只要有耦合关系，就叫朋友关系。耦合分为依赖、关联、聚合、组合等。我们称出现为成员变量、方法参数、方法返回值中的类为直接朋友。局部变量、临时变量则不是直接的朋友。我们要求陌生的类不要作为局部变量出现在类中。

    

   ### 6、合成复用原则（Composite Reuse Principle）

   原则是尽量首先使用合成/聚合的方式，而不是使用继承。

   ## 三、常用设计模式

   ### 单例模式(Singleton)--单线程

   保证一个类仅有一个实例，并提供一个访问它的全局访问点，避免一个全局使用的类频繁的创建和销毁，节省系统资源，提高程序效率。怎么创建唯一的实例？Java是这么创建实例的 Person p = new Person();但是这么创建会创建多个实例，所以我们必须把构造器设为私有，这样其他类就不能使用new来实例化一个类。

   ```
   //单线程实现单例模式
   class Singleton {
       private static Singleton instance;
       
       public Singleton() {};
       
       public static Singleton getInstance (){
           instance = new Singleton();
           
           return instance;
       }
   }
    
   public class Main {
    
       public static void main(String[] args) {
           Singleton s1 = Singleton.getInstance();
           Singleton s2 = Singleton.getInstance();
           //判断两个实例s1 s2是否为同一个实例
           System.out.println(s1 == s2);
       }
    
   }public class Singleton {
    
       //定义一个属性,用来保存Singleton类对象的实例
       private static Singleton instance;
    
       //私有构造器,该类不能被外部类使用new方式实例化
       private Singleton(){
    
       }
    
       //外部通过该方法获取Singleton类的唯一实例
       public static Singleton getInstance(){
           if (instance == null) {
               instance = new Singleton();
           }
           return instance;
       }
   }
   ```

    

   这种实现方式并不是线程安全的，当有多个线程同时调用Singleton.getInstance()方法时会产生多个实例。下节我们来学习多线下如何实现单例模式。

   Java多线程程序，线程执行顺序是不确定的，所以在同时多个线程调用Singleton.getInstance()方法时，存在创建多个实例的可能，会引起程序执行错误。那我们该如何实现多线程下安全的创建一个唯一的实例呢？锁，加锁。在线程调用Singleton.getInstance()方法时，判断instance == null ? 是，加锁，其他线程这时只能等待这个线程释放锁，才能进入临界区。那如何加锁，可以使用synchronized。

   ```
    public static Singleton getInstance() {
           //synchronized加锁同步会降低效率,这里先判断是否为空
           //不为空则不需要加锁,提高程序效率
           if (instance == null) {
               synchronized (Singleton.class) {
                   if (instance == null) {
                       instance = new Singleton();
                   }
               }
           }
           return instance;
       }
   ```

    

   ### 单例模式优点

   - 1 在内存中只有一个对象，节省内存空间。
   - 2 避免频繁的创建销毁对象，可以提高性能。
   - 3 避免对共享资源的多重占用。
   - 4 可以全局访问。

   ### 适用场景

   - 1 需要频繁实例化然后销毁的对象。
   - 2 创建对象时耗时过多或者耗资源过多，但又经常用到的对象。
   - 3 有状态的工具类对象。
   - 4 频繁访问数据库或文件的对象。
   - 5 以及其他我没用过的所有要求只有一个对象的场景。

   ### 工厂方法模式(Factory Method)

   定义一个用于创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类的实例化延迟到其子类。

   类图：

   ![img](https://img-blog.csdn.net/20180825225912104?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    

   - 1.很多工厂都有一些相同的行为，比如汽车工厂。我们需要抽象这些相同的行为成接口，每个工厂都实现这个接口。

   ```
   public interface IFactory {
    
       public void createProduct();
   }
   ```

    

   - 2.生产相同的产品每个工厂所使用的方法可能不同，所以具体如何生产产品由具体工厂实现。

   ```
   public class Factory implements IFactory {
    
       @Override
       public void createProduct() {
    
       }
   }
   ```

    

   ### 工厂模式两要点：

   -  

   1.工厂接口是工厂方法模式的核心，与调用者直接交互用来提供产品。

   2.工厂实现决定如何实例化产品，是实现扩展的途径，需要有多少种产品，就需要有多少个具体的工厂实现。

   -  

   ### 适用场景：

   -  

   1.在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过new就可以完成创建的对象，无需使用工厂模式。

   2.工厂模式是一种典型的解耦模式，迪米特法则在工厂模式中表现的尤为明显。假如调用者自己组装产品需要增加依赖关系时，可以考虑使用工厂模式。将会大大降低对象之间的耦合度。

   3.当需要系统有比较好的扩展性时，可以考虑工厂模式，不同的产品用不同的实现工厂来组装。

    

    

   ### 抽象工厂模式(Abstract Factory)

   为创建一组相关或相互依赖的对象提供一个接口，而且无需指定他们的具体类。

   抽象工厂是工厂模式的升级版，他用来创建一组相关或者相互依赖的对象。来看下抽象工厂模式的类图：

   ![img](https://img-blog.csdn.net/20180825234146542?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    

   上节学习了工厂模式，类的创建依赖工厂类，程序需要扩展时，我们必须创建新的工厂类。工厂类是用来生产产品的，那我们也可以把“工厂类当成我们要生产的产品”，所以抽象工厂就是“工厂的工厂”，即生产工厂的工厂。下面通过一个例子来深入理解。

   通过一个例子，来加深对抽象工厂的理解。

   ```
   //CPU工厂接口
   public interface CPUFactory {
       public void createCPU();
   }
   //IntelCPU工厂
   public class IntelCPU implements CPUFactory {
       @Override
       public void createCPU() {
           System.out.println("Intel CPU");
       }
   }
   //AMDCPU工厂
   public class AMDCPU implements CPUFactory {
       @Override
       public void createCPU() {
           System.out.println("AMD CPU");
       }
   }
   //创建抽象工厂类接口
   public interface Provider {
       public CPUFactory createCPUFactory();
   }
   public class InterCPUFactory implements Provider {
       @Override
       public CPUFactory createCPUFactory() {
           return new InterCPU();
       }
   }
   public class AMDCPUFactory implements Provider {
       @Override
       public CPUFactory createCPUFactory() {
           return new AMDCPU();
       }
   }
   public static void main(String[] args) {
           //创建一个生产CPU工厂的工厂
           Provider cpufactory = new InterCPUFactory();
           //通过CPU工厂的工厂创建一个IntelCPU工厂
           CPUFactory intelcpu = cpufactory.createCPUFactory();
           //IntelCPU工厂生产intelCPU
           intelcpu.createCPU();}
   ```

    

   ### 抽象工厂的优点：

   抽象工厂模式除了具有工厂方法模式的优点外，最主要的优点就是可以在类的内部对产品族进行约束。所谓的产品族，一般或多或少的都存在一定的关联（例如不同厂商生产CPU）。

   ### 适用场景：

   一个继承体系中，如果存在着多个等级结构（即存在着多个抽象类），并且分属各个等级结构中的实现类之间存在着一定的关联或者约束，就可以使用抽象工厂模式。

    

   ### 建造者模式(Builder)

   将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，由于需求的变化，这个复杂对象的某些部分经常面临着剧烈的变化，一些基本部件不会变。所以需要将变与不变分离。与抽象工厂的区别：在建造者模式里，有个指导者(Director)，由指导者来管理建造者，用户是与指导者联系的，指导者联系建造者最后得到产品。即建造者模式可以强制实行一种分步骤进行的建造过程。

   建造者类图：

   ![img](https://img-blog.csdn.net/20180825234252635?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    

   建造者模式四要素：

   -  

   1.产品类Product：一般是一个较为复杂的对象，也就是说创建对象的过程比较复杂，一般会有比较多的代码量。

   2.抽象建造者类Builder: 将建造的具体过程交与它的子类来实现，这样更容易扩展。

   3.建造者类ConcreteBuilder: 组建产品；返回组建好的产品。

   4.指导类Director: 负责调用适当的建造者来组建产品，指导类一般不与产品类发生依赖关系，与指导类直接交互的是建造者类。

   似乎很抽象。举个例子：前面你创建了一个生产保时捷的工厂，生产一台保时捷911需要的主要部件都一样(引擎，轮子，方向盘...)和流程是不变的，变的是引擎，轮子，控制系统等等部件具体实现，这些部件的生产交由具体的builder去生产。

   ```
   /抽象生产者
   public interface Builder {
    
       void buildPartA();
       void buildPartB();
       void buildPartC();
    
       Product buildProduct();
   }
   //具体生产者
   public class ConcreteBuilder implements Builder {
    
       Product product;
    
       @Override
       public void buildPartA() {
    
       }
    
       @Override
       public void buildPartB() {
    
       }
    
       @Override
       public void buildPartC() {
    
       }
    
       @Override
       public Product buildProduct() {
           return product;
       }
   }
   //产品由各个组件组成
   public class Product {
    
       //partA
       //partB
       //partC
   }
   //指导者,产品生产流程规范
   public class Director {
    
       Builder builder;
       //由具体的生产者来生产产品
       public Director(Builder builder) {
           this.builder = builder;
       }
       //生产流程
       public void buildProduct(){
           builder.buildPartA();
           builder.buildPartB();
           builder.buildPartC();
       }
   }
   public static void main(String[] args) {
           //只需要关心具体建造者,无需关心产品内部构建流程。
           //如果需要其他的复杂产品对象，只需要选择其他的建造者.
           Builder builder = new ConcreteBuilder();
           //把建造者注入指导者
           Director director = new Director(builder);
           //指导者负责流程把控
           director.buildProduct();
           // 建造者返回一个组合好的复杂产品对象
           Product product = builder.buildProduct();
       }
   ```

    

   ### 建造者模式优点：

   -  

   1.建造者模式的封装性很好。使用建造者模式可以有效的封装变化，在使用建造者模式的场景中，一般产品类和建造者类是比较稳定的，因此，将主要的业务逻辑封装在指导者类中对整体而言可以取得比较好的稳定性。

   2.建造者模式很容易进行扩展。如果有新的需求，通过实现一个新的建造者类就可以完成。

    

   ### 适用场景：需要生成的对象具有复杂的内部结构；需要生成的对象内部属性本身相互依赖。

   ###  

   ### 模板方法模式(Template Method)

   定义一个操作中算法的框架，而将一些步骤延迟到子类中，使得子类可以不改变算法的结构即可重定义该算法中的某些特定步骤。

   类图：

   ![img](https://img-blog.csdn.net/20180825234515385?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    

   模板方法模式是编程中经常用到的模式，其非常简单，AbstractClass叫抽象模板，其方法分为3类：

   -  

   1.抽象方法：父类中只声明但不加以实现，而是定义好规范，然后由它的子类去实现。

   -  
   -  

   2.模版方法：由抽象类声明并加以实现。一般来说，模版方法调用抽象方法来完成主要的逻辑功能，并且，模版方法大多会定义为final类型，指明主要的逻辑功能在子类中不能被重写。

   -  
   -  

   3.钩子方法：由抽象类声明并加以实现。但是子类可以去扩展，子类可以通过扩展钩子方法来影响模版方法的逻辑。

   -  

   实现类用来实现细节。抽象类中的模版方法正是通过实现类扩展的方法来完成业务逻辑。

   现在要实现一个对无序数组从小到大排序并打印数组的类。排序算法有很多种，打印功能固定的。定义一个AbstractClass定义抽象排序方法由子类去实现；模板类实现打印方法。

   ```
   //抽象模板类
   public abstract class AbstractSort {
    
       public abstract void sort(int[] array);
       //防止子类覆盖使用final修饰
       public final void printArray(int[] array) {
           sort(array);
           for (int i = 0; i < array.length; i++) {
               System.out.println(array[i]);
           }
       }
   }
   //具体实现类
   public class QuickSort extends AbstractSort {
       @Override
       public void sort(int[] array) {
           //使用快排算法实现
       }
   }
   public class MergeSort extends AbstractSort {
       @Override
       public void sort(int[] array) {
           //使用归并排序算法实现
       }
   }
   public static void main(String[] args) {
           int[] arr = {3,5,2,45,243,341,111,543,24};
           //AbstractSort s = new MergeSort();
           AbstractSort s = new QuickSort();
           s.printArray(arr);
       }
   ```

    

   ### 模板方法模式优点：

   -  

   1.容易扩展。一般来说，抽象类中的模版方法是不易反生改变的部分，而抽象方法是容易反生变化的部分，因此通过增加实现类一般可以很容易实现功能的扩展，符合开闭原则。

   -  
   -  

   2.便于维护。对于模版方法模式来说，正是由于他们的主要逻辑相同，才使用了模版方法。

   -  

   ### 适用场景：

   在多个子类拥有相同的方法，并且这些方法逻辑相同时，可以考虑使用模版方法模式。在程序的主框架相同，细节不同的场合下，也比较适合使用这种模式。

   上节你生产了保时捷911和Cayma，现在要对其进行检测。检测程序顺序：能否启动start,加油门speeup,刹车brake，熄火stop。由于检测标准不一样，请你用模板方法模式来实现。

    

   ### 适配器模式(Adapter Class/Object)

   是指将一个接口转换成客户端希望的另外一个接口，该模式使得原本不兼容的类可以一起工作。

   举个例子：macbook pro有一个HDMI接口，一条HDMI接口的数据线，现在要外接显示器，而显示器只有VGI接口，我们需要一个HDMI-VGI转换器，这个转换器其实起到的作用就是适配器，让两个不兼容的接口可以一起工作。

   类图：

   ![img](https://img-blog.csdn.net/20180825234627657?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   适配器有4种角色：

   -  

   1.目标抽象角色(Target)：定义客户所期待的使用接口。(GVI接口)

   -  
   -  

   2.源角色(Adaptee)：需要被适配的接口。(HDMI接口)

   -  
   -  

   3.适配器角色(Adapter)：把源接口转换成符合要求的目标接口的设备。(HDMI-VGI转换器)

   -  
   -  

   4.客户端(client)：例子中指的VGI接口显示器。

   -  

   继续学习 

   把HDMI接口转换成VGI接口，使得macbook pro可以外接显示器。

   ```
   //HDMI接口，需要被适配的接口
   public interface HDMIPort {
       void workByHDMI();
   }
   //VGI接口，客户端所期待的接口
   public interface VGIPort {
       void workByVGI();
   }
   //将HDMI接口转换为VGI,这就是适配器
   public class HDMIToVGI implements VGIPort{
    
       HDMIPort hdmiPort;
    
       public HDMIToVGI(HDMIPort hdmiPort) {
           this.hdmiPort = hdmiPort;
       }
       //将HDMI接口转换为VGI接口
       @Override
       public void workByVGI() {
           hdmiPort.workByHDMI();
       }
   }
    public static void main(String[] args) {
           //定义一个HDMI接口
           HDMIPort hdmiPort = new HDMIPort() {
               @Override
               public void workByHDMI() {
                   //hdmi接口工作方式
               }
           };
           //将HDMI接口转换为VGI接口
           VGIPort vgiPort = new HDMIToVGI(hdmiPort);
           //经过转换HDMI接口变成了VGI接口
           vgiPort.workByVGI();
       }
   ```

    

   ### 适配器模式优点：

   -  

   1.可以让任何两个没有关联的类一起运行。

   2.提高了类的复用。

   3.增加了类的透明度。

   4.灵活性好。

   适配器模式缺点：过多地使用适配器，会让系统非常零乱，不易整体进行把握。

   ### 适用场景：

   1.系统需要使用现有的类，而此类的接口不符合系统的需要。

   2.想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。

   3.通过接口转换，将一个类插入另一个类系中。

   -  

   ### 外观模式(Facade)

   为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。降低访问复杂系统的内部子系统时的复杂度。

   类图：

   ![img](https://img-blog.csdn.net/20180825234852835?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    

   在客户端和复杂系统之间再加一层，将调用顺序、依赖关系等处理好。举个例子：我们经常用的电脑，开机其实是个非常复杂的过程，而我们只需要按开机按钮就可以了。

   模拟电脑启动，假设电脑启动顺序：启动CPU，启动内存，启动硬盘，加载数据等。

   ```
   public class CPU {
    
       public void startup(){
           System.out.println("启动CPU");
       }
   }
   public class Memory {
    
       public void startup(){
           System.out.println("启动内存");
       }
   }
   public class Disk {
    
       public void startup(){
           System.out.println("启动硬盘");
       }
   }
   //facade
   public class Computer {
    
       CPU cpu;
       Memory memory;
       Disk disk;
    
       public Computer(){
           cpu = new CPU();
           memory = new Memory();
           disk = new Disk();
       }
    
       public void start(){
           cpu.startup();
           memory.startup();
           disk.startup();
       }
   }
   public static void main(String[] args) {
           Computer computer = new Computer();
           //启动computer是个很复杂的过程,我们并不需要知道其启动各个子系统的加载过程
           //只需要调用computer为各个子系统提供统一的一个接口start()就可以启动computer了
           computer.start();
       }
   ```

    

   ### 外观模式优点：

   -  

   1.减少系统相互依赖。

   2.提高灵活性。

   3.提高了安全性。

   ### 适用场景：

    

   1.为复杂的模块或子系统提供外界访问的模块。

   2.客户程序与抽象类的实现部分之间存在着很大的依赖性。引入facade 将这个子系统与客户以及其他的子系统分离，可以提高子系统的独立性和可移植性。

   -  

   ### 装饰器模式(Decorator)

   对客户透明的方式动态地给一个对象附加上更多的责任，同时又不改变其结构。装饰模式可以在不使用创造更多子类的情况下，将对象的功能加以扩展。

   类图：

    

   ![img](https://img-blog.csdn.net/20180825235040302?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   1.抽象构件(Component)角色：给出一个抽象接口，以规范准备接收附加责任的对象。

   2.具体构件(ConcreteComponent)角色：定义一个将要接收附加责任的类。

   3.装饰(Decorator)角色：持有一个构件(Component)对象的实例，并定义一个与抽象构件接口一致的接口。

   4.具体装饰(ConcreteDecorator)角色：负责给构件对象“贴上”附加的责任。

   Java IO中就是典型的装饰器

   ```
   //InputStream提供的基本方法(Component)
   public abstract class InputStream implements Closeable {
    
   }
   //默认目标实现类(ConcreteComponent)
   public class FileInputStream extends InputStream {
    
   }
   /*装饰实现类（FilterInputStream）一定是继承或实现原始接口(InputStream)的，内部有包含一个原始接口的超类(其实就是某个默认目标实现类)*/
   //Decorator
   public class FilterInputStream extends InputStream {
       /**
        * The input stream to be filtered.
        */
       protected volatile InputStream in;
    
       protected FilterInputStream(InputStream in) {
           this.in = in;
       }
   }
   //具体装饰类(ConcreteDecorator)
   public class BufferedInputStream extends FilterInputStream {
    
       public BufferedInputStream(InputStream in) {
           this(in, DEFAULT_BUFFER_SIZE);
       }
   }
   //具体装饰类(ConcreteDecorator)
   public class DataInputStream extends FilterInputStream implements DataInput {
    
       public DataInputStream(InputStream in) {
           super(in);
       }
   }
   ```

    

   ### 装饰器模式优点：

   1.装饰类和被装饰类可以独立发展，不会相互耦合。

   2.装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。就增加功能来说，装饰器模式相比生成子类更为灵活。

   ### 适用场景：

   1.扩展一个类的功能。

   2.动态增加功能，动态撤销。

   ### 观察者模式(Observer)

   对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

   类图：

    

    

   1.抽象主题(Subject)角色：把所有对观察者对象的引用保存在一个集合中，每个抽象主题角色都可以有任意数量的观察者。抽象主题提供一个接口，可以增加和删除观察者角色。一般用一个抽象类和接口来实现。

   2.抽象观察者(Observer)角色：为所有具体的观察者定义一个接口，在得到主题的通知时更新自己。

   3.具体主题(ConcreteSubject)角色：在具体主题内部状态改变时，给所有登记过的观察者发出通知。具体主题角色通常用一个子类实现。

   4.具体观察者(ConcreteObserver)角色：该角色实现抽象观察者角色所要求的更新接口，以便使本身的状态与主题的状态相协调。通常用一个子类实现。如果需要，具体观察者角色可以保存一个指向具体主题角色的引用。

   控件按钮、报警器等都是观察者模式。

   ```
   public interface Subject {
       //添加观察者
       void attach(Observer o);
       //删除观察者
       void detach(Observer o);
       //通知观察者
       void notifyObservers();
       //发生某事
       void doSomeThings()
   }
   //观察者
   public interface Observer {
    
       void update();
   }
   public class ConcreteSubject implements Subject {
    
       ArrayList<Observer> observers = new ArrayList<>();
    
       @Override
       public void attach(Observer o) {
           observers.add(o);
       }
    
       @Override
       public void detach(Observer o) {
           observers.remove(o);
       }
    
       @Override
       public void notifyObservers() {
           for (Observer o : observers) {
               o.update();
           }
       }
    
       public void doSomeThings(){
           //doSomeThings
           notifyObservers();//通知观察者
       }
   }
   //具体观察者
   public class ConcreteObserver implements Observer {
       @Override
       public void update() {
           System.out.println("我观察到subject发生了某事");
       }
   }
   public static void main(String[] args) {
           Subject cs = new ConcreteSubject();
           //添加观察者
           cs.attach(new ConcreteObserver());
           //subject发生了某事，通知观察者
           cs.doSomeThings();
       }
   ```

    

   ### 观察者模式优点：

   -  

   1.观察者和被观察者是抽象耦合的。

   2.建立一套触发机制。

   ### 观察者模式缺点：

    

   1.如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。

   2.如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。

   3.观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

   -  

   ### 适用场景：

   1.当一个抽象模型有两个方面, 其中一个方面依赖于另一方面。将这二者封装在独立的对象中以使它们可以各自独立地改变和复用。

   2.当对一个对象的改变需要同时改变其它对象, 而不知道具体有多少对象有待改变。

   3.当一个对象必须通知其它对象，而它又不能假定其它对象是谁。换言之, 你不希望这些对象是紧密耦合的。

   ### 策略模式(Strategy)

   定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。

   类图：

    

   ![img](https://img-blog.csdn.net/20180825235421747?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlaWp1bndlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   1.Strategy：策略接口，用来约束一系列具体的策略算法。Context使用这个接口来调用具体的策略，实现定义的策略。

   2.ConcreteStrategy：具体的策略实现，也就是具体的算法实现。

   3.Context：上下午，负责与具体的策略交互，通常上下文会持有一个真正的策略实现。

   策略模式是把一个类中经常改变或者将来可能改变的部分提取出来作为一个接口，然后在类中包含这个对象的实例，这样类的实例在运行时就可以随意调用实现了这个接口的类的行为。

   现在我们要根据不同需求，计算两个数的四则运算( + - * /)

   ```
   //策略定义算法的接口
   public interface Strategy {
       int calculate(int num1,int num2);
   }
   //具体算法,加法
   public class OperationAdd implements Strategy {
       @Override
       public int calculate(int num1, int num2) {
           return num1 + num2;
       }
   }
   //具体算法,减法
   public class OperationSubstract implements Strategy {
       @Override
       public int calculate(int num1, int num2) {
           return num1 - num2;
       }
   }
   //具体算法,乘法
   public class OperationMultiply implements Strategy {
       @Override
       public int calculate(int num1, int num2) {
           return num1 * num2;
       }
   }
   //具体算法,除法
   public class OperationDivide implements Strategy {
       @Override
       public int calculate (int num1, int num2){
           int res = 0;
           try {
               res = num1 / num2;
           }catch (Exception e) {
               e.printStackTrace();
           }
           return res;
       }
   }
   //上下文
   public class Context {
       //持有一个具体策略对象
       private Strategy strategy;
    
       //传入一个具体策略对象
       public Context(Strategy strategy) {
           this.strategy =strategy;
       }
    
       public int calculate(int num1,int num2){
           //调用具体策略对象进行算法运算
           return strategy.calculate(num1,num2);
       }
   }
    public static void main(String[] args) {
           //计算 1 + 1
           Context context = new Context(new OperationAdd());
           System.out.println("1 + 1 = " + context.calculate(1,1));
           //计算 1 - 1
           context = new Context(new OperationSubstract());
           System.out.println("1 - 1 = " +context.calculate(1,1));
       }
   ```

    

   ### 策略模式优点：

   1.算法可以自由切换。

   2.避免使用多重条件判断。

   3.扩展性良好。

   ### 策略模式缺点：

   1.策略类会增多。

   2.所有策略类都需要对外暴露。

   ### 适用场景：

   1.如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。

   2.一个系统需要动态地在几种算法中选择一种。

   3.一个类定义了多种行为, 并且这些行为在这个类的操作中以多个条件语句的形式出现。将相关的条件分支移入它们各自的Strategy类中以代替这些条件语句。

    

   转载自:https://www.cnblogs.com/geek6/p/3951677.html  

   转载自:https://blog.csdn.net/heijunwei/article/details/82056313
