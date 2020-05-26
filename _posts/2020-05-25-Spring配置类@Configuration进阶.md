---
layout:     post
title:      Spring配置类@Configuration进阶
subtitle:   Configuration进阶
date:       2020-05-25
author:     geek_li
header-img: img/codeviewer2.jpg
catalog: true
tags:
    - Configuration进阶
---


很久之前我问了同学，什么是Spring  
回答我说，Spring是春天啊（此时我感觉我被嘲笑了）  

回到正文：  

### 版本约定

* JDK ： 1.8
* Spring Framework：5.2.2.RELEASE  

### 正文

“ BeanFactoryAwareMethodInterceptor ”，写过spring工程的都知道，它会拦截setBeanFactory()方法从而完成给代理类指定属性赋值。通过第一个拦截器的讲解，你能够成功“忽悠”很多人了，但仍旧不能够解释我们最常使用中的这个疑惑：为何通过调用@Bean方法最终指向的仍旧是同一个Bean呢？  

上面见到的术语看不懂的请不要上车了  

### Spring配置类的使用误区

根据不同的配置方式，展示不同情况。从Lite模式的使用产生误区，到使用Full模式解决问题，最后引出解释为何有此效果的原因分析/源码解析。  

#### Lite模式：错误姿势

配置类：  
````

public class AppConfig {

    @Bean
    public Son son() {
        Son son = new Son();
        System.out.println("son created..." + son.hashCode());
        return son;
    }

    @Bean
    public Parent parent() {
        Son son = son();
        System.out.println("parent created...持有的Son是：" + son.hashCode());
        return new Parent(son);
    }

}

````  

程序：
````

public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

    AppConfig appConfig = context.getBean(AppConfig.class);
    System.out.println(appConfig);

    // bean情况
    Son son = context.getBean(Son.class);
    Parent parent = context.getBean(Parent.class);

    System.out.println("容器内的Son实例：" + son.hashCode());
    System.out.println("容器内Person持有的Son实例：" + parent.getSon().hashCode());
    System.out.println(parent.getSon() == son);
}

````  

执行结果：  
````
son created...624271064
son created...564742142
parent created...持有的Son是：564742142
com.handsomegeekli.fullliteconfig.config.AppConfig@1a38c59b
容器内的Son实例：624271064
容器内Person持有的Son实例：564742142
false

````  


结果分析：  

* Son实例被创建了2次。很明显这两个不是同一个实例
* 第一次是由Spring创建并放进容器里（624271064这个）
* 第二次是由构造parent时创建，只放进了parent里，并没放进容器里（564742142这个）  

这样的话，就出问题了。问题表现在这两个方面：  

1. Son对象被创建了两次，单例模式被打破
2. 对Parent实例而言，它依赖的Son不再是IoC容器内的那个Bean，而是一个非常普通的POJO对象而已。所以这个Son对象将不会享有Spring带来的任何“好处”，这在实际场景中一般都是会有问题的  

上面的二点已经很说明问题啦，怎么避免 ？ 

#### Lite模式：正确姿势  

配置：
````
@Bean
public Parent parent(Son son){
    System.out.println("parent created...持有的Son是：" + son.hashCode());
    return new Parent(son);
}
````

运行结果：
````
son created...624271064
parent created...持有的Son是：624271064
com.handsomegeekli.fullliteconfig.config.AppConfig@667a738
容器内的Son实例：624271064
容器内Person持有的Son实例：624271064
true
````

完美解决问题，如果你坚持使用Lite模式，那么请注意它的优缺点  

有可能会出现的一种情况就是：明明是按照按照第一种方式去写，也会是正常通过的，并没有上面说的问题，那要看看你到底用没用@Configuration注解了。  

#### Full模式

Full模式是容错性最强的一种方式，你乱造都行，没啥顾虑。  

当然啦，方法不能是private/final。但一般情况下在配置里final调一个方法就扑街啦  

````

