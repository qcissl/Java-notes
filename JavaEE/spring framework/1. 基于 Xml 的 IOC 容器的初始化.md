# 基于Xml 的 IOC 容器的初始化

## 1. 寻找入口

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

## 2. 获取配置路径

```java
setConfigLocations(configLocations);
```

* 初始化资源解析器
  * `PathMatchingResourcePatternResolver`  和  `AbstractApplicationContext`  共同继承 `ResourceLoader`  并持有 `AbstractApplicationContext`  对象，属于装饰者模式，增强`AbstractApplicationContext`  获取资源的能力  

## 3. 开始启动（refresh方法）

```java
//1、 调用容器准备刷新的方法， 获取容器的当时时间， 同时给容器设置同步标识
prepareRefresh();
```

## 4. 创建容器

* `obtainFreshBeanFactory()` 
  * `refreshBeanFactory ()`
    * 如果已经有容器， 销毁容器中的 bean， 关闭容器 
    * 创建 IOC 容器 
    * 对 IOC 容器进行定制化， 如设置启动参数， 开启注解的自动装配等 
    * 调用`loadBeanDefinitions` 方法， 具体的实现调用子类容器 

## 5. 创建并设置Bean读取器

* `loadBeanDefinitions(beanFactory);`
  * 创建 `XmlBeanDefinitionReader`， 即创建 Bean 读取器， 并通过回调设置到容器中去， 容器使用该读取器读取 Bean 配置资源 
  * 为 Bean 读取器设置 Spring 资源加载器 
  * Bean 读取器加载 Bean 配置资源 

## 6. 解析配置文件，并将其封装成resource

* `ResourcePatternResolver`  通过资源解析器将locations 转换成 resources 

## 7. 开始读取配置内容 

* 将读入的 XML 资源进行特殊编码处理 
* 将XML文件 转为输入流
* 将 XML 文件转换为 DOM 对象， 解析过程由 `documentLoader` 实现 

## 8. 准备文档对象 

* 创建文件解析器工厂 
* 创建文档解析器 
* 解析 Spring 的 Bean 配置资源 

## 9. 分配解析策略 

* 得到 `BeanDefinitionDocumentReader` 来对 xml 格式的 `BeanDefinition` 解析 
* 具体的解析实现过程有实现类 `DefaultBeanDefinitionDocumentReader` 完成 
  * 具体的解析过程由 `BeanDefinitionParserDelegate` 实现

## 10.载入\<bean\>元素 

* 解析 Bean 配置信息中的<Bean>元素， 这个方法中主要处理<Bean>元素的 id， name 和别名属性 
  * 如果\<Bean\>元素中没有配置 id 属性时， 将别名中的第一个值赋值给 `beanName` 
  * 检查\<Bean\>元素所配置的 id 或者 name 的唯一性 

* 详细对\<Bean\>元素中配置的 Bean 定义其他属性进行解析 
  * 这里只读取\<Bean>元素中配置的 class 名字 
  * 如果\<Bean>元素中配置了 parent 属性， 则获取 parent 属性的值 
  * 对\<Bean>元素的 meta(元信息)属性解析 
  * 对\<Bean>元素的 lookup-Method 属性解析 
  * 对\<Bean>元素的 replaced-Method 属性解析 
  * 解析\<Bean>元素的构造方法设置 
  * 解析\<Bean>元素的<property>设置 
  * 解析\<Bean>元素的 qualifier 属性 
  * 为当前解析的 Bean 设置所需的资源和依赖对象 

## 11. 载入\<property>元素 

* 获取\<property>元素的名字 
* 如果一个 Bean 中已经有同名的 property 存在， 则不进行解析， 直接返回。 
* 解析获取 property 的值 
  * 1、ref 被封装为指向依赖对象一个引用。
  * 2、value 配置都会封装成一个字符串类型的对象。
  * 3、ref 和 value 都通过`“解析的数据类型属性值.setSource(extractSource(ele));”`方法将属性值/引用与所引用的属性关联起来。 
* 根据 property 的名字和值创建 property 实例 
* 解析\<property>元素中的属性 

## 12. 载入\<property>的子元素 

* 如果\<property>没有使用 Spring 默认的命名空间， 则使用用户自定义的规则解析内嵌元素 
* 如果子元素是 bean， 则使用解析\<Bean>元素的方法解析 
* 如果子元素是 ref， ref 中只能有以下 3 个属性： bean、 local、 parent 
* 如果子元素是\<value>， 使用解析 value 元素的方法解析 
* 如果子元素是集合， 使用解析相应集合子元素的方法解析 

## 13. 载入\<list>的子元素 

* 获取\<list>元素中的 value-type 属性， 即获取集合元素的数据类型 
* 获取\<list>集合元素中的所有子节点 
* Spring 中将 List 封装为 `ManagedList` 
* 具体解析\<list>集合元素， \<array>、 \<list>和\<set>都使用该方法解析 
  * 遍历所有节点
  * 将解析的元素加入集合中， 递归调用下一个子元素 

## 14. 向容器注册`BeanDefinition` 

* 获取解析的 `BeanDefinition` 的名称 
* 向 IOC 容器注册 `BeanDefinition` 
  * 校验解析的 `BeanDefiniton` 
  * 向IOC容器中注册 `BeanDefinition` 
* 如果解析的 `BeanDefinition` 有别名， 向容器为其注册别名 