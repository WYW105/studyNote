1. 匿名内部类写法：编译之后会生成一个独立的LengthComparator的.class文件，每次代码运行到这里都要创建这个类对象并分配内存，GC压力大。

   ```java
        class LengthComparator implements Comparator<String> {
            public int compare(String first, String second) {
                return first.length() - second.length();
            }
        }
        Arrays.sort(friends, new LengthComparator());
   ```

2. Lambda表达式是替代匿名内部类。

   (1). **编译时把Lambda体抽成一个私有方法，并生成invokedynamic指令**。

   ```java
    源码：
    Comparator<Integer> comparator = (a, b) -> Integer.compare(a, b);

    编译后类似：
    Comparator<Integer> comparator = invokedynamic(...)

   ```

   (2). 第一次运行到这里时，invokedynamic指令会调用LambdaMetafactory.metafactory()方法。该方法返回一个CallSite调用点，该调用点指向一个MethodHandle方法句柄,这个MethodHandle方法句柄是用于创建或获取到函数式接口的实例对象，因此**MethodHandle方法句柄主要是生成或创建实例对象的逻辑，此时实例对象可能还没被真的生成**。

   (3). 后续运行到这里时因为这个调用点已经连接好了，通常不会重复调用LambdaMetafactory.metafactory()方法。

   (4). 对于不捕获外部变量的 Lambda，JVM 通常可以复用同一个实例；对于捕获外部变量的 Lambda，则需要保存捕获变量，因为外部变量可能会变化，就要重新创建实例对象。==> 因此Lambda表达式不一定每次都会创建新的实例对象，GC压力更低。==> 这里是否复用同一个实例对象和上述是否重复调用LambdaMetafactory.metafactory()方法是两回事。

   ```java
    Arrays.sort(friends, (first, second) -> first.length() - second.length());
   ```

3. 方法引用本质上可以看成 Lambda 的一种简写形式，底层通常同样通过 invokedynamic 和 LambdaMetafactory.metafactory() 来生成或获取函数式接口实例。==> 方法引用和Lambda的性能上没什么优劣之分。

4. Lambda表达式的延迟执行，本质是**行为参数化**。Lambda 里面那段代码不会在定义 Lambda 的地方立刻执行，而是被包装成函数式接口对象，等到这个接口方法被真正调用时才执行。

   ```java
        //这里不会立刻执行Lambda表达式里写的方法，
        Runnable r = () -> {
            System.out.println("执行 Lambda 代码");
        };

        System.out.println("Lambda 已经定义完成");


        // 等到run方法被调用的时候，才会执行。run方法就是函数式接口的那个唯一的抽象方法
        r.run();
   ```

