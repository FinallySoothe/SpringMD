

# BeanFactory

## 1.定义

​	Spring的核心，所有的bean都是有BeanFactory类构建，其中定义了相当多的bean的方法，依赖注入，控制反转都有它实现。



## 2 .实现

![](pic\image-20250723230323323.png)

### DefaultListableBeanFactory

​	是BeanFactory的一个实现类，使用具体的实现类来加载bean，下面为一个加载bean的示例：

​	首先创建一个Config类，里面配置两个类为bean1、bean2，然后再bean2注入bean1。

```java
@Configuration
static class Config{
    @Bean
    public Bean1 bean1(){
        return new Bean1();
    }
    @Bean
    public Bean2 bean2(){
        return new Bean2();
    }
}
static class Bean1{
}
static class Bean2{
    @Autowired
    private Bean1 bean1;

    public Bean1 getBean1() {
        return bean1;
    }
}
```

下面为获取config对象的过程

```java
public static void main(String[] args) {
    //创建beanFactory实现类
    DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
    //创建bean描述类
    AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).getBeanDefinition();
    beanFactory.registerBeanDefinition("config",beanDefinition);
    //查看此时beanfactory中的所有bean描述
    for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
        System.out.println(beanDefinitionName);
}
```

运行结果如下：

![](pic\7.png)

可以看到在beanfactory中只有config，并没有bean1和bean2，这是因为解析注解并不是由beanfactory所实现的，而是由他的后处理器去解决的，我们尝试在beanfactory中加入后处理器

```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).getBeanDefinition();
beanFactory.registerBeanDefinition("config",beanDefinition);

AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).forEach((s, beanFactoryPostProcessor) -> {
    System.out.println(s+":"+beanFactoryPostProcessor);
    beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
});
for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
    System.out.println(beanDefinitionName);
}
```

这样将beanfactory后处理器加入就可以解析config的相关注解了，运行结果如下

![](pic\8.png)

但是仅仅是这样还不够，如果我在bean2中注入bean1，那么还是无法获取到bean1，具体代码如下

```java
@Configuration
static class Config{
    @Bean
    public Bean1 bean1(){
        return new Bean1();
    }
    @Bean
    public Bean2 bean2(){
        return new Bean2();
    }
}
static class Bean1{
}
static class Bean2{
    @Autowired
    private Bean1 bean1;

    public Bean1 getBean1() {
        return bean1;
    }
}
```

```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).getBeanDefinition();
beanFactory.registerBeanDefinition("config",beanDefinition);

AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
//beanFactory后处理器
beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).forEach((s, beanFactoryPostProcessor) -> {
    System.out.println(s+":"+beanFactoryPostProcessor);
    beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
});
for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
    System.out.println(beanDefinitionName);
}
Bean2 bean = beanFactory.getBean(Bean2.class);
System.out.println(bean);
System.out.println(bean.getBean1());
```

运行结果如下，@Autoware注解没有生效，没有从bean2中拿到bean1,就是说依赖注入没有生效。

![](pic\9.png)

此时我们加入beanfactory后处理器并不能生效，要解决依赖注入，则需要引入bean后处理器来解决。

具体实现代码如下：

```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).getBeanDefinition();
beanFactory.registerBeanDefinition("config",beanDefinition);

AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
//beanFactory后处理器
beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).forEach((s, beanFactoryPostProcessor) -> {
    System.out.println(s+":"+beanFactoryPostProcessor);
    beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
});
//bean后处理器
beanFactory.getBeansOfType(BeanPostProcessor.class).forEach((s, beanPostProcessor) -> {
    System.out.println(s+":"+beanPostProcessor);
    beanFactory.addBeanPostProcessor(beanPostProcessor);
});
for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
    System.out.println(beanDefinitionName);
}
Bean2 bean = beanFactory.getBean(Bean2.class);
System.out.println(bean);
System.out.println(bean.getBean1());
```

具体运行结果如下：

![](pic\10.png)

此时bean1就可以成功获取了。

注意：添加了两次后处理器，这两次是不一样的，其中第一次属于beanfactory后处理器，是给beanfactory提供创建bean的一些扩展功能，比如@Configuration注解，第二次添加的是bean后处理器，是给创建bean添加的一些扩展功能，比如@Autoware自动注入