@Configuration
public class AppConfig {

    @Bean
    public Son son() {
        Son son = new Son();
        System.out.println("son created..." + son.hashCode());
        return son;
    }

    @Bean
    public Parent parent() {
        Son son = son();
        System.out.println("parent created...持有的Son是：" + son.hashCode());
        return new Parent(son);
    }

}

````

运行结果：  

````

son created...1797712197
parent created...持有的Son是：1797712197
com.handsomegeekli.fullliteconfig.config.AppConfig$$EnhancerBySpringCGLIB$$8ef51461@be64738
容器内的Son实例：1797712197
容器内Person持有的Son实例：1797712197
true
````

结果是完美的。它能够保证你通过调用标注有@Bean的方法得到的是IoC容器里面的实例对象，而非重新创建一个。相比较于Lite模式，它还有另外一个区别：它会为配置类生成一个CGLIB的代理子类对象放进容器，而Lite模式放进容器的是原生对象。  

凡事皆有代价，一切皆在取舍。原生的才是效率最高的，是对Cloud Native最为友好的方式。但在实际“推荐使用”上，业务端开发一般只会使用Full模式，毕竟业务开发的同学水平是残参差不齐的，容错性就显得至关重要了。  

如果是开发容器、中间件...推荐使用Lite模式配置，为容器化、Cloud Native做好准备  

Full模式既然是面向使用侧为常用的方式，那么接下来就趴一趴Spring到底是施了什么“魔法”，让调用@Bean方法竟然可以不进入方法体内而指向同一个实例。  

### BeanMethodInterceptor拦截器 entree

终于到了今天的主菜。关于前面的流程分析本文就一步跳过，单刀直入分析BeanMethodInterceptor这个拦截器，也也就是所谓的两个拦截器的后者。  

相较于上个拦截器，这个拦截器不可为不复杂。官方解释它的作用为：拦截任何标注有@Bean注解的方法的调用，以确保正确处理Bean语义，例如作用域（请别忽略它）和AOP代理。  

复杂归复杂，但没啥好怕的，一步一步来呗。同样的，我会按如下两步去了解它：执行时机 + 做了何事。  

#### 执行时机

源码：
````

BeanMethodInterceptor：

@Override
public boolean isMatch(Method candidateMethod) {
    return (candidateMethod.getDeclaringClass() != Object.class &&
            !BeanFactoryAwareMethodInterceptor.isSetBeanFactory(candidateMethod) &&
            BeanAnnotationHelper.isBeanAnnotated(candidateMethod));
}

````

三行代码，三个条件  
1. 该方法不能是Object的方法（即使你Object的方法标注了@Bean，我也不认）
2. 不能是setBeanFactory()方法。这很容易理解，它交给上个拦截器搞定即可
3. 方法必须标注标注有@Bean注解  

简而言之，标注有@Bean注解方法执行时会被拦截。  

所以下面例子中的son()和parent()这两个，以及parent()里面调用的son()方法的执行它都会拦截（一共拦截3次）~  

注意：方法只要是个Method即可，无论是static方法还是普通方法，都会“参与”此判断逻辑哦  

#### 做了什么事情

这里是具体拦截逻辑，会比第一个拦截器复杂很多。源码不算非常的多，但牵扯到的东西还真不少，比如AOP、比如Scope、比如Bean的创建等等，理解起来还蛮费劲的。

本处以拦截到parent()方法的执行为例，结合源码进行跟踪讲解：  

````

BeanMethodInterceptor：

