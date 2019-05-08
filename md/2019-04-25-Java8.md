## Java8
### 零.Lambda表达式
1. 为什么需要Lambda表达式  
   1.在Java中，我们无法将函数作为参数传递给一个方法，也无法声明返回一个函数的方法。  
   2.在JS中，函数参数是一个函数，返回值是另一个函数是常见的，所以JS是一门非常典型的函数式语言  
2. 什么是Lambda表达式：维基百科：函数式编程（英语：functional programming）或称函数程序设计，又称泛函编程，是一种编程范型，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。函数编程语言最重要的基础是λ演算（lambda calculus）。而且λ演算的函数可以接受函数当作输入（引数）和输出（传出值）。比起命令式编程，函数式编程更加强调程序执行的结果而非执行的过程，倡导利用若干简单的执行单元让计算结果不断渐进，逐层推导复杂的运算，而不是设计一个复杂的执行过程。  
3. Lambda表达式(针对匿名内部类)  
   * eg1(匿名内部类中的函数无参数，可以使用()代替): 
```
Thread thread = new Thread(new Runnable() {
         @Override
         public void run() {
           System.out.println("Hello Lambda Expression");
         }
       });
```
转为Lambda表达式写法：```Thread thread = new Thread(() -> System.out.println("Hello Lambda Expression"));```
   * eg2（针对有参数的情况）:
```
TreeSet<String> ts = new TreeSet<>(new Comparator<String>() {
      @Override
      public int compare(String o1, String o2) {
        return Integer.compare(o1.length(), o2.length());
      }
    });
``` 
   * 转为Lambda表达式的写法1：```TreeSet<String> ts = new TreeSet<>((o1, o2) -> Integer.compare(o1.length(), o2.length()));```    
   * 转为Lambda表达式的写法2：```TreeSet<String> ts = new TreeSet<>(Comparator.comparingInt(String::length));```
4. Lambda表达式的语法:  
Lambda 表达式在Java 语言中引入了一个新的语法元素和操作符。这个操作符为 “ ->” ， 该操作符被称为 Lambda 操作符或剪头操作符。它将 Lambda 分为两个部分：  
   * 左侧：指定了 Lambda 表达式需要的参数。  
   * 右侧：指定了 Lambda 体，即 Lambda 表达式要执行的功能。
   * 语法格式1：无参数，无返回值 `Runnable runnable = () -> System.out.println("Hello Lambda Expression");`
   * 语法格式2：一个参数 `Consumer<String> consumer = string -> System.out.println(string);`
   * 语法格式3：有两个参数，有返回值 
   ```
   BinaryOperator<Long> binaryOperator = (Long aLong, Long aLong2) -> {
         System.out.println("Hello Lambda Expression");
         return aLong + aLong2;
       };
   ```
   * 语法格式4：当 Lambda 体只有一条语句时，return与大括号可以省略 `BinaryOperator<Long> binaryOperator = (aLong, aLong2) -> aLong + aLong2;`
   * 语法格式5：类型省略：Lambda表达式的类型依赖于上下文环境，表达式中的参数类型都是由编译器推断得出的。
   ```
   BinaryOperator<Long> binaryOperator = (aLong, aLong2) -> {
         System.out.println("Hello Lambda Expression");
         return aLong + aLong2;
       };
   ```
5. Lambda表达式的作用
   - Lambda表达式为Java添加了确实的函数式编程的特性，使我们能将函数式当做一等公民看待。
   - 在函数式中，Lambda表达式的类型是函数。但在Java中，Lambda表达式是对象，他们必须依附于一类特别的对象类型——函数式接口(FunctionalInterface)
6. Lambda表达式的范围  
对于Lambda表达式外部的变量，其访问权限的粒度与匿名对象的方式非常类似。你能够访问局部对应的外部区域的局部final变量，以及成员变量和静态变量。
   - 访问局部变量：局部变量可以不定义为final类型的，编译器会自动当做final来处理，如果后面的代码中还要给age这个字段赋值，那么就会编译错误。在Lambda中想要改变age的值也是不允许的。
   ```
   final int age = 30;
   FunctionDefaultTest functionDefaultTest = name -> "name is : " + name + " age is " + age;
   ```
   - 访问成员变量和静态变量：
   ```
   private String homeAddress = "homeAddress"; // 成员变量
     private static String companyAddress = "companyAddress"; // 静态变量
     void test() {
       int age = 30;
       FunctionDefaultTest functionDefaultTest = name -> "name is : " + name + " age is " + age +
           " homeAddress is " + homeAddress +
           " companyAddress is " + companyAddress;
       System.out.println(functionDefaultTest.sayHello("zhangsan"));
     }
   ```
