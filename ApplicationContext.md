# ApplicationContext

# 1.定义

​	Spring中的一个容器抽象类，用于管理Spring中的bean，里面有很多的实现类，可以从该容器中拿到bean，SpringBoot中的主方法SpringApplication.run(TestCenterApplication.class, args)的返回值就是ConfigurableApplicationContext，这是ApplicationContext的一个实现类。但ApplicationContext中的方法主要是由BeanFactory所实现。

![image-20250723225434713](pic\image-20250723225434713.png)

## 2.功能

​	主要有四个实现类实现其相应功能

### MessageSource

主要提供国际化、翻译功能

![image-20250723234531386](pic\11)

![image-20250723234552520](pic\12.png)

可以通过获取ApplicationContext对象，使用getMessage方法实现国际化，只要设置不同的语言，就会从文件中找到不同翻译的配置文件进行翻译

### ResourcePatternResolver

提供通配符，匹配资源（磁盘或类路径）功能，主要提供getResources方法获取资源

![](pic\1.png)

![](pic\2.png)

### ApplicationEventPublisher

主要用于发布事件

需要定义一个事件类型，一个发布者，一个接收者

![](pic\4.png)



![](pic\5.png)

![](pic\6.png)

### EnvironmentCapable

读取环境信息，环境变量、yaml等等，提供getEnvironment方法获取环境信息，下图获取了环境变量java_home以及配置文件信息server.port

![](pic\3.png)
