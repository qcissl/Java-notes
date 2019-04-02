# BeanDefinitionReader 

​	Bean 的解析过程非常复杂，功能被分的很细，因为这里需要被扩展的地方很多，必须保证有足够的灵活性，以应对可能的变化。Bean 的解析主要就是对 Spring 配置文件的解析。这个解析过程主要通过BeanDefintionReader 来完成，最后看看 Spring 中 BeanDefintionReader 的类结构图： 

![1554215430016](assets/1554215430016.png)