// enhancedConfigInstance：被拦截的对象实例，也是代理对象
// beanMethod：parent()方法
// beanMethodArgs：空
// cglibMethodProxy：代理。用于调用其invoke/invokeSuper()来执行对应的方法
@Override
@Nullable
public Object intercept(Object enhancedConfigInstance, 
    Method beanMethod, Object[] beanMethodArgs, MethodProxy cglibMethodProxy) throws Throwable {

    // 通过反射，获取到Bean工厂。也就是$$beanFactory这个属性的值~
    ConfigurableBeanFactory beanFactory = getBeanFactory(enhancedConfigInstance);
    // 拿到Bean的名称
    String beanName = BeanAnnotationHelper.determineBeanNameFor(beanMethod);

    // 判断这个方法是否是Scoped代理对象 很明显本利里是没有标注的 暂先略过
    // 简答的说：parent()方法头上是否标注有@Scoped注解~~~
    if (BeanAnnotationHelper.isScopedProxy(beanMethod)) {
        String scopedBeanName = ScopedProxyCreator.getTargetBeanName(beanName);
        if (beanFactory.isCurrentlyInCreation(scopedBeanName)) {
            beanName = scopedBeanName;
        }
    }

    // ========下面要处理bean间方法引用的情况了========
    // 首先：检查所请求的Bean是否是FactoryBean。也就是bean名称为`&parent`的Bean是否存在
    // 如果是的话，就创建一个代理子类，拦截它的getObject()方法以返回容器里的实例
    // 这样做保证了方法返回一个FactoryBean和@Bean的语义是效果一样的，确保了不会重复创建多个Bean
    if (factoryContainsBean(beanFactory, BeanFactory.FACTORY_BEAN_PREFIX + beanName) &&
            factoryContainsBean(beanFactory, beanName)) {

        // 先得到这个工厂Bean
        Object factoryBean = beanFactory.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName);
        if (factoryBean instanceof ScopedProxyFactoryBean) {
            // Scoped proxy factory beans are a special case and should not be further proxied
            // 如果工厂Bean已经是一个Scope代理Bean，则不需要再增强
            // 因为它已经能够满足FactoryBean延迟初始化Bean了~
        }

        // 继续增强
        else {
            return enhanceFactoryBean(factoryBean, beanMethod.getReturnType(), beanFactory, beanName);
        }
    }


    // 检查给定的方法是否与当前调用的容器相对应工厂方法。
    // 比较方法名称和参数列表来确定是否是同一个方法
    // 怎么理解这句话，参照下面详解吧
    if (isCurrentlyInvokedFactoryMethod(beanMethod)) {

        // 这是个小细节：若你@Bean返回的是BeanFactoryPostProcessor类型
        // 请你使用static静态方法，否则会打印这句日志的~~~~
        // 因为如果是非静态方法，部分后置处理失效处理不到你，可能对你程序有影像
        // 当然也可能没影响，所以官方也只是建议而已~~~
        if (logger.isInfoEnabled() &&
                BeanFactoryPostProcessor.class.isAssignableFrom(beanMethod.getReturnType())) {
            ... // 输出info日志
        }

        // 这表示：当前parent()方法，就是这个被拦截的方法，那就没啥好说的 
        // 相当于在代理代理类里执行了super(xxx);
        // 但是，但是，但是，此时的this依旧是代理类
        return cglibMethodProxy.invokeSuper(enhancedConfigInstance, beanMethodArgs);
    }

    // parent()方法里调用的son()方法会交给这里来执行
    return resolveBeanReference(beanMethod, beanMethodArgs, beanFactory, beanName);
}

````  

总结：  