---
### 一.函数式接口
1. 什么是函数式接口
   - 如果一个接口只有一个抽象方法，那么该接口就是一个函数式接口。
   - 如果我们在某个接口上声明了FunctionalInterface注解，那么编译器就会按照函数式接口的的定义来要求该接口。
   - 如果某个接口只有一个抽象方法，但是我们没有给该方法声明FunctionalInterface注解，那么编译器会把该接口看作是函数式接口。
   - 如果我们有一个抽象方法，一个默认方法，那该接口还是函数式接口，不管有多少个默认方法，只要其中包含一个抽象方法，那就是函数式接口。  
   ```java
   @FunctionalInterface
   public interface FunctionDefaultTest {
   
     String sayHello(String name);
   
     String toString();
   
     default void sayWorld() {
   
     }
   }
   ```
   使用：
   ```
   FunctionDefaultTest functionDefaultTest = name -> "name is : " + name;
       System.out.println(functionDefaultTest.sayHello("zhangsan"));
   ```
   注意：如果接口中的方法是重载了Object类的方法，那么该接口还是会被认定为函数式接口，如上代码中的`toString()`方法，编译器不会报错。
2. Java8中内置的四大核心函数式接口
   * Consumer<T> : 消费型接口  
   接口中唯一的抽象方法`void accept(T t);` // 对给定的参数执行此操作。  
   eg：
   ```
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
       numbers.forEach(new Consumer<Integer>() {
         @Override
         public void accept(Integer integer) {
           System.out.println(integer);
         }
       });
       // 简化写法
       numbers.forEach((Integer number) -> System.out.println(number));
       // 简化写法(省略类型的声明，编译器根据上下文自动推断)
       numbers.forEach(number -> System.out.println(number));
       // 更简化写法 方法引用
       numbers.forEach(System.out::println);
   ```
   * Supplier<T> : 供给型接口  
   接口中唯一的抽象方法`T get();` // 得到一个结果。  
   eg：
   ```
   Supplier<Person> personSupplier = Person::new;
   personSupplier.get();   // new Person
   ```
   * Function<T, R> : 函数型接口  
   接口中唯一的抽象方法`R apply(T t);` // 将此函数应用于给定的参数。接收一个T类型的参数，返回一个R类型的结果。  
   eg:
   ```
   Function<String, Integer> convertToString = new Function<String, Integer>() {
         @Override
         public Integer apply(String s) {
           return Integer.valueOf(s);
         }
       };
       convertToString.apply("1233"); // 1233
       convertToString.apply("1233x"); // 报错，包含字符不能转为Integer
   ```
   * Predicate<T> : 断言型接口  
   接口中唯一的抽象方法`boolean test(T t);` // 在给定的参数上计算这个谓词。  
   eg:
   ```
   Predicate<Integer> predicate = new Predicate<Integer>() {
         @Override
         public boolean test(Integer integer) {
           return integer == 1;
         }
       };
       System.out.println(predicate.test(1)); // true
       System.out.println(predicate.test(2)); // false
   
       // 简化写法
       Predicate<Integer> predicate2 = integer -> integer == 1;
       System.out.println(predicate2.test(1)); // true
       System.out.println(predicate2.test(2)); // false
   ```
3. 函数式接口的实例可以：
   - 通过Lambda表达式来创建
   - 通过方法引用来创建
   - 通过构造方法引用来创建
---
### 二.方法引用和构造器引用
1. 当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！（ 实现抽象方法的参数列表，必须与方法引用方法的参数列表保持一致！ ）方法引用：使用操作符 “ ::” 将方法名和对象或类的名字分隔开来。
2. 引用方式：  
   * 方法引用： 
      - 对象::实例方法名 `(x) -> System.out.println(x);` -> `System.out::println;`
      - 类::静态方法名 `(x, y) -> Math.pow(x, y);` -> `Math::pow;`
      - 类::实例方法名 `compare((x, y) -> x.equals(y), "abcd", "abdc");` -> `compare(String::equals, "abcd", "abdc")`
   * 构造器引用：
      - ClassName::new `Function<Integer, MyClass> fun = (n) -> new MyClass(n);` -> `Function<Integer, MyClass> fun = MyClass::new;`
   * 数组引用：
      - type[]::new `Function<Integer, Integer[]> fun = (n) -> new Integer[n];` -> `Function<Integer, Integer[]> fun = Integer[]::new;`
