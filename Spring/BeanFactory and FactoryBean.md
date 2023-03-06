# BeanFactory
说明： spring容器的底层接口，可以根据bean定义的信息，返回对应的实例对象，支持整个sring的生命周期流程。
提供的方法：


# FactoryBean
说明： 就是一个简单的对象工程，实现了此接口的方法在整个bean周期中，可以使用自定义的方式来创建对象，而不需要进行默认的bean流程的创建。
提供的方法


使用
```java
public class MyFactoryBean implements FactoryBean<Person> {
    
    @Override
    public Person getObject() throws Exception {
        Person person = new Person();
        person.setName("张三");
        person.setAddress("上海市上海");
        return person;
    }

    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}

// 当我们使用时，如下就可以获取到对应的Person对象
public static void main(String[] args) {
        ApplicationContext bf = new ClassPathXmlApplicationContext("beanFactory.xml");
        // 直接就可过去到对应的Person对象
        Person person = (Person) bf.getBean("myFactoryBean");
}

```
这时候细心的同学不知道有没有发现，那我需要怎么获取对应的myFactoryBean对象呢？
这时我们看源码在BeanFactory中定义了一个


也就是如果我们需要获取FactoryBean的实例而不是对应的getObject方法时，需要在getBean的前面加个&，也就是如下所示
```java
ApplicationContext bf = new ClassPathXmlApplicationContext("beanFactory.xml");
MyFactoryBean myFactoryBean = bf.getBean("&myFactoryBean", MyFactoryBean.class);
```

并且在BeanFactoryUtils.transformedBeanName()，方法中进行判断是否带了对应的&符号，如果带了就返回对应的工厂Bean实力，如果没有带则获取调用对应的getObject方法返回对应的具体bean实力（Person对象）

```java
// 判断是否是&开头
public static boolean isFactoryDereference(@Nullable String name) {
    return (name != null && name.startsWith(BeanFactory.FACTORY_BEAN_PREFIX));
  }
  
  protected Object getObjectForBeanInstance(
      Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

    // Don't let calling code try to dereference the factory if the bean isn't a factory.
    // 如果是&开头进行此方法获取对应的FactroyBean对象
    if (BeanFactoryUtils.isFactoryDereference(name)) {
      if (beanInstance instanceof NullBean) {
        return beanInstance;
      }
      if (!(beanInstance instanceof FactoryBean)) {
        throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
      }
      if (mbd != null) {
        mbd.isFactoryBean = true;
      }
      return beanInstance;
    }
    // 省略部分代码  
    if (object == null) {
      // Return bean instance from factory.
      FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
      // Caches object obtained from FactoryBean if it is a singleton.
      if (mbd == null && containsBeanDefinition(beanName)) {
        mbd = getMergedLocalBeanDefinition(beanName);
      }
      boolean synthetic = (mbd != null && mbd.isSynthetic());
      // 从Factory的getObject获取实例
      object = getObjectFromFactoryBean(factory, beanName, !synthetic);
    }
}

private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
      throws BeanCreationException {

    Object object;
    try {
      if (System.getSecurityManager() != null) {
        AccessControlContext acc = getAccessControlContext();
        try {
          object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
        }
        catch (PrivilegedActionException pae) {
          throw pae.getException();
        }
      }
      else {
      // 对应的Factory的getObject方法
        object = factory.getObject();
      }
    }
  }
```
# BeanFactory和FactoryBean的关系
我们从FactoryBean的源码注释中，就能看出一些说明，实现了FactoryBean的bean不单单是一个bean的实例，我们可以通过其中的getObject()方法，得到我们自定义的对象。


简单总结：FactoryBean只是BeanFactory在bean的周期中的一个条件分支的处理，可以让我们自定义实现对象的创建，而不需要完成整个复杂的Bean的创建过程。