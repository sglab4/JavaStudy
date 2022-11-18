参考 https://www.cnblogs.com/javazhiyin/p/10905294.html, https://blog.csdn.net/weixin_39911567/article/details/111039200

![深究Spring中Bean的生命周期](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209191554357.jpeg)

如上图所示，Bean 的生命周期还是比较复杂的，下面来对上图每一个步骤做文字描述:

1. Spring启动，查找并加载需要被Spring管理的bean，利用 Java Reflection API 创建一个Bean的实例
2. Bean实例化后对将Bean的引入和值注入到Bean的属性中
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

------

Bean 的生命周期概括起来就是 **4 个阶段**：

1. 实例化（Instantiation）
2. 属性赋值（Populate）
3. 初始化（Initialization）
4. 销毁（Destruction）

![374544735d0dd4104601ef98749c7f89.png](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209211023063.jpeg)

![e8a473640bcd06e544c86fa729133ac3.png](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202202221710070.jpeg)

```java
// public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory 
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
    }

    if (instanceWrapper == null) {
        // 实例化
        instanceWrapper = this.createBeanInstance(beanName, mbd, args);
    }
    
    ....
    
    Object exposedObject = bean;

    try {
        // 属性赋值
        this.populateBean(beanName, mbd, instanceWrapper);
        // 初始化
        exposedObject = this.initializeBean(beanName, exposedObject, mbd);
    } catch (Throwable var18) {
        if (var18 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var18).getBeanName())) {
            throw (BeanCreationException)var18;
        }
	 try {
        // 销毁
        this.registerDisposableBeanIfNecessary(beanName, bean, mbd);
        return exposedObject;
    } catch (BeanDefinitionValidationException var16) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", var16);
    }
....
}
```

> **影响实例化和初始化的接口**

**InstantiationAwareBeanPostProcessor
BeanPostProcessor**

实现这两个接口的bean，会自动切入到相应的生命周期中，其中InstantiationAwareBeanPostProcessor是作用于实例化阶段的前后，BeanPostProcessor是作用于初始化阶段的前后。

**InstantiationAwareBeanPostProcessorAdapter ：**

```java
package cn.xmx.ioc.lifecycle;

import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessorAdapter;

import java.beans.PropertyDescriptor;

public class MyInstantiationAwareBeanPostProcessorAdapter extends InstantiationAwareBeanPostProcessorAdapter {
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if(beanName.equals("car")){
            System.out.println(beanName+"在实例化之前");
        }

        return super.postProcessBeforeInstantiation(beanClass, beanName);
    }
    
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(beanName.equals("car")){
            System.out.println(beanName+"在实例化之后");
        }
        return super.postProcessAfterInstantiation(bean, beanName);
    }

}
```

**BeanPostProcessor ：**

```java
package cn.xmx.ioc.lifecycle;

import java.lang.reflect.Proxy;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;


//后置处理器
public class NdBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
       System.out.println("NdBeanPostProcessor 在"+beanName+"对象初始化之【前】调用......");
       if(beanName.equals("car")) {
           return   new CglibInterceptor().getIntance(bean.getClass());
       }
       return bean;
        
    }
    
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("NdBeanPostProcessor 在"+beanName+"对象初始化之【后】调用......");
        return bean;
    }

}
```

```java
package cn.xmx.ioc.lifecycle;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/*
 四个主生命周期
         （1）实例化之前干预     InstantiationAwareBeanPostProcessorAdapter.postProcessBeforeInstantiation()
     1.实例化
          （2）实例化之后干预   InstantiationAwareBeanPostProcessorAdapter.postProcessAfterInstantiation()
     2.填充属性(给属性赋值)
     

          （1）初始化之前干预   BeanPostProcessor.postProcessBeforeInitialization()
     3.初始化化(比如准备资源文件)
             3) 属性进行干预  ----修改属性或属性值
     
         （2）初始化之后干预    BeanPostProcessor.postProcessAfterInitialization()
         
     4.销毁(释放资源---对象从内存销毁)

 N个接口
     1、干预多次
         1)BeanPostProcessor
     2、干预一次
         1）Aware
     
 */
public class TestLifeCycle {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class);
        Car car = (Car) context.getBean("car");
        System.out.println(car);//toString
        System.out.println("beanname:"+car.getBeanName());
        System.out.println("beanfactory:"+car.getBeanFactory());
        System.out.println("applicationContext:"+car.getApplicationContext());
    }
}
```

> **Aware 相关接口**

Spring 中提供的 Aware 接口有：

1. BeanNameAware：注入当前 bean 对应 beanName；
2. BeanClassLoaderAware：注入加载当前 bean 的 ClassLoader；
3. BeanFactoryAware：注入 当前BeanFactory容器 的引用。

```java
package cn.xmx.ioc.lifecycle;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class Car implements BeanNameAware, BeanFactoryAware, ApplicationContextAware {
    private String  name;
    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;
    public void init() {
        System.out.println("car 在初始化---加载资源");
    }
    
    public void destroy() {
        System.out.println("car 在销毁---释放资源");
    }

    public Car() {
        super();
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public Car(String name) {
        super();
        this.name = name;
        System.out.println("car实例化了");
    }


    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("beanFactory:"+beanFactory);
        this.beanFactory = beanFactory;
    }
    
    public BeanFactory getBeanFactory(){
        return this.beanFactory;
    }
    
    public void setBeanName(String s) {
        System.out.println("beanName:"+s);
        this.beanName = s ;
    }
    
    public String getBeanName() {
        return this.beanName;
    }
    
    public ApplicationContext getApplicationContext() {
        return this.applicationContext;
    }
    
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("applicationContext:"+applicationContext);
        this.applicationContext = applicationContext;
    }
}
```

> **总结**

最后总结下如何记忆 Spring Bean 的生命周期：

- 首先是实例化、属性赋值、初始化、销毁这 4 个大阶段；
- 再是初始化的具体操作，有 Aware 接口的依赖注入、BeanPostProcessor 在初始化前后的处理以及 InitializingBean 和 init-method 的初始化操作；
- 销毁的具体操作，有注册相关销毁回调接口，最后通过DisposableBean 和 destory-method 进行销毁。

