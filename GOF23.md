#GOF23设计模式



#创建型模式：
##单例模式汇总
单例模式是为了保证一个类只有一个实例，并提供一个访问该实例的全局访问点。
由于只有一个实例，减少了系统性能开销，当一个对象的产生需要较多资源，如读取配置，产生其他依赖对象时，则可以通过在应用启动时直接产生一个单例对象，然后永久驻留内存。

###饿汉式
    public class singleDemo{
	pravite static instance=new singleDemo();
	pravite singleDemo（）{}；
	public static SingleDemo getInstance(){
	return instance;
	}
	}
总结：线程安全，加载速度快，但不能延迟加载

###懒汉式
	public class singleDemo{
	pravite static instance;
	pravite singleDemo(){};
	public static synchronized SingleDemo getInstance(){
	if(instance=null){
	instance=new singleDem();
	}
	return instance;
	}
	}
总结：如果加上synchroized则线程安全，加载速度慢，可延迟加载，如果不加上则线程不安全，加载速度较快，可延迟加载

###双重锁式
	public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
	}  
volatile 变量.用来确保将变量的更新操作通知到其他线程,保证了新值能立即同步到主内存,以及每次使用前立即从主内存刷新. 当把变量声明为volatile类型后,编译器与运行时都会注意到这个变量是共享的.

该方法理论上:线程安全，加载速度快，可延迟加载
但由于JVM虚拟机的原因 这种方式有时候会出问题，不建议使用

###静态内部类方法
	public class singleton{
	private singleton(){};
	pravite static class singletonHolder{
	pravite static final singleton instance=new singleton();
	}
	public singleton getInstance(){
	return singleton.instance;
	}
	}

总结：由于在调用getInstance（）时类加载器才会加载静态内部类，类加载是天然的线程安全，故该方法 线程安全，加载速度快，可延迟加载

###枚举式
	public enum singleton{
	INSTANCE;
	public void singletonOperation(){
	//这里做功能处理
	}
	}
总结：枚举本身就是单例，还能避免通过反射和反序列化的漏洞产生多个对象，唯一的缺点是不能延时加载。

###防止反射漏洞
私有化构造器时里面添加：

	private singleTon(){
	if(instance！=null){
	throw new RuntimeException();
	}
	}
使通过反射调用的构造器时直接抛出异常

###防止反序列化
添加一个方法

	private Objiect readResolve()throw ObjectStreamException{ 
	return instance;
	}
反序列化时，直接返回instance而不是反序列化时新建的对象

##工厂模式
实现了创建者和调用者的分离

核心本质：
实例化对象，用工厂方法代替new操作。
将选择实现类，创建对象统一管理和控制。从而将调用者跟我们的实现类解耦。
如：  

如果有许多地方都需要生成A的对象，那么你需要写很多A  a=new A（）。

如果需要修改的话，你要修改许多地方。
但是如果用工厂模式，你只需要修改工厂代码。其他地方引用工厂，可以做到只修改一个地方，其他代码都不动，就是解耦了。

###简单工厂模式
模拟一个汽车工厂
建立工厂（创建者）：
第一种写法
	
	public class CarFactory{
	public Car createCar(String type){
	switch(type){
		case "奥迪":
			return new  Audi();
			break;
		case "比亚迪":
			return new Byd();
			break;
		defualt:
			return null;
			break;
	}
	}
	}
第二种
	
	public Car createCar(String type){
	public static Car createAudi(){
		return new Audi();
	}
	public static Car createByd(){
		return new Byd();
	}
	}
建立司机（调用者）：
	
	public class Driver{
	CarFactory carFactory=new CarFactory();
	carFactory.createCar("奥迪").run();
	}

总结：不方便拓展，拓展新车需要修改代码

###工厂方法模式

需要建立一个工厂接口

	public interface CarFactory{
		Car createCar();
	}

当需要造一辆奥迪车时，则新建一个工厂
	
	public class AudiFactory implements CarFactory{
		@Override
		public car createCar(){
			return new Audi();
		}
	}

需要造BYD，就建一个BYD工厂

	public class BydFactory implements CarFactory{
		@Override
		public car createCar(){
			return new Byd();
		}
	}

总结：相对简单工厂更易扩展，不需要改已有的代码 ，但是类结构复杂

###抽象工厂模式

还是以建造汽车为例：

首先是汽车引擎：分为好引擎，差引擎

	public interface Engine {
		void run();
		void start();
	}
	
	class goodEngine implements Engine{
	
		@Override
		public void run() {
			System.out.println("转的快");
			
		}
	
		@Override
		public void start() {
			System.out.println("启动快，可以自动启停");
			
		}}
	
	class lowEngine implements Engine{
	
		@Override
		public void run() {
			System.out.println("转的慢");
			
		}
	
		@Override
		public void start() {
			System.out.println("启动慢");
			
		}}

然后是汽车座椅：同样分为好坏

		public interface Seat {
		void massage();
	}
	
	class goodSeat implements Seat{
		@Override
		public void massage() {
			System.out.println("可以按摩");
			
		}
		
	}
	
	class lowSeat implements Seat{
		@Override
		public void massage() {
			System.out.println("不可以按摩");
			
		}
		
	}

汽车轮胎：

	public interface Tyre {
		void revolve();
	}
	
	class goodTyre implements Tyre{
	
		@Override
		public void revolve() {
			System.out.println("磨损小");
		}
	}
	
	class lowTyre implements Tyre{
	
		@Override
		public void revolve() {
			System.out.println("磨损大");	
		}	
	}

汽车工厂接口：

	public interface CarFactory {
		Engine createEngine();
		Seat createSeat();
		Tyre createTyre();
	}

