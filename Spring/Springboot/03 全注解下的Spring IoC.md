IoC（Inversion of Control，IoC）是一种通过描述来生成或者获取对象的技术。

IoC容器需要具备两个基本的功能：

1、通过描述管理Bean，包括发布和获取Bean

2、通过描述完成Bean之间的依赖关系

# 1 IoC容器简介

Spring IoC容器是一个管理Bean的容器，在Spring的定义中，所有的IoC容器都需要实现接口BeanFactory，它是一个顶级容器接口。

```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface BeanFactory {
    // 前缀
    String FACTORY_BEAN_PREFIX = "&";

    // 多个getBean方法
    Object getBean(String var1) throws BeansException;

    <T> T getBean(String var1, @Nullable Class<T> var2) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;

    // 是否包含Bean
    boolean containsBean(String var1);

    // Bean是否单例。默认情况下，Bean都是以单例存在的。
    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    // 是否原型
    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    // 是否类型匹配
    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, @Nullable Class<?> var2) throws NoSuchBeanDefinitionException;

    // 获取Bean的类型
    @Nullable
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;

    // 获取Bean的别名
    String[] getAliases(String var1);
}
```





# 2 装配你的Bean



# 3 依赖注入





# 4 生命周期





# 5 使用属性文件



# 6 条件装配Bean



# 7 Bean的作用域



# 8 使用@Profile



# 9 引入XML配置Bean





# 10 使用Spring EL

