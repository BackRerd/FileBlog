---
title: springboot学习笔记
date: 2024-11-16 11:09:05
tags: ["java","spring"]
---

# 热部署

spring-boot-devtools组件可以实现热部署。

# 控制器

- Spring Boot提供了@Controller和@RestController两种注解来标识此类负责接收和处理HTTP请求
- 如果请求的是页面和数据，使用@Controller注解即可；如果只是请求数据，则可以使用@RestController注解。
- @RequestController对方法进行定义，参数value填写访问的地址，method填写访问的方式比如get/post
- @RequestBody要求这个参数必须接收

