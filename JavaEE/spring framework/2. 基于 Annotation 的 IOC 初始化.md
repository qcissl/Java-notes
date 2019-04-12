# 基于 Annotation 的 IOC 初始化 

## Spring IOC 容器对于类级别的注解和类内部的注解分以下两种处理策略：
* 1)、类级别的注解：如@Component、@Repository、@Controller、@Service 以及 JavaEE6 的@ManagedBean 和@Named 注解，都是添加在类上面的类级别注解，Spring 容器根据注解的过滤规则扫描读取注解 Bean 定义类，并将其注册到 Spring IOC 容器中。
* 2)、类内部的注解：如@Autowire、@Value、@Resource 以及 EJB 和 WebService 相关的注解等，都是添加在类内部的字段或者方法上的类内部注解，SpringIOC 容器通过 Bean 后置注解处理器解析Bean 内部的注解。 

## 1. 定位 Bean 扫描路径 

```
	在Spring中管理注解Bean定义的容器有两个：AnnotationConfigApplicationContext 和
AnnotationConfigWebApplicationContex。
	这两个类是专门处理 Spring 注解方式配置的容器，直接依赖于注解作为容器配置信息来源的 IOC 容器。AnnotationConfigWebApplicationContext 是AnnotationConfigApplicationContext的Web版本，两者的用法以及对注解的处理方式几乎没有差别。
```

### 初始化AnnotationConfigApplicationContext

* 初始化一个读取注解的 Bean 定义读取器， 并将其设置到容器中 
* 初始化一个扫描指定类路径中注解 Bean 定义的扫描器， 并将其设置到容器中 
* 通过将涉及到的配置类或扫描路径传递给该构造函数， 以实现将相应配置类中的 Bean 自动注册到容器中
* 调用 `refresh()` 方法 

## 2. 读取 Annotation 元数据 

### 注册多个注解 Bean 定义类 (`@Configuration`类)

* 需要使用注解元数据解析器解析注解 Bean 中关于作用域的配置。 
* 使用 AnnotationConfigUtils 的 processCommonDefinitionAnnotations()方法处理注解 Bean 定义类中通用的注解。 
* 使用 AnnotationConfigUtils 的 applyScopedProxyMode()方法创建对于作用域的代理对象。 
* 通过 BeanDefinitionReaderUtils 向容器注册 Bean。 

### AnnotationScopeMetadataResolver 解析作用域元数据 

* 从注解 Bean 定义类的属性中查找属性为”Scope”的值， 即@Scope 注解的值 
* 获取@Scope 注解中的 proxyMode 属性值 
* 返回解析的作用域元信息对象 

### AnnotationConfigUtils 处理注解 Bean 定义类中的通用注解 

* 如果 Bean 定义中有@Lazy 注解， 则将该 BeanDefinition 属性设置为@lazy 注解的值 
* 如果 Bean 定义中有@Primary 注解， 则为该 Bean 设置为 autowiring 自动依赖注入装配的首选对象 
* 如果 Bean 定义中有@ DependsOn 注解， 则为该 Bean 设置所依赖的 Bean 名称 

### AnnotationConfigUtils 根据注解 Bean 定义类中配置的作用域为其应用相应的代理策略 

* 获取注解 Bean 定义类中@Scope 注解的 proxyMode 属性值 
* 如果配置的@Scope 注解的 proxyMode 属性值为 NO， 则不应用代理模式 
* 获取配置的@Scope 注解的 proxyMode 属性值为 TARGET_CLASS ，则使用CGLIB
* 获取配置的@Scope 注解的 proxyMode 属性值为 INTERFACES ，则使用JDK动态代理

### BeanDefinitionReaderUtils 向容器注册 Bean 

* BeanDefinitionReaderUtils 主要是校验 BeanDefinition 信息，然后将 Bean 添加到容器中一个管理
  BeanDefinition 的 HashMap 中。 

## 3. 扫描指定包并解析为 BeanDefinition 

* 初始化`ClassPathBeanDefinitionScanner` 
  * 为容器创建一个类路径 BeanDefinition 扫描器， 并指定是否使用默认的扫描过滤规则。即 Spring 默认扫描配置： @Component、 @Repository、 @Service、 @Controller注解的 Bean， 同时也支持 JavaEE6 的@ManagedBean 和 JSR-330 的@Named 注解 
* 遍历扫描所有给定的包，获取符合条件的 Bean 定义 
  * 初始化`ClassPathScanningCandidateComponentProvider` 
    * 保存过滤规则要包含的注解， 即 Spring 默认的@Component、 @Repository、 @Service、@Controller 注解的 Bean， 以及 JavaEE6 的@ManagedBean 和 JSR-330 的@Named 注解 
    * 保存过滤规则要排除的注解 
    * 向容器注册过滤规则 
  * 为指定资源获取元数据读取器， 元信息读取器通过汇编(ASM)读取Bean 定义元信息  
* 遍历扫描到的 Bean 
  * 获取 Bean 定义类中@Scope 注解的值， 为 Bean 设置注解配置的作用域  
  * 设置 Bean 的自动依赖注入装配属性等 
  * 如果扫描到的 Bean 是 Spring 的注解 Bean， 则处理其通用的 Spring 注解 
  * 根据注解中配置的作用域， 为 Bean 应用相应的代理模式 
  * 向容器注册扫描到的 Bean 

## 4. AnnotationConfigWebApplicationContext 注册注解 BeanDefinition 
* 为容器设置注解 Bean 定义读取器 `AnnotatedBeanDefinitionReader` 
* 为容器设置类路径 Bean 定义扫描器 `ClassPathBeanDefinitionScanner` 
* 为注解 Bean 定义读取器和类路径扫描器设置 Bean 名称生成器 
* 为注解 Bean 定义读取器和类路径扫描器设置作用域元信息解析器 
* 获取容器定义的 Bean 定义资源路径 并封装BeanDefinition 