1. 拿到当前BeanFactory工厂对象。该工厂对象通过第一个拦截器BeanFactoryAwareMethodInterceptor已经完成了设值
2. 确定Bean名称。默认是方法名，若通过@Bean指定了以指定的为准，若指定了多个值以第一个值为准，后面的值当作Bean的alias别名
3. 判断当前方法(以parent()方法为例)是否是个Scope域代理。也就是方法上是否标注有@Scope注解
1. 若是域代理类，那旧以它的方式来处理喽。beanName的变化变化为scopedTarget.parent
2. 判断scopedTarget.parent这个Bean是否正在创建中...若是的，那就把当前beanName替换为scopedTarget.parent，以后就关注这个名称的Bean了~
3. 试想一下，如果不来这个判断的话，那最终可能的结果是：容器内一个名为parent的Bean，一个名字为scopedTarget.parent的Bean，那岂不又出问题了麽~
4. 判断请求的Bean是否是个FactoryBean工厂Bean。
1. 若是工厂Bean，那么就需要enhance增强这个Bean，以拦截它的getObject()方法
2. 拦截getObject()的做法是：当执行getObject()方法时转为 -> getBean()方法
3. 为什么需要这么做：是为了确保FactoryBean产生的实例是通过getBean()容器去获取的，而非又自己创建一个出来了
4. 这种case先打个❓，下面会结合代码示例加以说明
5. 判断这个beanMethod是否是当前正在被调用的工厂方法。
1. 若是正在创建的方法，那就好说了，直接super(xxx)执行父类方法体完事~
2. 若不是正在创建的方法，那就需要代理喽，以确保实际调用的仍旧是实际调用getBean方法而保证是同一个Bean
3. 这种case先打个❓，下面会结合代码示例加以说明。因为这个case是最常见的主线case，所以先把它搞定

这是该拦截器的执行步骤，留下两个打❓下面我来一一解释（按照倒序）。  

多次调用@Bean方法为何不会产生新实例？  

这是最常见的case，上实例：

````

@Configuration
public class AppConfig {

    @Bean
    public Son son() {
        Son son = new Son();
        System.out.println("son created..." + son.hashCode());
        return son;
    }

    @Bean
    public Parent parent() {
        notBeanMethod();
        Son son = son();
        System.out.println("parent created...持有的Son是：" + son.hashCode());
        return new Parent(son);
    }

    public void notBeanMethod(){
        System.out.println("notBeanMethod invoked by 【" + this + "】");
    }

}
````  

* son()，标注有@Bean。  
因此它最终交给cglibMethodProxy.invokeSuper(enhancedConfigInstance, beanMethodArgs);方法直接执行父类（也就是目标类）的方法体  

值得注意的是：此时所处的对象仍旧是代理对象内，这个方法体只是通过代理类调用了super(xxx)方法进来的而已嘛~  

* parent()：标注有@Bean。它内部会还会调用notBeanMethod()和son()两个方法

同上，会走到目标类的方法体里，开始调用 notBeanMethod()和son() 这两个方法，这个时候处理的方式就不一样了：

1. 调用notBeanMethod()方法，因为它没有标注@Bean注解，所以不会被拦截 -> 直接执行方法体
2. 调用son()方法，因为它标注有@Bean注解，所以会继续进入到拦截器里。但请注意和上面 直接调用 son()方法不一样的是：此时当前正在被invoked的方法是parent()方法，而并非son()方法，所以他会被交给resolveBeanReference()方法来处理  

BeanMethodInterceptor：  

