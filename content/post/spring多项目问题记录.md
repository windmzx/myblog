---
title: spring多项目问题记录
url: 157.html
id: 157
categories:
  - 计算机杂论
date: 2018-05-30 16:21:58
tags:
  - spring 
---

# jpa问题

>Error creating bean with name 'entityManagerFactory' defined in class path resource \[org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class\]:
>Invocation of init method failed; nested exception is org.hibernate.AnnotationException:
>No identifier specified for entity: com.mzx.ntms.model.Student

从英语就能看出没有指定ID，当使用data jpa的时候在Entity上指定主键，使用javax.persistence.Id;的@id指定即可
<!--more-->
# 包结构

Field service in com.mzx.ntms.web.HomeController required a bean of type 'com.mzx.ntms.service.TestService' that could not be found.

无法使用@Autowired，启动的时候会找不到bean,明明已经配置了却提示找不到，联系使用xml配置单项目spring的时候，因为路径不对导致dispatherservlet无法扫描包的问题。可以考虑是包层次不对。

![](https://img.0xaa.top/Image.png)

如图当WebApplication和Controller在一个层次的时候，可以顺利的扫描到controller却不能扫描到service，导致报错无法注入bean。

如果controller和service在一个层次但都比application高的时候会导致请求404，即application不能扫描到controller。

总结如下：application必须处于包结构的最顶层这样@SpringBootApplication所集成的@ComponentScan才能顺利扫描包结构