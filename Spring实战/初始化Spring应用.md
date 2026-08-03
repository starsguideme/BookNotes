### @SpringBootApplication 到底是个啥

它是一个**组合注解**，就是把三个常用的注解打包在一起了，省得你每次都写三个。

它里面装了这三样东西：

- **@SpringBootConfiguration**：告诉Spring：“这个类是配置类。”它的作用和@Configuration是一样的，只是换了个名字，专门给Spring Boot用。配置类的作用就是可以在里面用@Bean注解，手动往IoC容器里塞东西。
    
- **@EnableAutoConfiguration**：开启自动配置。这个是Spring Boot最重要的东西。它会根据你引入了什么依赖，自动帮你配置好对应的东西。比如你引入了spring-boot-starter-web，它就自动帮你配好Spring MVC、Tomcat这些，不用你手动搞。
    
- **@ComponentScan**：开启组件扫描。它会自动扫描启动类所在包以及所有子包，把带了@Component、@Service、@Controller、@Repository这些注解的类全部找出来，注册到IoC容器里。
    

---

### 启动类的参数

在正式的启动类中，往往会有两个参数：

- **第一个参数：Application.class**。这个告诉Spring Boot哪个是启动类，它就会从这个类所在的包开始扫描。
    
- **第二个参数：args**。这是命令行参数，你启动的时候可以传一些东西进去。比如 `java -jar xxx.jar --server.port=8081`，这个 `--server.port=8081` 就是命令行参数。在代码里可以用 `@Value("${server.port}")` 或者注入 `ApplicationArguments` 来拿到这些参数。
    

---

### Spring MVC 到底是什么

Spring MVC是Spring Web端的一个框架，用来处理前端的HTTP请求。

它的核心是 **DispatcherServlet**（前端控制器）。所有请求都会先到它这里，它来统一分发给对应的控制器。

**@Controller 和 @RestController**：

- **@Controller**：用来标识一个类是控制器，处理前端请求。如果要在方法上直接返回JSON数据，需要用@ResponseBody。
    
- **@RestController**：等于@Controller + @ResponseBody，所有方法的返回值都直接变成JSON返回给前端（目前前后端分离项目中更常用）。
    

---

### @Controller 和 @Service、@Mapper/@Repository 的关系

这几个注解，**本质上做的事情是一样的**：把类标记成一个Bean，交给IoC容器管理。

那为什么要有不同名字？

**为了语义分层。** 看到@Controller就知道这个是控制层，看到@Service就知道这个是业务层，看到@Mapper就知道这个是持久层。这样代码的可读性更强，维护的时候一目了然。

**他们之间是怎么配合的**：

- @Controller接收前端请求，调用@Service
    
- @Service处理业务逻辑，需要操作数据时调用@Mapper
    
- @Mapper操作数据库
    

---

### 数据流转的完整过程

**从前端到后端，再从后端回到前端的完整过程：**

1. 用户在前端点击一个按钮，前端发送一个HTTP请求（比如GET /user/1）。
    
2. 请求到达后端，**DispatcherServlet**（前端控制器）拿到请求路径 `/user/1`。
    
3. DispatcherServlet根据这个路径去**HandlerMapping**中查找，看哪个Controller的哪个方法可以处理这个请求。
    
4. 找到对应的Controller方法（比如 `@GetMapping("/user/{id}")`）。
    
5. **IoC容器**介入：Controller里的Service依赖是怎么来的？是通过@Autowired自动注入的。Spring从容器里找到对应的Service实例，注入给Controller。Controller调用Service的方法。
    
6. Service处理业务逻辑。如果需要数据库数据，Service里也通过@Autowired注入了Mapper，调用Mapper的方法。
    
7. Mapper查询数据库，返回数据给Service。
    
8. Service收到数据，进行封装和处理，返回给Controller。
    
9. Controller拿到结果，如果是@RestController，直接把Java对象序列化成JSON，返回给前端。
    
10. 前端拿到JSON数据，解析并渲染出来。