````
private Object resolveBeanReference(Method beanMethod, Object[] beanMethodArgs, ConfigurableBeanFactory beanFactory, String beanName)
{
    
// 当前bean（son这个Bean）是否正在创建中... 本处为false嘛
// 这个判断主要是为了防止后面getBean报错~~~
boolean alreadyInCreation = beanFactory.isCurrentlyInCreation(beanName);
try {
    // 如果该Bean确实正在创建中，先把它标记下，放置后面getBean报错~
    if (alreadyInCreation) {
        beanFactory.setCurrentlyInCreation(beanName, false);
    }

    // 更具该方法的入参，决定后面使用getBean(beanName)还是getBean(beanName,args)
    // 基本原则是：但凡只要有一个入参为null，就调用getBean(beanName)
    boolean useArgs = !ObjectUtils.isEmpty(beanMethodArgs);
    if (useArgs && beanFactory.isSingleton(beanName)) {
        for (Object arg : beanMethodArgs) {
            if (arg == null) {
                useArgs = false;
                break;
            }
        }
    }
    // 通过getBean从容器中拿到这个实例  本处拿出的就是Son实例喽
    Object beanInstance = (useArgs ? beanFactory.getBean(beanName, beanMethodArgs) : beanFactory.getBean(beanName));

    // 方法返回类型和Bean实际类型做个比较，因为有可能类型不一样
    // 什么时候会出现类型不一样呢？当BeanDefinition定义信息类型被覆盖的时候，就可能出现此现象
    if (!ClassUtils.isAssignableValue(beanMethod.getReturnType(), beanInstance)) {
        if (beanInstance.equals(null)) {
            beanInstance = null;
        } else {
            ...
            throw new IllegalStateException(msg);
        }
    }

    // 当前被调用的方法，是parent()方法
    Method currentlyInvoked = SimpleInstantiationStrategy.getCurrentlyInvokedFactoryMethod();
    if (currentlyInvoked != null) {
        String outerBeanName = BeanAnnotationHelper.determineBeanNameFor(currentlyInvoked);
        // 这一步是注册依赖关系，告诉容器：
        // parent实例的初始化依赖于son实例
        beanFactory.registerDependentBean(beanName, outerBeanName);
    }
    // 返回实例
    return beanInstance;
}

// 归还标记：笔记实际确实还在创建中嘛~~~~
finally {
    if (alreadyInCreation) {
        beanFactory.setCurrentlyInCreation(beanName, true);
    }
}
}
````  

这么一来，执行完parent()方法体里的son()方法后，"实际得到的是容器内的实例"，从而保证了我们这么写是不会有问题的。

"notBeanMethod()"：因为没有标注@Bean，所以它并不会被容器调用，而只能是被上面的`parent()`方法调用到，并且也不会被拦截（值得注意的是：因为此方法不需要被代理，所以此方法可以是`private final`的哦~）

以上程序的运行结果是：  

````

son created...347978868
notBeanMethod invoked by 【com.handsomegeekli.fullliteconfig.config.AppConfig$$EnhancerBySpringCGLIB$$ec611337@12591ac8】
parent created...持有的Son是：347978868
com.handsomegeekli.fullliteconfig.config.AppConfig$$EnhancerBySpringCGLIB$$ec611337@12591ac8
容器内的Son实例：347978868
容器内Person持有的Son实例：347978868
true

````  

可以看到，Son自始至终都只存在一个实例，这是符合我们的预期的。


* Lite模式下表现如何？  

同样的代码，在Lite模式下（去掉@Configuration注解即可），不存在“如此复杂”的代理逻辑，所以上例的运行结果是：  

````
son created...624271064
notBeanMethod invoked by 【com.handsomegeekli.fullliteconfig.config.AppConfig@21a947fe】
son created...90205195
parent created...持有的Son是：90205195
com.handsomegeekli.fullliteconfig.config.AppConfig@21a947fe
容器内的Son实例：624271064
容器内Person持有的Son实例：90205195
false
````
这个结果很好理解，这里我就不再啰嗦了。总之就不能这么用就对了~  

* FactoryBean模式剖析  
FactoryBean也是向容器提供Bean的一种方式，如最常见的SqlSessionFactoryBean就是这么一个大代表，因为它比较常用，并且这里也作为此拦截器一个单独的执行分支，所以很有必要研究一番。  

执行此分支逻辑的条件是：容器内已经存在&beanName和beanName两个Bean。执行的方式是：使用enhanceFactoryBean()方法对FactoryBean进行增强。  

````
ConfigurationClassEnhancer：

