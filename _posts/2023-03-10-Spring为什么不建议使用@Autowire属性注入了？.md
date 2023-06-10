---
title: Spring为什么不建议使用@Autowired属性注入了?
categories:
- java
- spring
tags:
- 我有一个问题
---

从spring4.0 开始，官方就不推荐@Autowired的使用在字段上。如果使用了在IDEA上会出现**Field injection is not recommended**的警告。目前，Spring官方推荐的注入方式是构造器注入。
<!--more-->

#### 1.使用@Autowire会出现的问题（暂时找到一个）

> ![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-03-10/Spring%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%40Autowire%E5%B1%9E%E6%80%A7%E6%B3%A8%E5%85%A5%E4%BA%86/Snipaste_2023-03-12_15-16-11.png)
>
> ​    因为在spring创建testService时会先执行构造函数进行testService对象的创建，在之后会进行populateBean来填充testService的属性，在填充属性的时候会解析@Autowired注解并通过AutowiredAnnotationBeanPostProcessor这个类进行处理。
>
> ​    下图为bean实例化的流程
>
> ![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-03-10/Spring%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%40Autowire%E5%B1%9E%E6%80%A7%E6%B3%A8%E5%85%A5%E4%BA%86/6a07af444a1b8f36e9aa2d493834631d.png)
>
> ​    所以在这样使用的时候会出现null的异常。

## spring推荐使用构造器注入

> ```java
>  private final CityDao cityDao;
> 
>     public CityServiceImpl(CityDao cityDao) {
>         this.cityDao = cityDao;
>     }
> ```

参考：

> [https://blog.csdn.net/weixin_46228112/article/details/124139921](https://blog.csdn.net/weixin_46228112/article/details/124139921)
>
> [https://blog.csdn.net/BASK2311/article/details/127528343](https://blog.csdn.net/BASK2311/article/details/127528343)
>
> [https://zhuanlan.zhihu.com/p/83492830](https://zhuanlan.zhihu.com/p/83492830)