---
### 三.Stream API
1. 流是什么？
流是数据渠道，用于操作数据源（集合，数组）等所生成的元素序列。集合讲的是数据，而流讲的是计算。  
2. 注意：
   * Stream自己不会存储元素。
   * Stream不会改变源对象，相反，他们会返回一个持有结果的新Stream。
   * Stream操作是延迟执行的。这就意味着他们会等到需要结果的时候才执行。   
3. Stream的三个步骤
   1. 创建Stream：一个数据源（如：集合，数组），获取一个流  
      - Java8中的Collection接口被扩展，提供了两个获取流的方法
         ```
         default Stream<E> stream() {
                 return StreamSupport.stream(spliterator(), false);
             }
         ```
         ---
         ```
         default Stream<E> parallelStream() {
                 return StreamSupport.stream(spliterator(), true);
             }
         ```
      - 由数组创建流：Arrays的静态方法 stream()可以获取数组流，多个重载版本
         ```
         public static <T> Stream<T> stream(T[] array) {
                 return stream(array, 0, array.length);
             }
         ```
      - 由值创建：使用Stream.of()，显示值创建流
         ```
         public static<T> Stream<T> of(T... values) {
                 return Arrays.stream(values);
             }
         ```
      - 创建无限流：iterate()和generate()方法
      ---
   2. 中间操作：一个中间操作链，对数据源的数据进行处理
      - 数据准备（参照Java8实战一书）
      ```java
      @Data // lombok 注解
      @AllArgsConstructor
      public class Dish {
      
        private final String name;
        private final boolean vegetarian;
        private final int calories;
        private final Type type;
      
        enum Type {
          MEAT, FISH, OTHER
        }
      }
      ```
      ---
      ```
      List<Dish> menus = Arrays.asList(
              new Dish("pork", false, 800, Dish.Type.MEAT),
              new Dish("beef", false, 700, Dish.Type.MEAT),
              new Dish("chicken", false, 400, Dish.Type.MEAT),
              new Dish("french fries", true, 530, Dish.Type.OTHER),
              new Dish("rice", true, 350, Dish.Type.OTHER),
              new Dish("season fruit", true, 120, Dish.Type.OTHER),
              new Dish("pizza", true, 550, Dish.Type.OTHER),
              new Dish("prawns", false, 300, Dish.Type.FISH),
              new Dish("salmon", false, 450, Dish.Type.FISH));
      ```
      - filter：该操作会接受一个谓词（一个返回boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流。
      ```
      // 收集全是素食的菜单列表
      menus.stream()
                  .filter(Dish::isVegetarian)
                  .collect(Collectors.toList());
      ```
      - distinct：它会返回一个元素各异（根据流所生成元素的hashCode和equals方法实现）的流。并确保没有重复。
      ```
      // 获取所有菜品的名字，并且去掉重复的
      menus.stream()
              .map(Dish::getName)
              .distinct()
              .collect(Collectors.toList());
      ```
      - limit(n)：该方法会返回一个不超过给定长度的流。所需的长度作为参数传递给limit。如果流是有序的，则最多会返回前n个元素。
      ```
      // 取得前3个菜品
      menus.stream()
              .limit(3)
              .collect(Collectors.toList());
      ```
      - skip(n)：扔掉前n个元素，和limit互补。
      ```
      // 扔掉前三个菜品
      menus.stream()
              .skip(3)
              .collect(Collectors.toList());
      ```
      - 综合实例
      ```
      // 你将如何利用流来筛选前两个荤菜呢？
      menus.stream()
              .filter(dish -> dish.getType() == Dish.Type.MEAT)
              .limit(2)
              .collect(Collectors.toList());
      ```
      - sorted：排序,依据给定的Comparator，返回按照此Comparator排序好的结果。
      ```
      // sorted：按照卡路里进行排序
      menus.stream()
          .sorted(Comparator.comparingInt(Dish::getCalories))
          .forEach(System.out::println);
      ```
      - peek：返回由该流的元素组成的流，另外，当从结果流中消耗元素时，对每个元素执行所提供的动作。
      ```
      // 改变流元素的内容
      menus.stream()
          .filter(Dish::isVegetarian)
          .peek(dish -> dish.setName("Beijing")).collect(Collectors.toList())
          .forEach(System.out::println);
      ```
      - peek和map的区别：
         * 两个方法接收不同的函数式接口，map方法入参为 Function，peek方法接收一个Consumer的入参。
         * 因为接收不同的入参，所以map方法执行后需要返回结果，而peek方法的Consumer是无返回值的。
         * 总结：peek接收一个没有返回值的λ表达式，可以做一些输出，外部处理等。map接收一个有返回值的λ表达式，之后Stream的泛型类型将转换为map参数λ表达式返回的类型。
      - parallel：执行该方法，返回一个并行的等效流。
      ```
      // 并行获取菜单名字
      menus.stream()
          .parallel()
          .map(Dish::getName)
          .forEach(System.out::println);
      ```
      - sequential：执行该方法，返回一个连续的等效流。
      ```
      // 转换为串行流
      menus.stream()
          .map(Dish::getName)
          .sequential()
          .forEach(System.out::println);
      ```
      - map：接受一个Lambda，将元素转换成其他形式或提取信息。它会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素（使用映射一词，是因为它和转换类似，但其中的细微差别在于它是“创建一个新版本”而不是去“修改”）
      ```
      // 获取所有菜品的名字
      menus.stream()
         .map(Dish::getName)
         .forEach(System.out::println);
      ```
      - flatMap(流的扁平化)：flatMap把流中的层级结构扁平化，就是将最底层元素抽出来放到一起。如：把几个小的list转换到一个大的list。flatmap方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。
      ```
      // 流的扁平化
      // 对于一张单词表,如何返回一张列表，列出里面各不相同的字符呢？
      String[] arrayOfWords = {"Goodbye", "World", "Hadoop", "Spark", "Park"};
      Stream<String> words = Arrays.stream(arrayOfWords);
      List<String> collect1 = words
              .map(word -> word.split(""))
              .flatMap(Arrays::stream)
              .distinct()
              .collect(Collectors.toList());
      System.out.println(collect1);
      ```
      ---
   3. 终止操作（终端操作）：一个终止操作，执行中间操作链，并产生结果。
      - forEach：为这个流的每个元素创建一个动作。
      ```
      // 获取菜品的名字
      menus.stream()
          .map(Dish::getName)
          // 给所有的元素执行一个打印的动作
          .forEach(System.out::println);
      ```
      - forEachOrdered：如果流具有已定义的遭遇顺序，则按流的遭遇顺序对此流的每个元素执行操作。
      ```
      // 打印菜品的卡路里
      menus.stream()
          .forEachOrdered(dish -> System.out.println(dish.getCalories()));
      ```
      - toArray：返回包含此流元素的数组。返回类型为Object[]
      ```
      final Object[] objects = menus.stream()
              .toArray();
      ```
      - reduce：归约，求和，此方法重载了3个版本，详见Javadoc
      ```
      List<Integer> numbers = Arrays.asList(2, 4, 5, 6, 9);
      // reduce求和
      final Integer integer = numbers.stream()
          // 接收两个参数，第一个参数是一个初始值，另一个参数是一个二元运算符
          .reduce(0, (a, b) -> a + b);
      System.out.println(integer);
      ```
      - collect：收集器
      - min：根据提供的Comparator返回该流的最小元素
      ```
      // 获取卡路里最小的菜品
      final Optional<Dish> min = menus.stream()
          .min((o1, o2) -> Integer.min(o1.getCalories(), o2.getCalories()));
      System.out.println(min.get());
      ```
      - max：根据提供的Comparator返回该流的最大元素
      ```
      // 获取卡路里最大的菜品
      final Optional<Dish> max = menus.stream()
          .max((o1, o2) -> Integer.max(o1.getCalories(), o2.getCalories()));
      System.out.println(max.get());
      ```
      - count：返回此流中的元素计数。
      ```
      // 统计菜品
      final long count = menus.stream()
          // 过滤
          .filter(Dish::isVegetarian)
          // 对过滤后的结果进行统计
          .count();
      System.out.println(count);
      ```
      - anyMatch：流中是否有一个元素能匹配给定的谓词
      ```
      // 菜单里面是否有素食可选择
      menus.stream()
              .anyMatch(Dish::isVegetarian);
      ```
      - allMatch：流中是否所有的元素都能匹配给定的谓词
      ```
      // 所有菜的热量都低于1000卡路里
      menus.stream()
              .allMatch(dish -> dish.getCalories() <= 1000);
      ```
      - noneMatch：确保流中没有任何元素与给定的谓词匹配
      ```
      // 没有任何菜的热量大于1000卡路里
      menus.stream()
              .noneMatch(dish -> dish.getCalories() > 1000);
      ```
      - 综上anyMatch，allMatch，noneMatch都用到了短路运算，即：&& 和 ||运算符
      ---
      - findFirst：返回流中的第一个元素
      ```
      // 返回素食流中的第一个元素
      menus.stream()
              .map(Dish::isVegetarian)
              .findFirst();
      ```
      - findAny：返回当前流中的任意元素（不代表第一个，因为并行，如果只找到一个元素没有限制是第一个，那么可以使用findAny）
      ```
      // 返回素食流中的任意一个元素
      menus.stream()
              .map(Dish::isVegetarian)
              .findAny();
      ```
      - iterator：返回此流元素的迭代器。
      ```
      final Dish next = menus.stream()
              .filter(Dish::isVegetarian)
              .iterator()
              .next();
      ```