// 创建一个子类代理，拦截对getObject()的调用，委托给当前的BeanFactory
// 而不是创建一个新的实例。这些代理仅在调用FactoryBean时创建
// factoryBean：从容器内拿出来的那个已经存在的工厂Bean实例（是工厂Bean实例）
// exposedType：@Bean标注的方法的返回值类型
private Object enhanceFactoryBean(Object factoryBean, Class<?> exposedType,
        ConfigurableBeanFactory beanFactory, String beanName) {

    try {
        // 看看Spring容器内已经存在的这个工厂Bean的情况，看看是否有final
        Class<?> clazz = factoryBean.getClass();
        boolean finalClass = Modifier.isFinal(clazz.getModifiers());
        boolean finalMethod = Modifier.isFinal(clazz.getMethod("getObject").getModifiers());

        // 类和方法其中有一个是final，那就只能看看能不能走接口代理喽
        if (finalClass || finalMethod) {
            // @Bean标注的方法返回值若是接口类型 尝试走基于接口的JDK动态代理
            if (exposedType.isInterface()) {
                // 基于JDK的动态代理
                return createInterfaceProxyForFactoryBean(factoryBean, exposedType, beanFactory, beanName);
            } else {
                // 类或方法存在final情况，但是呢返回类型又不是
                return factoryBean;
            }
        }
    }
    catch (NoSuchMethodException ex) {
        // 没有getObject()方法  很明显，一般不会走到这里
    }

    // 到这，说明以上条件不满足：存在final且还不是接口类型
    // 类和方法都不是final，生成一个CGLIB的动态代理
    return createCglibProxyForFactoryBean(factoryBean, beanFactory, beanName);
}
````  

总结：  

1. 拿到容器内已经存在的这个工厂Bean的类型，看看类上、getObject()方法是否用final修饰了
2. 但凡只需有一个被final修饰了，那注定不能使用CGLIB代理了喽，那么就尝试使用基于接口的JDK动态代理：
1. 若你标注的@Bean返回的是接口类型（也就是FactoryBean类型），那就ok，使用JDK创建个代理对象返回
2. 若不是接口（有final又还不是接口），那老衲无能为力了：原样return返回
3. 若以上条件不满足，表示一个final都木有，那就统一使用CGLIB去生成一个代理子类。大多数情况下，都会走到这个分支上，代理是通过CGLIB生成的  


说明：无论是JDK动态代理还是CGLIB的代理实现均非常简单，就是把getObject()方法代理为使用beanFactory.getBean(beanName)去获取实例（要不代理掉的话，每次不就执行你getObject()里面的逻辑了麽，就又会创建新实例啦~）  

需要明确，此拦截器对FactoryBean逻辑处理分支的目的是：确保你通过方法调用拿到FactoryBean后，再调用其getObject()方法（哪怕调用多次）得到的都是同一个示例（容器内的单例）。因此需要对getObject()方法做拦截嘛，让该方法指向到getBean()，永远从容器里面拿即可。  

这个拦截处理逻辑只有在@Bean方法调用时才有意义，比如parent()里调用了son()这样子才会起到作用，否则你就忽略它吧~  
针对于此，下面给出不同case下的代码示例，加强理解。  

代码示例（重要）  
准备一个SonFactoryBean用于产生Son实例：  

````

public class SonFactoryBean implements FactoryBean<Son> {
    @Override
    public Son getObject() throws Exception {
        return new Son();
    }

    @Override
    public Class<?> getObjectType() {
        return Son.class;
    }
}

````

并且在配置类里把它放好：  

````

@Configuration
public class AppConfig {

    @Bean
    public FactoryBean<Son> son() {
        SonFactoryBean sonFactoryBean = new SonFactoryBean();
        System.out.println("我使用@Bean定义sonFactoryBean：" + sonFactoryBean.hashCode());
        System.out.println("我使用@Bean定义sonFactoryBean identityHashCode：" + System.identityHashCode(sonFactoryBean));
        return sonFactoryBean;
    }

