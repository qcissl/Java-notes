# BeanFactory

​	这个接口由持有许多bean定义的对象实现，每个bean定义都由一个字符串名称惟一标识。根据bean定义，工厂将返回所包含对象的独立实例(原型设计模式)，或者单个共享实例(相对于单例设计模式，**单例设计模式中的实例是工厂范围内的单例**)。返回哪种类型的实例取决于bean工厂配置:API是相同的。自Spring 2.0以来，根据具体的应用程序上下文(例如，web环境中的“请求”和“会话”范围)。

​	请注意，通常依靠依赖项注入(“push”配置)通过setter或构造函数配置应用程序对象比使用任何形式的“pull”配置(如BeanFactory查找)更好。Spring的依赖注入功能是使用这个BeanFactory接口及其子接口实现的。

​	通常，BeanFactory将加载存储在配置源(如XML文档)中的bean定义，并使用{@code org.springframework.bean}包来配置bean。但是，实现可以简单地返回它在Java代码中直接创建的Java对象。定义的存储方式没有限制:LDAP、RDBMS、XML、属性文件等等。鼓励实现支持bean之间的引用(依赖项注入)。

​	与{@link ListableBeanFactory}中的方法相反，该接口中的所有操作也将检查父工厂是否为{@link hierarchy icalbeanfactory}。如果在这个工厂实例中没有找到bean，则会询问直接的父工厂。这个工厂实例中的bean应该覆盖任何父工厂中同名的bean。

## BeanFactory实现应该尽可能支持标准Bean生命周期接口。

### 完整的初始化方法及其标准顺序是：

1. `BeanNameAware` 
2. `BeanClassLoaderAware`
3. `BeanFactoryAware`
4. `EnvironmentAware`
5. `EmbeddedValueResolverAware  `  解析配置文件
6. `ResourceLoaderAware`  
7. `ApplicationEventPublisherAware`
8. `MessageSourceAware`
9. `ApplicationContextAware`
10. `ServletContextAware`
11. `BeanPostProcessor#postProcessBeforeInitialization`
12. `InitializingBean#afterPropertiesSet`
13. `a custom init-method definition`
14. `BeanPostProcessor#postProcessAfterInitialization`

### 在关闭bean工厂时，应用以下生命周期方法：

1. `DestructionAwareBeanPostProcessor#postProcessBeforeDestruction`
2. `DisposableBean#destroy`
3. `a custom destroy-method definition`