5. Optional作为方法的返回值，核心就是强制养成好习惯：写代码时就要把返回值可能不存在的情况考虑进去，作为处理流程的一个常规分支，而不是抛出空指针异常NullPointerException。

   ```java
   import java.util.Optional;
   import java.util.function.Supplier;

   public class OptionalCheatSheet {

       public static void main(String[] args) {

           // ==================== 1. 创建 Optional ====================

           // 确定非空
           Optional<String> opt1 = Optional.of("Hello");

           // 可能为空
           Optional<String> opt2 = Optional.ofNullable(null);
           Optional<String> opt3 = Optional.ofNullable("World");

           // 显式创建空的
           Optional<String> empty = Optional.empty();


           // ==================== 2. 判断与消费 ====================

           // isPresent() + get() —— 不推荐，但要知道
           if (opt1.isPresent()) {
               System.out.println(opt1.get());  // Hello
           }

           // ifPresent() —— 有值就消费
           opt1.ifPresent(val -> System.out.println("值存在: " + val));
           opt2.ifPresent(val -> System.out.println("这行不会打印"));


           // ==================== 3. 取出值（带默认） ====================

           // orElse() —— 无论Optional是否有值，默认值表达式都会执行
           String result1 = opt1.orElse(expensiveDefault());  // 会打印"执行了昂贵操作"
           String result2 = opt2.orElse("默认值");

           // orElseGet() —— 只有Optional为空时，才会执行Supplier
           String result3 = opt1.orElseGet(() -> expensiveDefault());  // 不会执行
           String result4 = opt2.orElseGet(() -> "默认值");

           // orElseThrow() —— 为空时抛异常
           try {
               String result5 = opt2.orElseThrow(() -> new IllegalStateException("值为空"));
           } catch (IllegalStateException e) {
               System.out.println("捕获到异常: " + e.getMessage());
           }


           // ==================== 4. 转换（map / flatMap） ====================

           // map —— 转换值
           Optional<Integer> length = opt1.map(String::length);
           length.ifPresent(len -> System.out.println("长度: " + len));  // 5

           // map 链式
           Optional<String> upper = opt1.map(String::toUpperCase)
                                        .map(s -> s + "!!!");
           System.out.println(upper.orElse("空"));  // HELLO!!!

           // flatMap —— 拍平嵌套Optional
           // 假设 getDepartment 返回 Optional<String>
           Optional<String> dept = opt1.flatMap(OptionalCheatSheet::getDepartment);
           System.out.println(dept.orElse("无部门"));  // 技术部

           // 对比 map 会产生的嵌套
           Optional<Optional<String>> nested = opt1.map(OptionalCheatSheet::getDepartment);


           // ==================== 5. 过滤 ====================

           // filter —— 不满足条件就变成空
           Optional<String> filtered = opt1.filter(s -> s.length() > 10);
           System.out.println(filtered.orElse("长度不足"));  // 长度不足

           Optional<String> passed = opt1.filter(s -> s.contains("ell"));
           System.out.println(passed.orElse("不包含"));  // Hello


           // ==================== 6. 串联使用示例 ====================

           String finalResult = Optional.ofNullable(getUserInput())
               .map(String::trim)
               .filter(s -> !s.isEmpty())
               .map(String::toUpperCase)
               .orElse("DEFAULT");
           System.out.println("最终结果: " + finalResult);
       }

       // 模拟返回 Optional 的方法
       private static Optional<String> getDepartment(String user) {
           return Optional.of("技术部");
       }

       // 模拟昂贵操作
       private static String expensiveDefault() {
           System.out.println("执行了昂贵操作");
           return "昂贵默认值";
       }

       // 模拟用户输入（可能为null）
       private static String getUserInput() {
           return null;  // 试试改成 "  hello world  "
       }
   }

   ```