    @Bean
    public Parent parent(Son son) throws Exception {
        // 根据前面所学，sonFactoryBean肯定是去容器拿
        FactoryBean<Son> sonFactoryBean = son();
        System.out.println("parent流程使用的sonFactoryBean：" + sonFactoryBean.hashCode());
        System.out.println("parent流程使用的sonFactoryBean identityHashCode：" + System.identityHashCode(sonFactoryBean));
        System.out.println("parent流程使用的sonFactoryBean：" + sonFactoryBean.getClass());
        // 虽然sonFactoryBean是从容器拿的，但是getObject()你可不能保证每次都返回单例哦~
        Son sonFromFactory1 = sonFactoryBean.getObject();
        Son sonFromFactory2 = sonFactoryBean.getObject();
        System.out.println("parent流程使用的sonFromFactory1：" + sonFromFactory1.hashCode());
        System.out.println("parent流程使用的sonFromFactory1：" + sonFromFactory2.hashCode());
        System.out.println("parent流程使用的son和容器内的son是否相等：" + (son == sonFromFactory1));

        return new Parent(sonFromFactory1);
    }

}
````  

执行程序：  

````
@Bean
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

    SonFactoryBean sonFactoryBean = context.getBean("&son", SonFactoryBean.class);
    System.out.println("Spring容器内的SonFactoryBean：" + sonFactoryBean.hashCode());
    System.out.println("Spring容器内的SonFactoryBean：" + System.identityHashCode(sonFactoryBean));
    System.out.println("Spring容器内的SonFactoryBean：" + sonFactoryBean.getClass());

    System.out.println("Spring容器内的Son：" + context.getBean("son").hashCode());
}
````  

输出结果：  


我使用@Bean定义sonFactoryBean：313540687  
我使用@Bean定义sonFactoryBean identityHashCode：313540687  

parent流程使用的sonFactoryBean：313540687  
parent流程使用的sonFactoryBean identityHashCode：70807318  
parent流程使用的sonFactoryBean：class com.handsomegeekli.fullliteconfig.  config.SonFactoryBean$$EnhancerBySpringCGLIB$$1ccec41d  
parent流程使用的sonFromFactory1：910091170  
parent流程使用的sonFromFactory1：910091170  
parent流程使用的son和容器内的son是否相等：true  

Spring容器内的SonFactoryBean：313540687  
Spring容器内的SonFactoryBean：313540687   
Spring容器内的SonFactoryBean：class com.handsomegeekli.fullliteconfig.config.SonFactoryBean  
Spring容器内的Son：910091170  

结果分析：  

达到了预期的效果：parent在调用son()方法时，得到的是在容器内已经存在的SonFactoryBean基础上CGLIB字节码提升过的实例，拦截成功，从而getObject()也就实际是去容器里拿对象的。  

通过本例有如下小细节需要指出：  

1. 原始对象和代理/增强后（不管是CGLIB还是JDK动态代理）的实例的.hashCode()以及.equals()方法是一毛一样的，但是identityHashCode()值（实际内存值）不一样哦，因为是不同类型、不同实例，这点请务必注意
2. 最终存在于容器内的仍旧是原生工厂Bean对象，而非代理后的工厂Bean实例。毕竟拦截器只是拦截了@Bean方法的调用来了个“偷天换日”而已~
3. 若SonFactoryBean上加个final关键字修饰，根据上面讲述的逻辑，那代理对象会使用JDK动态代理生成喽，形如这样（本处仅作为示例，实际使用中请别这么干）：  

public final class SonFactoryBean implements FactoryBean<Son> { ... }

再次运行程序，结果输出为：执行的结果一样，只是代理方式不一样而已。从这个小细节你也能看出来Spring对代理实现上的偏向：优先选择CGLIB代理方式，JDK动态代理方式用于兜底。  

提示：若你标注了final关键字了，那么请保证@Bean方法返回的是FactoryBean接口，而不能是SonFactoryBean实现类，否则最终无法代理了，原样输出。因为JDK动态代理和CGLIB都搞不定了嘛~  

在以上例子的基础上，我给它“加点料”，再看看效果呢：

使用BeanDefinitionRegistryPostProcessor提前就放进去一个名为son的实例：  


````

