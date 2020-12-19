#### 第三次Spring分享会遗留问题回归：

##### 1、枚举类单例和登记式单例的区别

（1）枚举类单例

在实现枚举类单例模式之前，首先先理解一个问题：Java枚举的本质是？

枚举本质上是通过普通的类来实现的，每个枚举类型都继承自java.lang.Enum，并自动添加了values和valueOf方法。而每个枚举常量是一个静态常量字段，使用内部类实现，该内部类继承了枚举类。所有枚举常量都通过静态代码块来进行初始化。

例：

```java
/**
 * @author ：xianhuili
 */
public enum MeterType {
    
    SINGLE_PHASE_METER,
    
    THREE_PHASE_METER,
    
    CT_METER,
}
```

在java编译之后，编译后MeterType.class内容如下：

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.boot.proxy.singleton;

public enum MeterType {
    SINGLE_PHASE_METER,
    THREE_PHASE_METER,
    CT_METER;

    private MeterType() {
    }
}
```

从编译文件可以看到，枚举类的构造器默认就是私有的构造器，那么也就是说，我们无法实例化枚举，满足单例模式私有构造的要求，private修饰符对于构造器是可以省略的，但这不代表构造器的权限是默认权限。

使用javap .\MeterType.class反编译class文件，内容如下

```java
PS D:\Users\xianhuili\project\boot-security-master\target\classes\com\boot\proxy\singleton>
    javap .\MeterType.class
        
Compiled from "MeterType.java"
public final class com.boot.proxy.singleton.MeterType extends java.lang.Enum<com.boot.proxy.singleton.MeterType> {
  public static final com.boot.proxy.singleton.MeterType SINGLE_PHASE_METER;
  public static final com.boot.proxy.singleton.MeterType THREE_PHASE_METER;
  public static final com.boot.proxy.singleton.MeterType CT_METER;
  public static com.boot.proxy.singleton.MeterType[] values();
  public static com.boot.proxy.singleton.MeterType valueOf(java.lang.String);
  static {};
}
```

枚举类中每一个如SINGLE_PHASE_METER属性就是定义了一个静态的final修饰的MeterType类型常量，

并且继承了values和valueOf方法，MeterType都继承了 Enum 类，同时也会 toString 、equals、hashcode 等方法。

```java

public String toString() {
    return name;
}
public final boolean equals(Object other) {
    return this==other;
}
public final int hashCode() {
    return super.hashCode();
}
```

因为 equals、hashcode 方法是 final 的，所以不可以被枚举重写（只可以继承）。

因此可以这样理解：当这个枚举类的实例个数只有一个的时候, 这个类就变成了单例模式了

再看看枚举类单例的代码实现：

```java
package com.boot.proxy.singleton;

/**
 * @author ：xianhuili
 */
public class EnumSingletonTest {

    //私有化构造函数
    private EnumSingletonTest(){ }

    //定义一个静态枚举类
     enum SingletonEnum{
        //创建一个枚举对象，该对象天生为单例
        INSTANCE;
        private EnumSingletonTest enumSingletonTest;
        //私有化枚举的构造函数
        SingletonEnum(){
            enumSingletonTest = new EnumSingletonTest();
        }
        public EnumSingletonTest getInstance(){
            return enumSingletonTest;
        }
    }

