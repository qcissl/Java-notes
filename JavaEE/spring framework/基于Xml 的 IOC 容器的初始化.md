# 基于Xml 的 IOC 容器的初始化 

## 1、寻找入口 

​	还有像 AnnotationConfigApplicationContext 、 FileSystemXmlApplicationContext 、
XmlWebApplicationContext 等都继承自父容器 AbstractApplicationContext主要用到了装饰器模式
和策略模式，最终都是调用 **refresh()**方法。 

```java
public ClassPathXmlApplicationContext(
	String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```

## 2、获得配置路径

​	通过分析ClassPathXmlApplicationContext 的源代码可以知道 ,在创建ClassPathXmlApplicationContext 容器时，构造方法做以下两项重要工作：

* 首先，调用父类容器的构造方法(super(parent)方法)为容器设置好 Bean 资源加载器。
* 然 后 ,再调用父类AbstractRefreshableConfigApplicationContext 的setConfigLocations(configLocations)方法设置 Bean 配置信息的定位路径。 

```java
	public AbstractApplicationContext() {
		this.resourcePatternResolver = getResourcePatternResolver();
	}

	/**
	 * Create a new AbstractApplicationContext with the given parent context.
	 * @param parent the parent context
	 */
	public AbstractApplicationContext(@Nullable ApplicationContext parent) {
		this();
		setParent(parent);
	}

	protected ResourcePatternResolver getResourcePatternResolver() {
		return new PathMatchingResourcePatternResolver(this);
	}
```

​	在设置容器的资源加载器之后，接下来 ClassPathXmlApplicationContext 执行**setConfigLocations()**方法通过调用其父类 AbstractRefreshableConfigApplicationContext 的方法进行对 Bean 配置信息的定位 

## 3、开始启动

​	SpringIOC 容器对 Bean 配置资源的载入是从 refresh()函数开始的，refresh()是一个模板方法，规定了IOC 容 器 的 启 动 流 程 ， 有 些 逻 辑 要 交 给 其 子 类 去 实 现 。 它 对 Bean 配 置 资 源 进 行 载 入ClassPathXmlApplicationContext 通过调用其父类 AbstractApplicationContext 的 refresh()函数启动整个 IOC 容器对 Bean 定义的载入过程， 现在我们来详细看看 refresh()中的逻辑处理： 

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        //1、 调用容器准备刷新的方法， 获取容器的当时时间， 同时给容器设置同步标识
        prepareRefresh();
        //2、 告诉子类启动 refreshBeanFactory()方法， Bean 定义资源文件的载入从
        //子类的 refreshBeanFactory()方法启动
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        //3、 为 BeanFactory 配置容器特性， 例如类加载器、 事件处理器等
        prepareBeanFactory(beanFactory);
        try {
            //4、 为容器的某些子类指定特殊的 BeanPost 事件处理器
            postProcessBeanFactory(beanFactory);
            //5、 调用所有注册的 BeanFactoryPostProcessor 的 Bean
            invokeBeanFactoryPostProcessors(beanFactory);
            //6、 为 BeanFactory 注册 BeanPost 事件处理器.
            //BeanPostProcessor 是 Bean 后置处理器， 用于监听容器触发的事件
            registerBeanPostProcessors(beanFactory);
            //7、 初始化信息源， 和国际化相关.
            initMessageSource();
            //8、 初始化容器事件传播器.
            initApplicationEventMulticaster();
            //9、 调用子类的某些特殊 Bean 初始化方法
            onRefresh();
            //10、 为事件传播器注册事件监听器.
            registerListeners();
            //11、 初始化所有剩余的单例 Bean
            finishBeanFactoryInitialization(beanFactory);
            //12、 初始化容器的生命周期事件处理器， 并发布容器的生命周期事件
            finishRefresh();
        } catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                "cancelling refresh attempt: " + ex);
            } 
            //13、 销毁已创建的 Bean
            destroyBeans();
            //14、 取消 refresh 操作， 重置容器的同步标识.
            cancelRefresh(ex);
            throw ex;
        } finally {
            //15、 重设公共缓存
            resetCommonCaches();
        }
    }
}
```

## 4、创建容器

​	obtainFreshBeanFactory()方法调用子类容器的 refreshBeanFactory()方法，启动容器载入 Bean 配置信息的过程 