---
### 四.Optional<T>类
1. 概念：Optional<T> 类(java.util.Optional) 是一个容器类，代表一个值存在或不存在，原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。
2. 常用方法：
   - Optional.of(T t) : 创建一个 Optional 实例，如果传入的t为null，那么直接报NPE。
   - Optional.empty() : 创建一个空的 Optional 实例
   - Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例
   - isPresent() : 判断是否包含值
   - get() : 返回该对象，有可能返回 null
   - orElse(T t) :  如果调用对象包含值，返回该值，否则返回t
   - orElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回 s 获取的值
   - map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()
   - flatMap(Function mapper):与 map 类似，要求返回值必须是Optional
   - 实例：数据准备，定义一个Employee类  
   ```java
   @Data
   public class Employee {
   
       private int id;
       private String name;
       private Integer age;
       private double salary;
       private Status status;
   }
   ```
   ---
   - 操作：基本用法
   ```
   // 构造一个Employee对象，设置对应参数
   Employee employee = new Employee();
   employee.setName("Zhangsan");
   Optional<Employee> optionalEmployee = Optional.of(employee);
   final String name = optionalEmployee.map(Employee::getName)
       .orElse("Unknown");
   System.out.println(name);
   ```
   --- 
   - Java8之前的写法，例子来自于网上
   ```
   public static String getChampionName(Competition comp) throws IllegalArgumentException {
     if (comp != null) {
       CompResult result = comp.getResult();
       if (result != null) {
         User champion = result.getChampion();
         if (champion != null) {
           return champion.getName();
         }
       }
     }
     throw new IllegalArgumentException("The value of param comp isn't available.");
   }
   ```
   - Java8的写法
   ```
   public static String getChampionName(Competition comp) throws IllegalArgumentException {
     return Optional.ofNullable(comp)
         .map(c->c.getResult())
         .map(r->r.getChampion())
         .map(u->u.getName())
         .orElseThrow(()->new IllegalArgumentException("The value of param comp isn't available."));
   }
   ```
