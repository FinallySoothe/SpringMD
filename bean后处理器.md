

# bean后处理器

​	spring处理bean很多都是扩展功能，由bean后处理器来实现

下面为一段代码展示没有后处理器的结果

```java
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean("bean1", Bean1.class);
context.registerBean("bean2", Bean2.class);
context.registerBean("bean3", Bean3.class);
context.registerBean("bean4", Bean4.class);
context.refresh();
context.close();
```

我们定义了一个容器，并注册了四个bean，下面为四个bean的代码

前两个bean没有做操作，只是作为第三个bean的引用，第三个bean中使用了@Autoware，@Resource，@Value,@PostConstruct，@PreDestroy五个注解，

第四个bean使用了SpringBoot中的@ConfigurationProperties

```java
@Component
public class Bean1 {
}
```

```java
@Component
public class Bean2 {
}
```

```java
package com.testcenter.a02;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.annotation.Resource;

@Component
public class Bean3 {
    private Bean1 bean1;
    private Bean2 bean2;
    public static final Logger log = LoggerFactory.getLogger(Bean3.class);
    public Bean3 bean3(){
        log.info("构造阶段");
        return new Bean3();
    }
    @Autowired
    public void setBean1(Bean1 bean1){
        log.info("@Autoware生效,{}",bean1);
        this.bean1=bean1;
    }
    @Resource
    public void setBean2(Bean2 bean2){
        log.info("@Resource生效,{}",bean2);
        this.bean2=bean2;
    }
    @Autowired
    private void testAnno(@Value("${JAVA_HOME}") String home){
        log.info("成功解析@Autoware,{}",home);
    }
    @PostConstruct
    public void testPostConstruct(){
        log.info("成功解析@PostConstruct");
    }
    @PreDestroy
    public void testPreDestroy(){
        log.info("成功解析@PreDestory");
    }
}

```

```java
@ConfigurationProperties(prefix = "java")
public class Bean4 {
    private String home;
    private String version;

    @Override
    public String toString() {
        return "Bean4{" +
                "home='" + home + '\'' +
                ", version='" + version + '\'' +
                '}';
    }

    public String getHome() {
        return home;
    }

    public void setHome(String home) {
        this.home = home;
    }

    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

    public Bean4() {
    }

    public Bean4(String home, String version) {
        this.home = home;
        this.version = version;
    }
}
```

下面是运行结果

![](.\pic\13.png)

可以看到，这些注解并没有去解析，因此解析这些注解是由bean后处理器解决的

## 1.AutowiredAnnotationBeanPostProcessor

这个注解主要是解析@Autoware注解

当我们加上这个后处理器后，代码如下

```java
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean("bean1", Bean1.class);
context.registerBean("bean2", Bean2.class);
context.registerBean("bean3", Bean3.class);
context.registerBean("bean4", Bean4.class);
context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
context.refresh();
context.close();
```

下面为运行结果，可以看到，bean3中使用@Autoware注解的部分被注入了进来

![](.\pic\14.png)

##  2.CommonAnnotationBeanPostProcessor

这个可以用来解析@Resource,@Value,@Autowired,@PostConstruct注解

代码及运行结果如下

```java
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean("bean1", Bean1.class);
context.registerBean("bean2", Bean2.class);
context.registerBean("bean3", Bean3.class);
context.registerBean("bean4", Bean4.class);
context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
context.registerBean(CommonAnnotationBeanPostProcessor.class);
context.refresh();
context.close();
```

![](.\pic\15.png)

## 3.ConfigurationPropertiesBindingPostProcessor

用于解析@ConfigurationProperties注解

代码及运行结果如下

```java
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean("bean1", Bean1.class);
context.registerBean("bean2", Bean2.class);
context.registerBean("bean3", Bean3.class);
context.registerBean("bean4", Bean4.class);
context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
context.registerBean(CommonAnnotationBeanPostProcessor.class);
ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());
context.refresh();
System.out.println(context.getBean(Bean4.class));
context.close();
```

![](.\pic\16.png)