好车的工厂：

	public class goodCarFactory implements CarFactory{

	@Override
	public Engine createEngine() {
		return new goodEngine();
	}

	@Override
	public Seat createSeat() {
		return new goodSeat();
	}

	@Override
	public Tyre createTyre() {
		return new goodTyre();
	}

差车的工厂：

	public class lowCarFactory implements CarFactory{

	@Override
	public Engine createEngine() {
		return new lowEngine();
	}

	@Override
	public Seat createSeat() {
		return new lowSeat();
	}

	@Override
	public Tyre createTyre() {
		return new lowTyre();
	}
	}

总结：不可以增加产品，但 可以增加产品族：如高级车，中级车，低级车，中低级车（可以自由组合）

##建造者模式

分离了对象子组件的单独构造（由Builder来负责）和装配（由Director负责）。从而可以构造出复杂的对象。这个模式适用于：某个对象的构建过程复杂的情况下使用。

由于实现了构建和装配的解耦。不同的构建器，相同的装配，也可以做出不同的对象；相同的构造器，不同的装配顺序也可以做出不同的对象。也就是实现了构建算法、装配算法的解耦，实现了更好的复用。

以建造一辆飞船为例：

首先是飞船的一个modle类：
	
	public class AirShip {
	private orbitalModule orbitalModule;
	private Engine engine;
	private EscapeTower escapeTower;
	public orbitalModule getOrbitalModule() {
		return orbitalModule;
	}
	public void setOrbitalModule(orbitalModule orbitalModule) {
		this.orbitalModule = orbitalModule;
	}
	public Engine getEngine() {
		return engine;
	}
	public void setEngine(Engine engine) {
		this.engine = engine;
	}
	public EscapeTower getEscapeTower() {
		return escapeTower;
	}
	public void setEscapeTower(EscapeTower escapeTower) {
		this.escapeTower = escapeTower;
	}
	}
	
飞船组件的modle类：
	
	轨道舱：
	class orbitalModule{
	private String name;

	public orbitalModule(String string) {
		this.name=string;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
	
	}
	
	引擎：
	class Engine{
	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Engine(String name) {
		super();
		this.name = name;
	}
	
	}
	
	逃逸塔：
	class EscapeTower{
	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public EscapeTower(String name) {
		super();
		this.name = name;
	}
	}	

然后生产各个组件的Builder接口：

	public interface AirShipBuilder {
		Engine buildEngine();
		orbitalModule buildOrbitalModule();
		EscapeTower buildEscapeTower();
	}  

继承该接口的实体类（负责生产部件）：

	public class MyAirShipBuilder implements AirShipBuilder{
	
		@Override
		public Engine buildEngine() {
			System.out.println("构建发动机");
			return new Engine("发动机！");//  可以用工厂模式生产
		}
	
		@Override
		public orbitalModule buildOrbitalModule() {
			System.out.println("构建轨道舱");
			return new orbitalModule("轨道舱");
		}
	
		@Override
		public EscapeTower buildEscapeTower() {
			System.out.println("逃逸塔");
			return new EscapeTower("逃逸塔");
		}
		
	}

组装各组件的公共接口：

	public interface AirshipDirector {
		AirShip directAirShip();
	}

组装类的实现类：
	
	public class MyAirShipDirector implements AirshipDirector{
		
		private AirShipBuilder builder;
		
		public MyAirShipDirector(AirShipBuilder builder) {
			this.builder=builder;
		}
	
		@Override
		public AirShip directAirShip() {
			Engine e=builder.buildEngine();
			EscapeTower et=builder.buildEscapeTower();
			orbitalModule o=builder.buildOrbitalModule();
			
			AirShip ship=new AirShip();
			ship.setEngine(e);
			ship.setEscapeTower(et);
			ship.setOrbitalModule(o);
			return ship;
		}
	}

在main方法中生产一个我的飞船：

	AirShipDirector director =new MyAirShipDirector(new MyAirShipBuilder());

	AirShip ship=director.directAirShip();

##原型模式（克隆/拷贝模式） prototype

通过new产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。

优势：效率高

类似于new，但不同于new。new创建出来的对象属性采用的是默认值，克隆出的对象的属性值和原型对象完全相同。然后再修改克隆对象的值。

实现：Cloneable接口和clone方法  
   Prototype模式中最难实现的是内存复制操作，所幸java中提供了clone（）方法替我们做了绝大部分事情。

代码实现很简单：
	
	public class Sheep implements Cloneable{
		private String name;
		private String birthday;
		
		public Sheep(String name, String birthday) {
			super();
			this.name = name;
			this.birthday = birthday;
		}
	
		public String getName() {
			return name;
		}
	
		public void setName(String name) {
			this.name = name;
		}
	
		public String getBirthday() {
			return birthday;
		}
	
		public void setBirthday(String birthday) {
			this.birthday = birthday;
		}
		//重写克隆方法,注意：Cloneable是个空接口，没有方法，这里的clone方法时Objiect的方法
		@Override
		protected Object clone() throws CloneNotSupportedException {
			Object obj=super.clone();	//直接调用object对象的clone方法
			return obj;
		}
		}

调用的时候：
		
	Sheep S1=new Sheep("多利"，new Date（12312321331L））；
	Sheep S2=S1.clone();

这两个对象是两个不同的对象，指向的不同内存空间，但内容的值相同。但注意上面这种写法：指向的是同一个date对象，这种称为浅克隆。

深克隆：把date对象也克隆出一个，使两个Sheep对象完全分离：   
修改clone 方法：

	@Override
		protected Object clone() throws CloneNotSupportedException {
			Object obj=super.clone();
			Sheep s=（Sheep）obj；
			s.birthday=(Date)this.birthday.clone();//把属性也进行克隆
			return obj;
		}

#结构型模式

##适配器模式adapter

将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以在一起工作。

模式中的角色：  
目标接口(Target)  
需要适配的类(Adaptee)  
适配器（Adapter）

这里以一个ps/2接口的键盘通过转接器插上电脑，电脑调用键盘打字为例子：

首先是需要适配的类（键盘）：

	/**
	 * 被适配的类
	 * 比如一个ps/2接口的键盘
	 * @author zyj010
	 *
	 */
	public class Adaptee {
		
		public void request(){
			System.out.println("可以完成客户需要的功能！比如键盘是打字");
		}
	}


目标接口（USB接口）：

	/**
	 * 目标接口
	 * 如USB接口
	 * @author zyj010
	 *
	 */
	public interface Target {
		void handleReq();
	}

适配器（ps/2转USB转接器）
	
	/**
	 * 适配器
	 * 像PS/2转USB接口的转接器
	 * @author zyj010
	 *
	 */
	public class Adapter  implements Target{
		Adaptee adptee;

		public Adapter(Adaptee adptee) {
			super();
			this.adptee = adptee;
		}
		@Override
		public void handleReq() {
			adptee.request();
		}
	
	}

客户端（计算器）：
	
	/**
	 * 客户端类
	 * 比如一个只有USB接口的计算机
	 * @author zyj010
	 *
	 */
	public class Client {  
		public  void test1(Target t){
			t.handleReq();
		}
		
		public static void main(String[] args) {
			Adaptee a=new Adaptee();
			Target adapter=new Adapter(a);
			Client c=new Client();
			c.test1(adapter);
		}
	}

常见场景：InputStreamReader(InputStream);OutputStreamWriter(OutputStream);经常用来做旧系统改造和升级。

##代理模式（Proxy pattern）

通过代理控制对对象的访问！  

可以详细控制访问某个（某类）对象的方法，在调用这个方法前做前置处理，调用这个方法后做后置处理。

**代理模式的角色：**   
-抽象角色：定义代理角色和真实角色的公共对外方法  
-真实角色：实现抽象角色，定义真实角色所要实现的业务逻辑，供代理角色调用，只关注真正的业务逻辑   
-代理角色：实现抽象角色，是真实角色的代理，通过真实角色的业务逻辑方法来实现抽象方法，并可以附加自己的操作，将统一的流程控制放到代理角色中去处理。 

**应用场景：**
安全代理：屏蔽对真实角色的直接访问。   
远程代理：通过代理类处理远程方法调用（RMI)
延迟加载：先加载轻量级的代理对象，真正需要再加载真实对象。

**分类：**
静态代理 （静态定义的代理类）  
动态代理	（动态生成的代理类）
.JDK自带的动态代理
.javaassist字节码操作库实现
.CGLIB
.ASM（底层使用指令，可维护性较差）

以周杰伦和经纪人的例子为例：

###静态代理

首先是抽象明星接口：

	public interface Star {
		void confer();//面谈
		void signContract();//签合同
		void bookTicket();//订票
		void sing();//唱歌
		void collectMoney();//收钱
	}	

实现抽象明显的真实明星类（周杰伦）：

	public class RealStar implements Star{
	
		@Override
		public void confer() {
			System.out.println("Realstar：面谈");
			
		}
	
		@Override
		public void signContract() {
			System.out.println("Realstar：签合同");
			
		}
	
		@Override
		public void bookTicket() {
			System.out.println("Realstar：订票");
			
		}
	
		@Override
		public void sing() {
			System.out.println("周杰伦本人：唱歌");
			
		}
	
		@Override
		public void collectMoney() {
			System.out.println("Realstar：收钱");
			
		}
	
	}

实现Star类的代理类（经纪人）：

	public class proxyStar implements Star{
		private Star star;
		
		public proxyStar(Star star) {
			super();
			this.star = star;
		}
	
		@Override
		public void confer() {
			System.out.println("proxyStar:面谈");
			
		}
	
		@Override
		public void signContract() {
			System.out.println("proxyStar:签合同");
			
		}
	
		@Override
		public void bookTicket() {
			System.out.println("proxyStar:买票");
			
		}
	
		@Override
		public void sing() {
			star.sing();
			
		}
	
		@Override
		public void collectMoney() {
			System.out.println("proxyStar:收钱");
			
		}
	}

外部调用（客户邀请周杰伦唱歌，只需要联系经纪人）：

		public static void main(String[] args) {
			Star real=new RealStar();
			Star proxy=new proxyStar(real);
			
			proxy.confer();
			proxy.signContract();
			proxy.bookTicket();
			proxy.sing();
			proxy.collectMoney();
		}

###动态代理

相比静态代理的优点：抽象角色中（接口）声明的所有方法都被转移到调用处理器，一个集中的方法中处理，这样，我们可以更加灵活和统一的处理众多的方法。

1.JDK自带的动态代理   
-java.lang.reflect.Proxy:动态生成代理类和对象   
-java.lang.reflect.InvocationHandler(处理器接口):可以通过invoke方法实现对真实角色的代理访问，每次通过proxy生成代理类对象时都要指定对应的处理器对象

和之前所写的静态方法的唯一不同是，取消代理类，加下面这个类

	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	
	public class StarHandler implements InvocationHandler{
		Star RealStar;
		
		
		public StarHandler(Star realStar) {
			super();
			RealStar = realStar;
		}
	
	
		@Override
		public Object invoke(Object proxy, Method method, Object[] args)
				throws Throwable {
			
			//在这里执行调用真正的对象前的前置方法
			
			if(method.getName().equals("sing")){
			method.invoke(RealStar, args);
			}
			
			//这里执行后置方法
			return null;
		}
		public static void main(String[] args) {
			Star realStar=new RealStar();
			StarHandler starHandler=new StarHandler(realStar);
			
			Star proxy=(Star) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{Star.class}, starHandler);

			proxy.sing();
		}
	}


##桥接模式（bridge）

处理多继承接口，处理多维度变化的场景，将各个维度设计成独立的继承结构，使各个维度可以独立的扩展，在抽象层建立关联

以电脑销售为例，电脑有两个维度：一个是电脑类型，有平板，笔记本，台式；另一个是品牌维度：有联想，神舟，戴尔等。  将这两个维度分别开来，通过桥接模式联系在一起，在新增电脑类型或者品牌的时候不用去改动另一个维度

首先是电脑的公共抽象类：

	public abstract class Computer {
		protected Brand brand;
	
		public Computer(Brand brand) {
			super();
			this.brand = brand;
		}
		
		public void sale(){
			brand.sale();
		}
	}

抽象类的实现类：

	class Desktop extends Computer{
		public Desktop(Brand brand) {
			super(brand);
		}
		@Override
		public void sale() {
			super.sale();
			System.out.println("销售台式");
		}
		
	}
	
	
	class Laptop extends Computer{
		public Laptop(Brand brand) {
			super(brand);
		}
		@Override
		public void sale() {
			super.sale();
			System.out.println("销售笔记本");
		}
	}
	
	class Pad extends Computer{
		public Pad(Brand brand) {
			super(brand);
		}
		@Override
		public void sale() {
			super.sale();
			System.out.println("销售平板");
		}
	}	

品牌的公共接口

	public interface Brand {
		void sale();
	}

品牌接口的实现类

	class Lenovo implements Brand{
	
		@Override
		public void sale() {
			System.out.println("销售联想电脑");
			
		}
		
	}
	
	class Dell implements Brand{
	
		@Override
		public void sale() {
			System.out.println("销售戴尔电脑");
			
		}
		
	}
	class Hasee implements Brand{
	
		@Override
		public void sale() {
			System.out.println("销售神舟电脑");
			
		}
		
	}


在调用的时候：
	Computer c=new Laptop(new Lenovo());
	c.sale();

这样就实现了品牌和电脑的分离，各自扩展互不影响。避免多继承。


##组合模式（composite）
把部分和整体的关系用**树形结构**来表示，从而使客户端可以使用统一的方式处理部分对象和整体对象。

核心：  
抽象构件角色：定义了叶子和容器构件的共同点 
叶子（Leaf）构件角色：无子节点  
容器（Composite）构件角色：有容器特征，可以包含子节点   
 
他们之间的一般关系：            
	
	/**
	 * 抽象组件
	 * @author zyj010
	 *
	 */
	public interface Component {
		void operation();
	}
	
	//叶子组件
	interface Leaf extends Component{	
	}
	
	//容器组件
	interface Composite extends Component{
		void add(Component c);
		void remove(Component c);
		Component getChild(int index);
	}

以一个杀毒软件为例：

抽象组件（文件）：
	
	public interface AbstractFile {
		void killVirus();//杀毒
	}

叶子组件（单个文件 ）：

	class ImageFile implements AbstractFile{
		String name;
		
		public ImageFile(String name) {
			super();
			this.name = name;
		}
	
		@Override
		public void killVirus() {
			System.out.println("---图片文件："+name+",进行查杀！");
		}
		
	}
	class TextFile implements AbstractFile{
		String name;
		
		public TextFile(String name) {
			super();
			this.name = name;
		}
	
		@Override
		public void killVirus() {
			System.out.println("---文本文件："+name+",进行查杀！");
		}
		
	}
	class VideoFile implements AbstractFile{
		String name;
		
		public VideoFile(String name) {
			super();
			this.name = name;
		}
	
		@Override
		public void killVirus() {
			System.out.println("---视频文件："+name+",进行查杀！");
		}
		
	}

容器组件（文件夹）：

	class Folder implements AbstractFile{
		private String name;
		private ArrayList<AbstractFile> list=new ArrayList<AbstractFile>();
	
		public void add(AbstractFile file) {
			list.add(file);
		}
		
		public void remove(AbstractFile file){
			list.remove(file);
		}
		
		public AbstractFile getChild(int index){
			return list.get(index);
		}
		@Override
		public void killVirus() {
			System.out.println("---文件夹"+name+",进行查杀");
			
			for(AbstractFile file:list){
				file.killVirus();
			}
		}	
	}

容器组件中可以添加容器组件或者叶子组件，形成树形结构，类似于文件系统，当进行操作时就会一层一层调用对应的operation方法进行操作。

##装饰模式（decorator）

动态的为一个对象增加新的功能                                                 
		
装饰模式是一种用于代替继承的技术，无须通过继承增加子类就能扩展对象的新功能。使用对象的关联关系替代继承关系，更加灵活，同时避免类型体系的快速膨胀。

用装饰模式给普通的car加上飞行和入水的功能：

Component抽象构件角色（车的公共接口）：
	
	/**
	 * 抽象构建
	 * @author zyj010
	 *
	 */
	public interface ICar {
		void move();
	}   

ConcreteComponent 具体构件角儿（真实对象--也就是一辆普通car）：

	class Car implements ICar{
	
		@Override
		public void move() {
			System.out.println("在陆地上跑");
		}
		
	}

Decorator 装饰角色：

	class superCar implements ICar{
		
		private ICar car;
		
	
		public superCar(ICar car) {
			super();
			this.car = car;
		}
	
		
	
		@Override
		public void move() {
			car.move();
		}
		
	}


ConcreteDecorator具体装饰角色：

	class FlyCar extends superCar{
	
		public FlyCar(ICar car) {
			super(car);
		}
		public void fly(){
			System.out.println("天上飞");
		}
		
		@Override
		public void move() {
			// TODO Auto-generated method stub
			super.move();
			fly();
		}
	}

	class WaterCar extends superCar{

		public WaterCar(ICar car) {
			super(car);
		}
		public void swim(){
			System.out.println("水里游");
		}
		
		@Override
		public void move() {
			// TODO Auto-generated method stub
			super.move();
			swim();
		}
	}

给车加上飞行和入水的功能：

	
	SuperCar car=new WaterCar(new Flycar(new Car()));

总结：   
装饰模式（Decorator）也叫包装器模式（Wrapper）   
装饰模式降低系统的耦合度，可以动态的增加或删除对象的职责，并使得需要装饰的具体构建类和具体装饰类可以独立变化，以便增加新的具体构建类和具体装饰类。   

常见场景：IO流的实现就是使用了装饰模式。    

优点：   
扩展对象功能，比继承灵活，不会导致类个数的急剧增加   
可以对一个对象进行多次装饰，创造出不同行为的组合，得到功能更加强大的对象  
具体构建类和装饰类可以独立变化，用户可以根据需要自己增加新的具体构件子类和具体装饰子类。   

缺点：  
产生很多小对象。大量小对象占据内存，对性能有一定影响。   
装饰模式容易出错，调试排查比较麻烦。

**装饰模式和桥接模式的区别：**

两个模式都是为了解决过多子类对象问题。但他们诱因不一样。桥接模式是对象自身现有机制沿着多维度变化，是既有部分不稳定。而装饰模式主要是为了增加新的功能

##外观模式  
为子系统提供统一的入口，封装子系统的复杂性，便于客户端调用。

这个模式平时编程潜移默化都会用到，就不提供例子了。其实就是一种封装。 

##享元模式 （FlyWeight） 
使用场景：		
内存属于稀缺资源，不能随便浪费。如果有很多个完全相同或相似的对象，可以通过享元模式，节省内存。

核心：   
享元模式以共享的方式高效地支持大量细粒度对象的重用。   
享元对象能做到共享的关键是区分了内部状态和外部状态。   
内部状态：可以共享，不会随环境变化而改变。  
外部状态：不能共享，会随环境变化而改变

享元模式实现：  
FlyweightFactory享元工厂类    
-创建管理享元对象，享元池一般设计成键值对  
Flyweight抽象享元类   
-通常是一个接口或抽象类，声明公共方法，这些方法可以向外界提供对象的内部状态，设置外部状态。   
ConcreteFlyWeight 具体享元类   
-为内部状态提供成员变量尽心存储  
UnsharedConcreteFlyWeight 非共享享元类   
-不能被共享的子类可以设计为非共享享元类

以围棋为例:围棋的棋子颜色属性是共享的，位置属性是特有的

非共享享元类（位置）：

	public class Coordinate {
		private int x,y;
	
		public Coordinate(int x, int y) {
			super();
			this.x = x;
			this.y = y;
		}
	
		public int getX() {
			return x;
		}
	
		public void setX(int x) {
			this.x = x;
		}
	
		public int getY() {
			return y;
		}
	
		public void setY(int y) {
			this.y = y;
		}  
		
	}

抽象享元类：

	public interface ChessFlyWeight {
		void setColor(String c);
		String getColor();
		void display(Coordinate c);
	}

实现抽象享元类的具体享元类（棋子）

	class ConcreteChess implements ChessFlyWeight{
		
		private String color;
		
		public ConcreteChess(String color) {
			super();
			this.color = color;
		}
	
		@Override
		public void setColor(String c) {
			this.color=c;	
		}
	
		@Override
		public String getColor() {
			return this.color;
		}
	
		@Override
		public void display(Coordinate c) {
			System.out.println("棋子颜色"+color);
			System.out.println("棋子位置"+c.getX()+"---"+c.getY());		
		}	
	}


享元工厂类：

	/**
	 * 享元工厂类
	 * @author zyj010
	 *
	 */
	public class ChessFlyWeightFactory {
		//享元池
		private static Map<String, ChessFlyWeight> map=new HashMap<String, ChessFlyWeight>();
		public static ChessFlyWeight getChess(String color){
			
			if(map.get(color)!=null){
				return map.get(color);
			}
			else {
				ChessFlyWeight cfw=new ConcreteChess(color);
				map.put(color, cfw);
				return cfw;
			}
		}
	}

调用：

	public static void main(String[] args) {
		ChessFlyWeight chess1=ChessFlyWeightFactory.getChess("黑色");
		ChessFlyWeight chess2=ChessFlyWeightFactory.getChess("黑色");
		System.out.println(chess1);
		System.out.println(chess2);
		
		System.out.println("增加外部状态的处理====================");
		chess1.display(new Coordinate(10, 10));
		chess2.display(new Coordinate(20, 20));
	}

运行结果：

	flyweight.ConcreteChess@15db9742
	flyweight.ConcreteChess@15db9742
	增加外部状态的处理====================
	棋子颜色黑色
	棋子位置10---10
	棋子颜色黑色
	棋子位置20---20

根据结果可以发现，chess1，chess2实际上是一个对象，通过增加外部状态处理将他们区分开来。相当于棋子只分为两类，一类是白色棋子，一类是黑色棋子，只有这两类用来共享，其他的只区分位置信息。

享元模式开发中应用的场景：    
根据其共享的特性，可以在任何"池"中操作，比如：线程池，数据库连接池

优点：   
极大减少内存中对象的数量             
相同或相似对象内存中只存一份，极大地节约资源，提高系统性能    
外部状态相对独立，不影响内部状态    

缺点：   
模式较复杂，使程序逻辑复杂化   
为了节省内存，共享了内部状态，分离出外部状态，而读取外部状态使运行时间变长。用时间换取了空间。


##中介模式（Mediator）
如果一个系统中对象之间的联系呈现为网状结构，对象之间存在大量多对多的关系，将导致关系及其复杂，这些对象称为"同事对象"  

我们可以引入一个中介者对象，使各个同事对象间只跟中介者对象打交道，将复杂的网状结构化解为星形结构。

以各部门之间的协调为例：   

中介类公共接口：   

	public interface Mediator {
		
		void register(String dname,Department d);
		
		void command(String dname);
		
	}

同事类的接口：

	//同事类的接口
	public interface Department {
		void selfAction(); //做本部门的事情
		void outAction();  //向总经理发出申请
	}

实现接口的中介类（总经理）：   
	
	public class President implements Mediator {
		
		private Map<String,Department> map = new HashMap<String , Department>();
		
		@Override
		public void command(String dname) {
			map.get(dname).selfAction();
		}
	
		@Override
		public void register(String dname, Department d) {
			map.put(dname, d);
		}
	
	}

实现同事类（各部门）：

开发部：                  

	public class Development implements Department {
	
		private Mediator m;  //持有中介者(总经理)的引用
		
		public Development(Mediator m) {
			super();
			this.m = m;
			m.register("development", this);
		}
	
		@Override
		public void outAction() {
			System.out.println("汇报工作！没钱了，需要资金支持！");
		}
	
		@Override
		public void selfAction() {
			System.out.println("专心科研，开发项目！");
		}
	
	}  

财务部：
	
	public class Finacial implements Department {
	
		private Mediator m;  //持有中介者(总经理)的引用
		
		public Finacial(Mediator m) {
			super();
			this.m = m;
			m.register("finacial", this);
		}
	
		@Override
		public void outAction() {
			System.out.println("汇报工作！没钱了，钱太多了！怎么花?");
		}
	
		@Override
		public void selfAction() {
			System.out.println("数钱！");
		}
	
	}

市场部：

	public class Market implements Department {
	
		private Mediator m;  //持有中介者(总经理)的引用
		
		public Market(Mediator m) {
			super();
			this.m = m;
			m.register("market", this);
		}
	
		@Override
		public void outAction() {
			System.out.println("汇报工作！项目承接的进度，需要资金支持！");
			
			m.command("finacial");
			
		}
	
		@Override
		public void selfAction() {
			System.out.println("跑去接项目！");
		}
	
	}

调用：

	public class Client {
		public static void main(String[] args) {
			Mediator m = new President();
			
			Market   market = new Market(m);
			Development devp = new Development(m);
			Finacial f = new Finacial(m);
			
			market.selfAction();
			market.outAction();
			
		}
	}


常见场景：MVC模式，其中C，控制器就是一个中介者对象。M,V都和他打交道

##命令模式（command）
将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。也称作：动作Action模式，事务transaction模式。

结构：   
Command 抽象命令类  
ConcreteCommand 具体命令类  
Invoker调用者/请求者   
Receiver接收者   
Client  客户类

命令接口及具体命令：

	public interface Command {
		/**
		 * 这个方法是一个返回结果为空的方法。
		 * 实际项目中，可以根据需求设计多个不同的方法
		 */
		void execute();
	}
	
	
	class ConcreteCommand implements Command {
		
		private Receiver receiver;	//命令的真正的执行者
		
		public ConcreteCommand(Receiver receiver) {
			super();
			this.receiver = receiver;
		}
	
		@Override
		public void execute() {
			//命令真正执行前或后，执行相关的处理！
			receiver.action();
		}
		
	}

命令的真正执行者：

	/**
	 * 真正的命令的执行者
	 * @author Administrator
	 *
	 */
	public class Receiver {
		public void action(){
			System.out.println("Receiver.action()");
		}
	}

调用者：

		//调用者/发起者
	public class Invoke {
		
		private Command command;   //也可以通过容器List<Command>容纳很多命令对象，进行批处理。数据库底层的事务管理就是类似的结构！
		
			public Invoke(Command command) {
				super();
				this.command = command;
			} 
			
			//业务方法 ，用于调用命令类的方法
			public void call(){
				command.execute();
			}
			
			
		}

调用：

	public class Client {
		public static void main(String[] args) {
			Command c = new ConcreteCommand(new Receiver());
			Invoke i = new Invoke(c);
			i.call();
		}
	}


常见场景：数据库事务机制的底层实现


##策略模式（strategy）  

举一个例子，关于商品打折

普通的写法：

	/**
	 * 实现起来比较容易，符合一般开发人员的思路
	 * 假如，类型特别多，算法比较复杂时，整个条件语句的代码就变得很长，难于维护。
	 * 如果有新增类型，就需要频繁的修改此处的代码！
	 * 不符合开闭原则！
	 * @author Administrator
	 *
	 */
	public class TestStrategy {
		public double getPrice(String type, double price) {
	
			if (type.equals("普通客户小批量")) {
				System.out.println("不打折,原价");
				return price;
			} else if (type.equals("普通客户大批量")) {
				System.out.println("打九折");
				return price * 0.9;
			} else if (type.equals("老客户小批量")) {
				System.out.println("打八五折");
				return price * 0.85;
			} else if (type.equals("老客户大批量")) {
				System.out.println("打八折");
				return price * 0.8;
			}
			return price;
		}
	
	}


按策略模式：

策略模式对应于解决某一个问题的一个算法族，允许用户从该算法族中任选一个算法解决某一问题，同时可以方便的更换算法或者增加新的算法。并且由客户端决定调用哪个算法。

策略的统一接口：

	public interface Strategy {
		public double getPrice(double  standardPrice);
	}


策略的各种实现类：  
老用户少量：

	public class OldCustomerFewStrategy implements Strategy {
	
		@Override
		public double getPrice(double standardPrice) {
			System.out.println("打八五折");
			return standardPrice*0.85;
		}
	
	}

老用户大量：

	public class OldCustomerManyStrategy implements Strategy {
	
		@Override
		public double getPrice(double standardPrice) {
			System.out.println("打八折");
			return standardPrice*0.8;
		}
	
	}

新顾客少量：

	public class NewCustomerFewStrategy implements Strategy {
	
		@Override
		public double getPrice(double standardPrice) {
			System.out.println("不打折，原价");
			return standardPrice;
		}
	
	}

新顾客大量：

	public class NewCustomerManyStrategy implements Strategy {
	
		@Override
		public double getPrice(double standardPrice) {
			System.out.println("打九折");
			return standardPrice*0.9;
		}
	
	}

以上形成了算法族

上下文管理类：

	/**
	 * 负责和具体的策略类交互
	 * 这样的话，具体的算法和直接的客户端调用分离了，使得算法可以独立于客户端独立的变化。
	 * 如果使用spring的依赖注入功能，还可以通过配置文件，动态的注入不同策略对象，动态的切换不同的算法.
	 * @author Administrator
	 *
	 */
	public class Context {
		private Strategy strategy;	//当前采用的算法对象
	
		//可以通过构造器来注入
		public Context(Strategy strategy) {
			super();
			this.strategy = strategy;
		}
		//可以通过set方法来注入
		public void setStrategy(Strategy strategy) {
			this.strategy = strategy;
		}
		
		public void pringPrice(double s){
			System.out.println("您该报价："+strategy.getPrice(s));
		}
		}

调用：
		
	Strategy s1 = new OldCustomerManyStrategy();
	Context ctx = new Context(s1);
		
	ctx.pringPrice(998);

##模板方法模式（template method）  
模板方法模式是编程中经常用到的模式。它定义了一个操作中的算法骨架，将某些步骤延迟到子类实现。这样，新的子类可以在不改变一个算法结构的前提下重新定义该算法的某些特定步骤。

**核心：**  
处理某个流程的代码已经都具备，但是其中某个节点的代码暂时不能确定。因此，我们采用工厂方法模式，将这个节点的代码实现转移给子类完成。	
即：处理步骤，父类中定义，具体实现延迟到子类中定义

以在银行办理业务为例：

模板类：

	public abstract class BankTemplateMethod {
		//具体方法
		public void takeNumber(){
			System.out.println("取号排队");
		}
		
		public abstract void transact(); //办理具体的业务	//钩子方法
		
		public void evaluate(){
			System.out.println("反馈评分");
		}
		
	
	
		public final void process(){	//模板方法！！！
			this.takeNumber();
	
			this.transact();
	
			this.evaluate();
		}
		
	}

模板使用：

	public class Client {
		public static void main(String[] args) {
			BankTemplateMethod btm = new DrawMoney();
			btm.process();
			
			//采用匿名内部类
			BankTemplateMethod btm2 = new BankTemplateMethod() {
				
				@Override
				public void transact() {
					System.out.println("我要存钱！");
				}
			};
			btm2.process();
			
			BankTemplateMethod btm3 = new BankTemplateMethod() {
				@Override
				public void transact() {
					System.out.println("我要理财！我这里有2000万韩币");
				}
			};
			
			btm3.process();
			
		}
	}
	class DrawMoney extends BankTemplateMethod {
	
		@Override
		public void transact() {
			System.out.println("我要取款！！！");
		}
		
	}


常用于整体步骤固定，某些部分易变。易变部分抽象出来，供子类实现。

##状态模式（state） 
用于解决系统中复杂对象的状态转换以及不同状态下行为的封装问题  

结构：  
Context环境类：环境类中维护一个state对象，他是定义了当前的状态。  
State抽象状态类   
ConcreteState具体状态类：每一个类封装了一个状态对应的行为。

以酒店房间入住流程来说：

状态统一接口：

	public interface State {
		void handle();
	}

具体状态类：

	/**
	 * 空闲状态
	 * @author Administrator
	 *
	 */
	public class FreeState implements State {
	
		@Override
		public void handle() {
			System.out.println("房间空闲！！！没人住！");
		}
	
	}


	/**
	 * 已入住状态
	 * @author Administrator
	 *
	 */
	public class CheckedInState implements State {
	
		@Override
		public void handle() {
			System.out.println("房间已入住！请勿打扰！");
		}
	
	}
	
	/**
	 * 已预订状态
	 * @author Administrator
	 *
	 */
	public class BookedState implements State {
	
		@Override
		public void handle() {
			System.out.println("房间已预订！别人不能定！");
		}
	
	}

环境类：  
	
	/**
	 * 房间对象
	 * @author Administrator
	 *
	 */
	public class HomeContext {
		//如果是银行系统，这个Context类就是账号。根据金额不同，切换不同的状态！
		
		private State state;
		
		
		public void setState(State s){
			System.out.println("修改状态！");
			state = s;
			state.handle();
		}
		
	}

调用：

	public class Client {
		public static void main(String[] args) {
			HomeContext ctx = new HomeContext();
			
			ctx.setState(new FreeState());
			ctx.setState(new BookedState());
			
		}
	}

常见场景：线程对象各状态之间的切换。

##观察者模式  
观察者模式主要用于1：N的通知，当一个对象（目标对象Subject或Objservable）的状态变化时，他需要及时告知一系列对象（观察者对象，Observer），令他们做出响应

示例：

观察者接口：

	public interface Observer {
		void  update(Subject subject);
	}

观察者实现类：

	public class ObserverA implements Observer {
	
		private int myState;   //myState需要跟目标对象的state值保持一致！
		
		
		@Override
		public void update(Subject subject) {
			myState = ((ConcreteSubject)subject).getState();
		}
	
	
		public int getMyState() {
			return myState;
		}
		public void setMyState(int myState) {
			this.myState = myState;
		}	
	}

订阅者（被观察者）公共类：

		public class Subject {
		
		protected List<Observer> list = new ArrayList<Observer>();
		
		public void registerObserver(Observer obs){
			list.add(obs);
		}
		public void removeObserver(Observer obs){
			list.add(obs);
		}
	
		//通知所有的观察者更新状态
		public void notifyAllObservers(){
			for (Observer obs : list) {
				obs.update(this);
			}
		}	
	}

订阅者公共类子类：

	public class ConcreteSubject extends Subject {
		
		private int state;
	
		public int getState() {
			return state;
		}
	
		public void setState(int state) {
			this.state = state;
			//主题对象(目标对象)值发生了变化，请通知所有的观察者
			this.notifyAllObservers();
		} 
	}

调用：

	public class Client {
		public static void main(String[] args) {
			//目标对象
			ConcreteSubject subject = new ConcreteSubject();
			
			//创建多个观察者
			ObserverA  obs1 = new ObserverA();
			ObserverA  obs2 = new ObserverA();
			ObserverA  obs3 = new ObserverA();
			
			//将这三个观察者添加到subject对象的观察者队伍中
			subject.registerObserver(obs1);
			subject.registerObserver(obs2);
			subject.registerObserver(obs3);
			
			
			//改变subject的状态
			subject.setState(3000);
			System.out.println("########################");
			//我们看看，观察者的状态是不是也发生了变化
			System.out.println(obs1.getMyState());
			System.out.println(obs2.getMyState());
			System.out.println(obs3.getMyState());
			
			//改变subject的状态
			subject.setState(30);
			System.out.println("########################");
			//我们看看，观察者的状态是不是也发生了变化
			System.out.println(obs1.getMyState());
			System.out.println(obs2.getMyState());
			System.out.println(obs3.getMyState());
			
			
		}
	}

##备忘录模式 memento
保存某个对象内部状态的拷贝，这样以后就可以恢复到原先的状态。  
结构：   
源发器类Originator   
备忘录类Memento   
负责人类CareTake

示例：

源发器：

	/**
	 * 源发器类
	 * @author Administrator
	 *
	 */
	public class Emp {
		private String ename;
		private int age;
		private double salary;
		
		
		//进行备忘操作，并返回备忘录对象
		public EmpMemento  memento(){
			return new EmpMemento(this);
		}
		
		
		//进行数据恢复，恢复成制定备忘录对象的值
		public void recovery(EmpMemento mmt){
			this.ename = mmt.getEname();
			this.age = mmt.getAge();
			this.salary = mmt.getSalary();
		}
		
		
		public Emp(String ename, int age, double salary) {
			super();
			this.ename = ename;
			this.age = age;
			this.salary = salary;
		}
		public String getEname() {
			return ename;
		}
		public void setEname(String ename) {
			this.ename = ename;
		}
		public int getAge() {
			return age;
		}
		public void setAge(int age) {
			this.age = age;
		}
		public double getSalary() {
			return salary;
		}
		public void setSalary(double salary) {
			this.salary = salary;
		}
		
		
	}

备忘录：

	/**
	 * 备忘录类
	 * @author Administrator
	 *
	 */
	public class EmpMemento {
		private String ename;
		private int age;
		private double salary;
		
		
		public EmpMemento(Emp e) {
			this.ename = e.getEname();
			this.age = e.getAge();
			this.salary = e.getSalary();
		}
		
		
		public String getEname() {
			return ename;
		}
		public void setEname(String ename) {
			this.ename = ename;
		}
		public int getAge() {
			return age;
		}
		public void setAge(int age) {
			this.age = age;
		}
		public double getSalary() {
			return salary;
		}
		public void setSalary(double salary) {
			this.salary = salary;
		}	
	}

负责人类：

	/**
	 * 负责人类
	 * 负责管理备忘录对象
	 * @author Administrator
	 *
	 */
	public class CareTaker {
		
		private EmpMemento memento;
	
	//	private List<EmpMemento> list = new ArrayList<EmpMemento>();可以用容器存储多个恢复点
		
		
		
		public EmpMemento getMemento() {
			return memento;
		}
	
		public void setMemento(EmpMemento memento) {
			this.memento = memento;
		}	
	}

调用:

	public class Client {
		public static void main(String[] args) {
			CareTaker taker = new CareTaker();
			
			Emp emp = new Emp("高淇", 18, 900);
			System.out.println("第一次打印对象："+emp.getEname()+"---"+emp.getAge()+"---"+emp.getSalary());
			
			taker.setMemento(emp.memento());   //备忘一次
			
			emp.setAge(38);
			emp.setEname("搞起");
			emp.setSalary(9000);
			System.out.println("第二次打印对象："+emp.getEname()+"---"+emp.getAge()+"---"+emp.getSalary());
			
			emp.recovery(taker.getMemento()); //恢复到备忘录对象保存的状态
			
			System.out.println("第三次打印对象："+emp.getEname()+"---"+emp.getAge()+"---"+emp.getSalary());
			
		}
	}