---
### 五.接口中的默认方法和静态方法
- **默认方法**的定义：默认方法是在接口中的方法签名前加上了 default 关键字的实现方法。
   - eg：
   ```java
   public interface MyInterface {
   
     void sayHello();
   
     default void sayWelcome() {
       System.out.println("Welcome");
     }
   }
   ```
- 和其他方法的区别：
   - 抽象方法，接口的子类需要实现
   - 默认方法，接口的子类不需要实现，可以直接使用
   - 可以定义一个或多个默认方法
- 多重继承：解决问题的三条规则：
   1. 类中的方法优先级最高。类或父类中声明的方法的优先级高于任何声明为默认方法的优先级。
   2. 果无法依据第一条进行判断，那么子接口的优先级更高：函数签名相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。
   3. 如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法，显式地选择使用哪一个默认方法的实现.
- **静态方法**的定义：在接口中直接定义方法签名和实现，需要加关键字 static
- 调用方式：直接 InterfaceName.methodName
- 注意事项：
   1. default 关键字只能在接口中使用（以及用在 switch 语句的 default 分支），不能用在抽象类中。
   2. 接口默认方法不能覆写 Object 类的 equals、hashCode 和 toString 方法。final的方法不允许覆写
   3. 接口中的静态方法必须是 public 的，public 修饰符可以省略，static 修饰符不能省略。