    //对外暴露一个获取User对象的静态方法
    public static EnumSingletonTest getInstance(){
        return SingletonEnum.INSTANCE.getInstance();
    }
}
```

首先看到的是EnumSingletonTest类实现单例，java枚举型是静态常量，隐式地用static final修饰，内部类enum不用加关键字static修饰

Singleton模式有三个特性，自由序列化，线程安全，单例唯一；

枚举单例实现

a.序列化：

查询资料得知，对于序列化和反序列化，因为每一个枚举类型和枚举变量在JVM中都是唯一的，即Java在序列化和反序列化枚举时做了特殊的规定，枚举的writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法是被编译器禁用的，因此也不存在实现序列化接口后调用readObject会破坏单例的问题

b.线程安全

类似于普通的饿汉模式，通过在第一次调用时的静态初始化创建的对象是线程安全的

c.单例唯一

Enum是Java提供给编译器的一个用于继承的类。枚举量的实现其实是public static final T 类型的未初始化变量，之后，会在静态代码中对枚举量进行初始化，如果用枚举去只有一个枚举，则全局只会有一个唯一的

对于单例模式的破坏有两种方式：反射攻击和序列化攻击

枚举单例模式刚好防御这两种攻击

反射攻击

有Java枚举都隐式继承自Enum抽象类，而Enum抽象类根本没有无参构造方法，只有如下一 个构造方法：

```java
protected Enum(String name, int ordinal) { 
this.name = name; 
this.ordinal = ordinal; 
}
```

同时在Constructor.newInstance()方法中有如下代码：

```java
if ((clazz.getModifiers() & Modifier.ENUM) != 0) throw new IllegalArgumentException("Cannot reflectively create enum objects");
```

如果是枚举类则直接抛出异常禁止反射创建

序列化攻击

通过类描述符取得枚举单例的类型EnumSingleton； 取得枚举单例中的枚举值的名字（这里是INSTANCE）； 调用Enum.valueOf()方法，根据枚举类型和枚举值的名字，获得最终的单例，此三个步骤主要绕过反射直接获取单例为目标。

最后再回到EnumSingletonTest类，EnumSingletonTest类本身构造器也要私有化，防止被实例化。

（2）登记式单例

传统单例模式最大的缺陷在于：a.所有单例类构造器私有化，导致无法被继承；b.在编写代码时不希望在一开始的时候就把一个类写成单例模式，但是在运用这个类的时候，我们却可以像单例一样使用他，传统单例模式无法实现。

登记式单例的出现就是解决这两个问题：

登记式单例实际上维护的是一组单例类的实例，将这些实例存储到一个Map(登记簿)中，对于已经登记过的单例，则从工厂直接返回，对于没有登记的，则先登记，而后返回；其实登记式单例并没有去改变类，他所做的就是起到一个登记并且管理的作用，如果没有登记，就先登记，并把生成的实例保存起来，使用的时候从登记簿中取出。

登记式单例实现过程就不讲解了，上次分享会的时候，我已经讲过了。

总结：登记式单例和枚举类单例是完全不同的东西，唯一的共同点就是都实现类的单例，登记式单例属于一个设计思想，解决传统单例模式的缺陷，因此不会直接对类进行设计和实现，而是对类进行登记和管理，不改变类的内容更方便编程，而枚举类单例是一种传统单例模式，需要在类中实现类的单例。

#####  2、三级缓存如何处理动态代理

（1）静态、动态代理

静态、动态代理：首先被代理类是通过代理类的构造方法写入的，一般情况下。代理的类型是接口，不会发生循环依赖的问题，其次如果代理类和和被代理类发生了循环依赖，spring三级缓存无法解决。

spring三级缓存总结：

不能解决的情况：

- 构造器注入循环依赖
- prototype模式field属性注入循环依赖

能解决的情况：

- singleton模式field属性注入（setter方法注入）循环依赖

构造器注入循环依赖不能解决原因：

Spring解决循环依赖依靠的是Bean的“中间态”这个概念，Spring的循环依赖的理论依据基于Java的引用传递，当获得对象的引用时，对象的属性是可以延后设置的。，而这个中间态指的是`已经实例化`，但还没初始化的状态。而构造器是完成实例化的东东，所以构造器的循环依赖无法解决。

prototype模式field属性注入循环依赖不能解决：无法保证线程安全，多例模式下spring直接抛出异常

不过我个人理解，循环依赖本就是代码设计的缺陷问题，spring只是帮忙规避了这个问题，但并代表可以为所欲为的循环依赖，按照代码规范本就不应该出现这个问题。

（2）CGLib

同上

（3）AOP代理

如下情形：

A -> A的AOP代理对象

B -> B的AOP代理对象

如果A和B 、A代理和B代理两相依，spring各走各的三级缓存流程

如果4个相互依赖，则按依赖顺序完成所有的以来创建

（4）循环依赖中的代理对象替换

这个问题产生主要在于spring 创建对象时在AbstractAutowireCapableBeanFactory类doCreateBean的方法中有段代码：

```java
if (exposedObject == bean) {
   exposedObject = earlySingletonReference;
}
```

这段代码的意思是如果exposedObject和bean相等则是同一对象，那exposedObject将引用于在二级缓存中去取出的对象；有点懵逼，还可以不是同一对象？

```java
try {
   populateBean(beanName, mbd, instanceWrapper);
   exposedObject = initializeBean(beanName, exposedObject, mbd);
}
```

在initializeBean方法进入查看

```java
if (mbd == null || !mbd.isSynthetic()) {
   wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
}
		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