// 这两种方式向容器扔bd or singleton bean都行  我就选择第二种喽
// 注意：此处放进去的是BeanFactory工厂，名称是son哦~~~  不要写成了&son
@Component
public class SonBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        // registry.registerBeanDefinition("son", BeanDefinitionBuilder.rootBeanDefinition(SonFactoryBean.class).getBeanDefinition());
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        SonFactoryBean sonFactoryBean = new SonFactoryBean();
        System.out.println("初始化时，注册进容器的sonFactoryBean：" + sonFactoryBean);
        beanFactory.registerSingleton("son", sonFactoryBean);
    }
} 

````

输出结果：  

初始化时最早进容器的sonFactoryBean：2027775614   
初始化时最早进容器的sonFactoryBean identityHashCode：2027775614  

parent流程使用的sonFactoryBean：2027775614  
parent流程使用的sonFactoryBean identityHashCode：1183888521  
parent流程使用的sonFactoryBean：class com.handsomegeekli.fullliteconfig.config.SonFactoryBean$$EnhancerBySpringCGLIB$$1ccec41d  
parent流程使用的sonFromFactory1：2041605291  
parent流程使用的sonFromFactory1：2041605291  
parent流程使用的son和容器内的son是否相等：true  

Spring容器内的SonFactoryBean：2027775614  
Spring容器内的SonFactoryBean：2027775614  
Spring容器内的SonFactoryBean：class com.handsomegeekli.fullliteconfig.config.SonFactoryBean  
Spring容器内的Son：2041605291  


效果上并不差异，从日志上可以看到：你配置类上使用@Bean标注的son()方法体并没执行了，而是使用的最开始注册进去的实例，差异仅此而已。  

为何是这样的现象？这就不属于本文的内容了，是Spring容器对Bean的实例化、初始化逻辑，本公众号后面依旧会采用专栏式讲解，让你彻底弄懂它。当前有兴趣的可以先自行参考DefaultListableBeanFactory#preInstantiateSingletons的内容~   

Lite模式下表现如何？

Lite模式下可没这些“加强特性”，所以在Lite模式下（拿掉@Configuration这个注解便可）运行以上程序，结果输出为：  

Lite模式下可没这些“加强特性”，所以在Lite模式下（拿掉@Configuration这个注解便可）运行以上程序，结果输出为：

我使用@Bean定义sonFactoryBean：477289012  
我使用@Bean定义sonFactoryBean identityHashCode：477289012  

我使用@Bean定义sonFactoryBean：2008966511  
我使用@Bean定义sonFactoryBean identityHashCode：2008966511  
parent流程使用的sonFactoryBean：2008966511  
parent流程使用的sonFactoryBean identityHashCode：2008966511  
parent流程使用的sonFactoryBean：class com.handsomegeekli.fullliteconfig.config.SonFactoryBean  
parent流程使用的sonFromFactory1：433874882  
parent流程使用的sonFromFactory1：572191680  
parent流程使用的son和容器内的son是否相等：false  

Spring容器内的SonFactoryBean：477289012  
Spring容器内的SonFactoryBean：477289012  
Spring容器内的SonFactoryBean：class com.handsomegeekli.fullliteconfig.config.SonFactoryBean  
Spring容器内的Son：211968962  

结果解释我就不再啰嗦，有了前面的基础就太容易理解了,这里在解释，我就成真傻了。  

为何是@Scope域代理就不用处理？  
要解释好这个原因，和@Scope代理方式的原理知识强相关。限于篇幅，本文就先卖个关子~  

关于@Scope我个人觉得足够用5篇以上文章专题讲解，虽然在Spring Framework里使用得比较少，但是在理解Spirng Cloud的自定义扩展实现上显得非常非常有必要，所以你可关注我公众号，会近期推出相关专栏的。    

### 总结

呃............（被盯着这里看，该有的都在上面了啦）  

参考自————《Spring In Action》、《Spring实战》
参考自博客——https://www.yourbatman.cn  

to be continued