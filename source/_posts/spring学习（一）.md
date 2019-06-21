---
title: spring学习（一）
date: 2016-07-10 10:38:29
tags: 开发笔记
categories:
- spring
---
# 一、注解解释
1. @controller 控制器（注入服务）
2. @service 服务（注入dao）
3. @repository dao（实现dao访问）
4. @component （把普通pojo实例化到spring容器中，相当于配置文件中的<bean id="" class=""/>）

  @Component,@Service,@Controller,@Repository注解的类，并把这些类纳入进spring容器中管理。 
下面写这个是引入component的扫描组件 
```
<context:component-scan base-package=”com.mmnc”>   
```

其中base-package为需要扫描的包（含所有子包） 
- @Service用于标注业务层组件 
- @Controller用于标注控制层组件(如struts中的action) 
- @Repository用于标注数据访问组件，即DAO组件. 
- @Component泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。 