if (mbd == null || !mbd.isSynthetic()) {
   wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}
```

这里面有两段代码，一个是在对象实例化之前执行，一个是在之后实行

进入applyBeanPostProcessorsAfterInitialization方法

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
      Object current = beanProcessor.postProcessAfterInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```

这里调用了BeanPostProcessor 的postProcessAfterInitialization方法

（5）BeanPostProcessor 

BeanPostProcessor 接口是Bean后置处理器。接口实现类有postProcessBeforeInitialization和afterProcessBeforeInitialization两个方法，分别在bean创建前和创建后出发调用(bean创建/初始化的顺序，需要进一步深化看一下)。因此bean创建之前或者之后对对应bean进行一些处理。这两个方法最终的返回值还是bean。但是在spring程序中，afterProcessBeforeInitialization这个方法不会讲bean注入到容器中

这可以简单理解为是类似AOP的实现。利用beanpostProcessor，捕获部分bean的加载，并对这些bean生成动态代理，并将代理类放到spring容器中，后续调用原本这个bean类的方法时，就会直接进代理类的invoke方法，实现对某些类的某些方法(比如自定义注解)的增强等。

代码实现：

通过实现BeanPostProcessor类可以在ProcessImpl实现类创建的时候生成一个ProcessImpl代理类

而这里可以TestA理解为是为了生成ProcessImpl代理类的代理类

```java
/**
 * @author ：xianhuili
 */
@Component
public class TestA implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        DynamicDeviceProcessProxy proxy = new DynamicDeviceProcessProxy(bean);
        if(beanName.contains("ProcessImpl")){
            Class[] iterClass = bean.getClass().getInterfaces();
            if(iterClass.length > 0){
                return Proxy.newProxyInstance(bean.getClass().getClassLoader(),iterClass,proxy);
            }else {
                return bean;
            }
        }else{
            return bean;
        }
    }
}
```



```java
/**
 * @author ：xianhuili
 * @date ：Created in 2020/4/19 23:20
 */
@Component
public class MeterProcessImpl implements DeviceProcess {

    private static final Logger log = LoggerFactory.getLogger(MeterProcessImpl.class);

    @Override
    public String install() {
        log.info("install meter");
        return "install meter finished";
    }

    @Override
    public void replace() {
        log.info("replace meter");
    }

    @Override
    public void remove() {
        log.info("remove meter");
    }
}

/**
 * @author ：xianhuili
 * @date ：Created in 2020/4/20 0:02
 */
@Component
public class RepeaterProcessImpl implements DeviceProcess {

    private static final Logger log = LoggerFactory.getLogger(RepeaterProcessImpl.class);

    @Override
    public String install() {
        log.info("install repeater");
        return "";
    }

    @Override
    public void replace() {
        log.info("replace repeater");
    }

    @Override
    public void remove() {
        log.info("remove repeater");
    }
}
```

```java
/**
 * @author ：xianhuili
 * @date ：Created in 2020/4/19 23:46
 */
@Component
public class DynamicDeviceProcessProxy implements InvocationHandler {
    //维护一个目标对象
    private Object target;

    public DynamicDeviceProcessProxy(Object target) {
        this.target = target;
    }

    public void pre()  {
        System.out.println("代理类的前方法");
    }

    public void late() {
        System.out.println("代理类的后方法");
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(target, args);
    }
}
```



写了这么多代码就是验证了一个东西，就是通过BeanPostProcessor aop的功能，在对象实例化的时候进行拦截，拦截后可以（自己继承实现）将正在创建对象的代理对象返回，并将所有的代理的接口实现类的代理类注入到容器中，这样就可以直接对实现类进行部分增强。

##### 3、spring 多容器

spring是可以实现多容器管理的，容器之间是是独立的，互相并不干扰。在同一容器中bean明必须唯一，spring直接抛出异常，在程序里加载加载哪个容器就用哪个容器！

比如spring和springMVC是两个容器，spring容器和springmvc容器的关系是父子容器的关系。spring容器是父容器，springmvc是子容器。在子容器里可以访问父容器里的对象，但是在父容器里不可以访问子容器的对象，说的通俗点就是，在controller里可以访问service对象，但是在service里不可以访问controller对象。