1. @`RequiredArgsConstructor` 是 **Lombok** 库提供的注解。

**它的作用是：自动生成一个包含 `final` 字段和 `@NonNull` 标记字段的构造方法。**

你只需要在类上加这个注解，Lombok 会在编译时帮你生成构造器，你不用手动写。代码看起来简洁，生成的字节码里仍然有完整的构造方法。
final指的是 创建之后不可修改 @NonNull表示不可为空
2. @Data 这个注解已经自动生成了@getter和@setter方法，@equals，@Hash Code()方法等
3. Lombok中不仅有@Data等注解 还有@Slf4j等日志方法 所以在SpringBoot中也是一个不可或缺的框架
4. @RequestMapping("/路径") 如果类的最上方，则意味着这是所有请求的前缀路径
5. SpringMvc的请求映射注解
         注解                                                        描述
        @RequestMapping                                   通用的请求处理
        @GetMapping                                          处理HTTP GET请求
        @PutMapping                                          处理HTTP PUT请求
        @PostMapping                                         处理HTTP  POST请求
        @DeleteMapping                                      处理HTTP DELETE 请求
        @PatchMapping                                        处理HTTP PATCH请求
6. 如果在包类上添加了一个@RequestMapping+路径 再在方法上加一个路径 则 前端的请求路径一个是 方法+@RequestMapping的路径+请求方法的路径 一般以“/”隔开，请求方法 就是以上的注解
7. @Size注解，这个注解的主要含义是提示某些东西的范围 如加上@Size的min=5，message="Name must be at least 5 characters long") 的意义是 最小值是5，且名字的长度应该＞5个字符
8. `@NotBlank` 是 Spring 提供的一个**数据校验注解**，专门用在 Controller 方法中接收参数的字段上，一般与@vaild注解连用 可有效的减少所需的开发时间。

**作用**：检查字符串不能为 `null`，也不能是空字符串 `""`，更不能全是空格，如果再其小括号内加入了一些字符串数据则意味着当请求不通过的时候 他会返回字符串中的数据。
9. **@Valid 注解的作用**

`@Valid` 是 Spring MVC 中用于**触发参数校验**的注解。它的核心作用是告诉 Spring：“在调用这个方法之前，先对这个参数对象做一次校验。”

**执行时机**：校验发生在请求数据绑定到 Java 对象之后、Controller 方法体执行**之前**。如果校验不通过，方法体根本不会被执行。

**校验流程**：

1. Controller 方法参数前加上 `@Valid` 注解
    
2. Spring 检查该参数对象中所有带校验注解（如 `@NotBlank`、`@NotNull`）的字段
    
3. 校验结果有两种：
    
    - **校验通过**：请求正常进入 Controller 方法
        
    - **校验失败**：Spring 抛出 `MethodArgumentNotValidException`，请求被拦截，通常返回 400 状态码
        

**与 Errors 对象的关系**（经典做法）：  
校验失败的细节可以被捕获到 `Errors` 对象中。可以通过 `errors.hasErrors()` 判断是否有校验错误，如果有则不再执行业务逻辑。不过现在更主流的做法是配合全局异常处理器（`@ControllerAdvice`）统一处理校验异常，避免在每个方法里重复写 `if-else`。

**与数据重复的关系**：  
`@Valid` 只做**格式校验**（如“用户名不能为空”），不涉及数据库操作，因此**不会导致数据重复**。数据重复问题通常出现在业务层——比如用户快速连续点击提交，多个请求都通过了校验，此时需要依靠**幂等性设计**（如数据库唯一约束、Redis 防重标记）来兜底。校验是入口防线，幂等性是业务防线，两者各司其职。