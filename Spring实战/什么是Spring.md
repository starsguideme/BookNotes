1. 在SpringBoot中封装了xml的语句，将其使用了注解的方式来将各类方法注入到IoC的池子里，其使用了@Bean注解的方式 注入到IoC中
2. bean注解的底层使用了DI(自动注入)的一种方式实现的，其可以将类变成Spring管理用到的东西。
3. 在spring上还提供了Web框架，web框架主要用于处理各种前端的东西并为其设置上一些安全措施，如SpringSecurity和SpringMVC.
4. 在spring中还可以通过@Configuartion注解 告诉Spring框架 这个类是配置类，并为应用上下文提供了@bean注解将该类注入到Spring中，其主要用于处理一些上下文事件
5. Spring中的自动配置启发于自动装配和组件扫描，后随着发展便由此诞生了SpringBoot，这个框架因为强大的自动配置功能可以有效的减少开发时间
6. 如何初始化spring 请参考[[初始化Spring应用]].
7. 在spring有一个特殊的框架 他是[[什么是SpingBootDevTools]].
8. 在当前的企业中 经常使用了名为SpringBoot的框架层，这个框架层为开发减少了很多不必要的配置，还可以帮程序员们了解当前出错的地方
9. 在spring 还有一个方法是springDATA,他在定义驱动存储和检索数据的方法中使用了命名约定的方法 来要求前端和后端在调用方法的名字需相同 否则 spring会无法查到对应的数据，然后springDATA还可以处理多种不同类型的数据库，如关系型的，非关系型的，图数据库等等
10.  Spring中还有[[Spring Security]] [[Spring BATCH]], [[Spring Integration]] 和[[Spring Cloud]].这一套构成了当前社会上常见的主流框架