---
### 六.新的时间日期API
新的API有如下设计原则：
1. 不变性：新的日期/时间API中，所有的类都是不可变的，这对多线程环境有好处。
2. 关注点分离：新的API将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。
3. 清晰：在所有的类中，方法都被明确定义用以完成相同的行为。举个例子，要拿到当前实例我们可以使用now()方法，在所有的类中都定义了format()和parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。
4. 实用操作：所有新的日期/时间API类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。
5. 可扩展性：新的日期/时间API是工作在ISO-8601日历系统上的，但我们也可以将其应用在非IOS的日历上。
---
API如下：
1. java.time.LocalDate：LocalDate是一个不可变的类，它表示默认格式(yyyy-MM-dd)的日期，我们可以使用now()方法得到当前时间，也可以提供输入年份、月份和日期的输入参数来创建一个LocalDate实例。该类为now()方法提供了重载方法，我们可以传入ZoneId来获得指定时区的日期。
   * eg：
   ```
   // 用now方法获取当前日期
   LocalDate today = LocalDate.now();
   System.out.println("Current Date=" + today);
   
   // 通过指定的参数来创建实例
   LocalDate firstDay_2014 = LocalDate.of(2014, Month.JANUARY, 1);
   System.out.println("Specific Date=" + firstDay_2014);
   
   // 根据时区来创建
   LocalDate todayKolkata = LocalDate.now(ZoneId.of("Asia/Shanghai"));
   System.out.println("Current Date in CTT=" + todayKolkata);
   
   // 在01/01/1970基础上加365天
   LocalDate dateFromBase = LocalDate.ofEpochDay(365);
   System.out.println("365th day from base date= " + dateFromBase);
   
   // 2014年后的100天
   LocalDate hundredDay2014 = LocalDate.ofYearDay(2014, 100);
   System.out.println("100th day of 2014=" + hundredDay2014);
   ```
   
2. java.time.LocalTime：LocalTime是一个不可变的类，它的实例代表一个符合人类可读格式的时间，默认格式是hh:mm:ss.zzz。像LocalDate一样，该类也提供了时区支持，同时也可以传入小时、分钟和秒等输入参数创建实例
   * eg:
   ```
   LocalTime time = LocalTime.now();
   System.out.println("Current Time=" + time);
   
   LocalTime specificTime = LocalTime.of(12, 20, 25, 40);
   System.out.println("Specific Time of Day=" + specificTime);
   
   LocalTime timeKolkata = LocalTime.now(ZoneId.of("Asia/Kolkata"));
   System.out.println("Current Time in IST=" + timeKolkata);
   
   LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
   System.out.println("10000th second time= " + specificSecondTime);
   ```
   
3. LocalDateTime：是一个不可变的日期-时间对象，它表示一组日期-时间，默认格式是yyyy-MM-dd-HH-mm-ss.zzz。它提供了一个工厂方法，接收LocalDate和LocalTime输入参数，创建LocalDateTime实例。
   * eg：
   ```
   LocalDateTime today = LocalDateTime.now();
   System.out.println("Current DateTime=" + today);
   
   today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
   System.out.println("Current DateTime=" + today);
   
   LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
   System.out.println("Specific Date=" + specificDate);
   
   LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
   System.out.println("Current Date in IST=" + todayKolkata);
   
   LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
   System.out.println("10000th second time from 01/01/1970= " + dateFromBase);
   ```

4. Instant: Instant类是用在机器可读的时间格式上的，它以Unix时间戳的形式存储日期时间
   ```
   Instant timestamp = Instant.now();
   System.out.println("Current Timestamp = " + timestamp);
   
   Instant specificTime = Instant.ofEpochMilli(timestamp.toEpochMilli());
   System.out.println("Specific Time = " + specificTime);
   
   Duration thirtyDay = Duration.ofDays(30);
   System.out.println(thirtyDay);
   ```
   
5. Duration : 计算两个时间的间隔  
6. Period：计算两个日期之间的间隔  
7. TemporalAdjuster：校正器  
8. TemporalAdjusters：提供了校正器的相关的静态方法  
9. DateTimeFormatter：格式化时间和日期  
10. ZonedDate, ZonedTime, ZonedDateTime
### Java8提供的重复注解和类型注解
@Repeatable