6. Stream：

   (1). 中间操作： 对于map、peek这种返回值是Stream\<T>的就可以继续链式调用,属于**中间操作**，中间操作**只要终端操作没被调用，中间操作就只是声明不会有任何数据处理**。map会把元素映射成新的元素，peek不改变元素。

   下面这段代码不会打印任何东西，因为没有终端操作。

   ```java

    list.stream()
        .map(x -> {
            System.out.println("map: " + x);
            return x * 2;
        });
   ```

   (2). 终端操作：foreach返回值是void，并且是会立即执行的，不产出新的流，属于**终端操作**。collect、count也属于终端操作。

   ```java
    import java.util.*;
    import java.util.stream.*;
    import java.util.function.*;

    public class StreamCheatSheet {

        public static void main(String[] args) {

            // ==================== 1. 创建流 ====================

            // 从集合
            List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");
            Stream<String> streamFromList = names.stream();

            // 从数组
            String[] arr = {"a", "b", "c"};
            Stream<String> streamFromArray = Arrays.stream(arr);

            // 直接造
            Stream<String> streamOf = Stream.of("x", "y", "z");

            // 生成无限流（需配合limit截断）
            Stream<Integer> infinite = Stream.iterate(0, n -> n + 2); // 0, 2, 4, 6, ...
            Stream<Double> random = Stream.generate(Math::random);


            // ==================== 2. 中间操作 ====================

            List<String> result;

            // filter —— 筛选
            result = names.stream()
                          .filter(n -> n.length() > 3)
                          .collect(Collectors.toList());
            System.out.println("filter: " + result);  // [Alice, Charlie, David]

            // map —— 转换
            result = names.stream()
                          .map(String::toUpperCase)
                          .collect(Collectors.toList());
            System.out.println("map: " + result);  // [ALICE, BOB, CHARLIE, DAVID, EVE]

            // flatMap —— 拍平嵌套
            List<List<String>> nested = Arrays.asList(
                Arrays.asList("a", "b"),
                Arrays.asList("c", "d")
            );
            List<String> flat = nested.stream()
                                      .flatMap(List::stream)
                                      .collect(Collectors.toList());
            System.out.println("flatMap: " + flat);  // [a, b, c, d]

            // sorted —— 排序
            result = names.stream()
                          .sorted(Comparator.reverseOrder())
                          .collect(Collectors.toList());
            System.out.println("sorted: " + result);  // [Eve, David, Charlie, Bob, Alice]

            // distinct —— 去重
            List<Integer> nums = Arrays.asList(1, 2, 2, 3, 3, 3);
            List<Integer> distinctNums = nums.stream()
                                             .distinct()
                                             .collect(Collectors.toList());
            System.out.println("distinct: " + distinctNums);  // [1, 2, 3]

            // limit / skip —— 截取 / 跳过
            List<String> limited = names.stream().limit(2).collect(Collectors.toList());
            List<String> skipped = names.stream().skip(3).collect(Collectors.toList());
            System.out.println("limit(2): " + limited);  // [Alice, Bob]
            System.out.println("skip(3): " + skipped);   // [David, Eve]

            // peek —— 调试，查看流中元素（不影响数据）
            System.out.print("peek: ");
            names.stream()
                 .peek(n -> System.out.print(n + " "))  // 有终端操作时才执行
                 .collect(Collectors.toList());
            System.out.println();


            // ==================== 3. 终端操作 ====================

            // collect —— 收集到集合
            List<String> collected = names.stream()
                                          .filter(n -> n.startsWith("A"))
                                          .collect(Collectors.toList());
            System.out.println("collect: " + collected);  // [Alice]

            // collect —— 收集到Map (名字 -> 长度)
            Map<String, Integer> nameMap = names.stream()
                .collect(Collectors.toMap(
                    Function.identity(),  // key: 名字本身
                    String::length        // value: 名字长度
                ));
            System.out.println("toMap: " + nameMap);

            // collect —— 分组（按长度）
            Map<Integer, List<String>> byLength = names.stream()
                .collect(Collectors.groupingBy(String::length));
            System.out.println("groupingBy: " + byLength);

            // collect —— 拼接字符串
            String joined = names.stream()
                                 .collect(Collectors.joining(", ", "[", "]"));
            System.out.println("joining: " + joined);  // [Alice, Bob, Charlie, David, Eve]

            // forEach —— 逐个消费
            System.out.print("forEach: ");
            names.stream().forEach(n -> System.out.print(n + " "));
            System.out.println();

            // reduce —— 归约/汇总
            List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
            int sum = numbers.stream().reduce(0, (a, b) -> a + b);
            int product = numbers.stream().reduce(1, (a, b) -> a * b);
            Optional<Integer> max = numbers.stream().reduce(Integer::max); // 无初始值返回Optional
            System.out.println("reduce sum: " + sum);        // 15
            System.out.println("reduce product: " + product); // 120
            System.out.println("reduce max: " + max.orElse(-1)); // 5

            // count —— 计数
            long count = names.stream().filter(n -> n.length() > 3).count();
            System.out.println("count: " + count);  // 3

            // anyMatch / allMatch / noneMatch
            boolean anyLong = names.stream().anyMatch(n -> n.length() > 5);
            boolean allShort = names.stream().allMatch(n -> n.length() < 10);
            boolean noneLong = names.stream().noneMatch(n -> n.length() > 10);
            System.out.println("anyMatch(>5): " + anyLong);   // true (Charlie)
            System.out.println("allMatch(<10): " + allShort); // true
            System.out.println("noneMatch(>10): " + noneLong); // true

            // findFirst / findAny —— 找第一个/任意一个
            Optional<String> first = names.stream()
                                          .filter(n -> n.length() > 3)
                                          .findFirst();
            System.out.println("findFirst: " + first.orElse("无")); // Alice

            // min / max —— 最大最小值
            Optional<String> minLen = names.stream()
                                           .min(Comparator.comparingInt(String::length));
            System.out.println("min: " + minLen.orElse("无")); // Bob
        }
    }

   ```
