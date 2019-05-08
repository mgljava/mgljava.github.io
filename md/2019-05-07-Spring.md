# Spring
## Spring Boot - Hibernate-Validator进行参数校验
1. 引入依赖
  ```
      <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>5.3.1.Final</version>
      </dependency>
  ```
2. 校验原生数据类型（基本数据类型）
  ```
  @Validated
  @Controller
  @RequestMapping(value = "/users")
  public class UserController {
  
    /**
     * 原生数据类型校验
     */
    @GetMapping("/primitiveTypesCheck")
    @ResponseBody
    public String check(@Min(value = 18, message = "年龄必须大于18岁") @RequestParam int age,
        @Size(min = 6, max = 18, message = "名字必须在6到18位字符之间") @RequestParam String name) {
      System.out.println(age);
      System.out.println(name);
      return "Success";
    }
    
    /**
    * 异常处理器
    */
    @ExceptionHandler(value = {ConstraintViolationException.class})
    @ResponseBody
    public String handleResourceNotFoundException(ConstraintViolationException e) {
      Set<ConstraintViolation<?>> violations = e.getConstraintViolations();
      StringBuilder strBuilder = new StringBuilder();
  
      return violations.stream()
          .map(violation -> strBuilder.append(violation.getMessage()))
          .map(stringBuilder -> stringBuilder.append("\n"))
          .findFirst().get().toString();
    }
  }
  ```
3. 校验对象类型
  ```
  /**
  * User 对象
  */
  @Data
  public class User {
  
    @Min(value = 0, message = "用户ID必须大于0")
    private Integer id;
  
    @NotBlank(message = "名字不能为空")
    @Size(min = 2, max = 30, message = "名字长度必须在2到30之中")
    private String name;
  
    @NotBlank(message = "用户名不能为空")
    @Size(max = 10, message = "用户名长度不能超过10个字符")
    private String username;
  }
  
  
  @Validated
  @Controller
  @RequestMapping(value = "/users")
  public class UserController {
    /**
    * 对象类型校验
    */
    @GetMapping("/objectTypeCheck")
    @ResponseBody
    public String user(@Valid User user, BindingResult bindingResult) {
      System.out.println(user.getName());
      return bindingResult.hasErrors() ? bindingResult.getFieldError().getDefaultMessage()
          : "Success";
    }
  }
  ```
## Spring AOP
1. 概念：面向AOP（面向切面），OOP属于一种横向扩展，AOP是一种纵向扩展。AOP依托于OOP，进一步将系统级别的代码抽象出来，进行纵向排列，实现低耦合。
2. AOP的家庭成员
   * PointCut（切点）：即在哪个地方进行切入,它可以指定某一个点，也可以指定多个点。比如类A的methord函数，当然一般的AOP与语言（AOL）会采用多用方式来定义PointCut,比如说利用正则表达式，可以同时指定多个类的多个函数。
   * Advice（通知，执行的代码）：在切入点干什么事情，指定在PointCut地方做什么事情（增强），打日志、执行缓存、处理异常等等。
   * Advisor/Aspect：PointCut + Advice 形成了切面Aspect，这个概念本身即代表切面的所有元素。但到这一地步并不是完整的，因为还不知道如何将切面植入到代码中，解决此问题的技术就是PROXY
   * Proxy：Proxy 即代理，其不能算做AOP的家庭成员，更相当于一个管理部门，它管理 了AOP的如何融入OOP。之所以将其放在这里，是因为Aspect虽然是面向切面核心思想的重要组成部分，但其思想的践行者却是Proxy,也是实现AOP的难点与核心据在。
3. AOP的技术实现Proxy：AOP仅仅是一种思想，那为了让这种思想发光，必然脱离语言本身的技术支持，Java在实现该技术时就是采用的代理Proxy,那我们就去了解一下，如何通过代理实现面向切面
   1. 静态代理：