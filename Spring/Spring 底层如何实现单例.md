参考 https://www.cnblogs.com/nickup/p/9800120.html

单例：

```xml
<bean id="hi" class="com.test.Hi" init-method="init" scope="singleton">
```

多例：

```xml
<bean id="hi" class="com.test.Hi" init-method="init" scope="prototype">
```

单例模式的实现源码：

```java
public abstract class AbstractBeanFactory implements ConfigurableBeanFactory{  
       /** 
        * 充当了Bean实例的缓存，实现方式和单例注册表相同 
        */  
       private final Map singletonCache =new HashMap();  
       public Object getBean(String name)throws BeansException{  
           return getBean(name,null,null);  
       }  
    ...  
       public Object getBean(String name,Class requiredType,Object[] args)throws BeansException{  
          //对传入的Bean name稍做处理，防止传入的Bean name名有非法字符(或则做转码)  
          String beanName=transformedBeanName(name);  
          Object bean=null;  
          //手工检测单例注册表  
          Object sharedInstance=null;  
          //使用了代码锁定同步块，原理和同步方法相似，但是这种写法效率更高  
          synchronized(this.singletonCache){  
             sharedInstance=this.singletonCache.get(beanName);  
           }  
          if(sharedInstance!=null){  
             ...  
             //返回合适的缓存Bean实例  
             bean=getObjectForSharedInstance(name,sharedInstance);  
          }else{  
            ...  
            //取得Bean的定义  
            RootBeanDefinition mergedBeanDefinition=getMergedBeanDefinition(beanName,false);  
             ...  
            //根据Bean定义判断，此判断依据通常来自于组件配置文件的单例属性开关  
            //<bean id="date" class="java.util.Date" scope="singleton"/>  
            //如果是单例，做如下处理  
            if(mergedBeanDefinition.isSingleton()){  
               synchronized(this.singletonCache){  
                //再次检测单例注册表
                 sharedInstance=this.singletonCache.get(beanName);  
                 if(sharedInstance==null){  
                    ...  
                   try {  
                      //真正创建Bean实例  
                      sharedInstance=createBean(beanName,mergedBeanDefinition,args);  
                      //向单例注册表注册Bean实例  
                       addSingleton(beanName,sharedInstance);  
                   }catch (Exception ex) {  
                      ...  
                   }finally{  
                      ...  
                  }  
                 }  
               }  
              bean=getObjectForSharedInstance(name,sharedInstance);  
            }  
           //如果是非单例，即prototpye，每次都要新创建一个Bean实例  
           //<bean id="date" class="java.util.Date" scope="prototype"/>  
           else{  
              bean=createBean(beanName,mergedBeanDefinition,args);  
           }  
    }  
    ...  
       return bean;  
    }  
    }
```

