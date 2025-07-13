<!--
 * @Author: wyw 355432666@qq.com
 * @Date: 2023-12-21 10:20:56
 * @LastEditors: wyw 355432666@qq.com
 * @LastEditTime: 2024-06-18 17:43:17
 * @FilePath: \Java Core学习笔记\第五章 继承.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
### 多态 Polymorphism  Java中的Object变量是多态的

1."is-a" rule 置换法则，子类可以替换任何地方的他的父类，可以把子类的对象赋值给超类的变量==》**多态就是一个对象可以有多种状态，他可以被自身类型的变量引用，也可以被他的（1）父类变量引用，（2）引用之后该变量不能调用只在子类中存在但是父类中没有的方法，（3）变量调用的某方法子类如果重写了，真正执行的就是子类重写的方法，子类没重写过执行的就是父类的方法。**

(1)  

    Employee e;
    e=new Employee(); //超类
    e=new Manager(); //子类  
(2) staff[0]=boss;

    Manager boss=new Manager();
    Employee[] staff=new Employee[3];
    staff[0]=boss;
    // staff数组是父类employee类型，boss是子类manager，“staff[0]=boss;”这句就是多态
    但是boss有子类manager的方法setBonus，staff[0]没有，因为staff[0]是父类employee类型
    staff[0].setBonus(5000); // ERROR

1. 可以把子类的数组赋值给父类的数组变量

    ```
    Manager[] managers=new Manager[10]
    Employee[] staff=managers;
    //staff和managers引用的是同一个数组，现在就不能再让staff中的元素引用new Employee对象了，会矛盾
    ```

### 方法的调用 重载解析（overloading resolution）

1. 可能存在多个名字一样但是参数类型不一样的方法，f(int)和f(String)，编译器首先获得所有名字为f的方法，然后根据调用方法的参数类型选择使用哪个方法，这个过程是**重载解析**
2. 方法名和参数类型组成方法的签名，返回值类型不属于方法的签名。
3. **重写父类方法**  
   （1）在子类中重写父类的方法时要注意兼容返回值===>**子类方法的返回值类型是父类方法返回值类型的子类型或者相同类型**  
   （2）重写的**子类方法的可见度不能低于父类方法**===>父类方法是public，子类也要是public
4. 绑定：把方法的调用和方法主体关联起来，JVM负责完成绑定工作。
5. private，static，final或者构造器方法是**静态绑定**的，编译期就绑定了。  
   （1）private不能继承，所以不可能通过子类对象调用，所以可以确定方法主体；  
   （2）final 不能被子类重写  
   （3）static与类关联而不是与实例关联，通过类名调用  
   （4）constructor
6. **运行时再绑定是动态绑定**，java大部分是动态绑定

```
public class ManagerTest
{
   public static void main(String[] args)
   {
      // construct a Manager object
      var boss = new Manager("Carl Cracker", 80000, 1987, 12, 15);
      boss.setBonus(5000);

      var staff = new Employee[3];

      // fill the staff array with Manager and Employee objects

      staff[0] = boss;
      staff[1] = new Employee("Harry Hacker", 50000, 1989, 10, 1);
      staff[2] = new Employee("Tommy Tester", 40000, 1990, 3, 15);

      // print out information about all Employee objects
      for (Employee e : staff)   //!!!!!!多态
         System.out.println("name=" + e.getName() + ",salary=" + e.getSalary());
   }
}
```

1. 动态绑定每次调用虚拟机都要搜索，耗时===》虚拟机预先创建类的方法表，在表中搜索。上面的就要找到Employee类和Manager类的方法表，===》运行时，虚拟机在表中查找符合签名的类的方法===>调用方法

   ```
   for (Employee e : staff)   //!!!!!!多态
         System.out.println("name=" + e.getName() + ",salary=" + e.getSalary());
   ```

2. 动态绑定让程序有**可扩展性**，e.getSalary()中的e的类型变成其他类的话，e.getSalary()这句代码不需要重新编译

### 5.1.7 final类不可继承

1. **final类，不能被继承；final类中的所有方法默认final方法，final类中的实例域不是默认final的但是也可以定义为final。**

   ```
   public final class Executive extends Manager 
   {

   }
   ```

2. 类中的final方法不能被子类重写

   ```
   public class Employee{
      ...
      public final String getName(){
         return name;
      }
   }
   ```

3. 使用final的情况===>不允许子类修改语义（阻止多态），将类或方法设置为final
4. **超纲内容** final和内联（内联就是方法调用少了一个栈，什么是栈？以后再说）
5. records和Enumerations也是final的

### 5.1.8 对象引用的强制类型转换（cast），尽量少用cast和instanceof

   ```
   double x=3.04;
   int nx=(int) x;

   var staff=new Employee[3];
   staff[0] = new Manager("Carl Cracker", 80000, 1987, 12, 15);
   Manager boss=(Manager) staff[0]  // Manager是子类，Employee是父类，强制类型转换
   ```

1.只能在继承层次内进行cast强制类型转换  
2. 超类变量引用子类对象，直接引用不用cast；  
3. **子类变量引用超类对象，必须把超类对象cast为子类才能使用子类的特有方法**。转换之前使用**instanceof** 检查

   ```java
   if (staff[i] instanceof Manager)
      {
      Manager boss = (Manager) staff[i];
      //转换之后才能使用Manager子类的特有方法
      boss.setBonus(5000);
      }
   ```  

4. null instanceof C ===>返回false，因为null不指向任何对象  

### 5.1.10 instanceof的模式匹配，匹配后直接定义变量

1. an instanceof pattern can shadow a field.

   ```java
   class Value
   {
      private double v;
      public boolean equals(Value other){
         if (other instanceof LabeledValue v)   
         // other是LabeledValue实例，就cast并声明局部变量v===>LabeledValue v=(LabeledValue) other
         可以在这里直接使用变量v，变量v引用的是LabeledValue的实例对象
         else
         // other不是LabeledValue实例，则v还是类的私有域
      }
      . . .
   }
   ```

### 5.1.9 protected

1. 使用protected的情况：  
   （1）允许子类访问超类中的方法或者域
2. 同一个package下的任何类都可以使用protected field
3. **protected访问控制的范围，同包或者子类可以访问**，子类可以访问的**子类本身的实例对象继承到的field**，而不是父类实例对象中的field
4. 访问控制符号
| Modifier    | 自身Class | Package | Subclass | World |
| ----------- | -----     | ------- | -------- | ----- |
| public      |  Y        |    Y    |    Y     |   Y   |
| protected   |  Y        |    Y    |    Y     |   N   |
| no modifier |  Y        |    Y    |    N     |   N   |
| private     |  Y        |    N    |    N     |   N   |

## 5.2 Object class 所有类的终极超类

### 5.2.1 Object类型的变量

Object类型的变量可以引用任何类型的对象实例(不能引用基本类型：6个数字类型，1个布尔类型，1个characters)

   ```
   Object obj=new Employee();
   //Object 变量只是通用持有，对变量具体操作需要cast
   Employee e=(Employee) obj;
   ```

### 5.2.2 equals method

1. **Object.equals()就是用==比较两个对象是否引用同一个内存地址，所以一个对象如果没有重写equals()方法那么就是比较内存地址，如果重写了才是比较值**，但是实际情况这种比较没有实际意义，因此会自定义equals()方法看对象的状态是否相等。

   ```
      public boolean equals(Object otherObject)
      {
         //第一步、 下面这句就是调用Object.equals()方法
         if (this == otherObject) return true;

         //第二步、 must return false if the explicit parameter is null
         if (otherObject == null) return false;

         //第三步、对象要属于同一个类
         if (getClass() != otherObject.getClass()) return false;

         //第四步、 otherObject跟当前对象属于同一个类，并且非null，就把otherObject转换为对应类型的变量
         var other = (Employee) otherObject;

         //第五步、比较域： 用==比较基本类型域，用Objects.equals()比较对象类型域
         return Objects.equals(name, other.name) 
            && salary == other.salary && Objects.equals(hireDay, other.hireDay);
      }
   ```

2. object.getClass()返回这个对象所属的类
3. 上面第三步中，如果equals的语义可以在子类中改变，使用getClass()方法判断；getClass()的子类和父类是不相等的；  
   如果所有子类中的语义相等，可以使用instanceof,因为child instanceof parent返回true，应该将parent.equals声明为final，子类不可以重写

   ```
   if(!(otherObject instanceof ClassName other)) return false
   ```

### 5.2.3 Equality Testing and Inheritance

1. java的equals方法要求以下属性：

   自反性reflexive:x.equals(x)return true

   对称性symmetric:x.equals(y)==y.equals(x)

   传递性transitive:x==y;y==z;则x==z

   一致性consistent：返回可重复的结果

   非null不等于null：x.equals(null) return false

2. 数组类型的域，使用数组的静态方法检查Arrays.equals;Arrays.deepEquals方法检查多维数组。
3. @override 重写的方法可以进行注解
4. 当数组a和b的长度相等，且a和b对应位置的元素equal才返回true

```
static boolean equals(xxx[] a,xxx[] b)
```

### 5.2.4 hashCode(散列码) Method，返回integer可以是负数

1. 散列码应该是没有规律的
2. **Object类中定义了hashCode()方法，所以每个对象都有默认的散列码**，起源于对象的内存地址（字符串的散列码起源于他们的内容）

   ```
   var s="OK";
   var sb=new StringBuilder(s);
   vat t=new String("OK");
   var tb=new StringBuilder(t)
   // s.hashCode()和t.hashCode()相同，sb.hashCode()和tb.hashCode()不同，因为s和t是字符串根据内容决定散列码，sb和tb是对象，根据内存地址决定散列码
   ```

3. 第九章散列表和equals待学习
4. **对象的散列码是结合对象的实例域产生的**

   ```
   public int hashCode(){
      return 7*Objects.hashCode(name)
         +11*Double.hashCode(salary)
   }
   ```

   **参数是null时objects.hashCode()方法返回0**
   objects.hash()方法结合所有参数给出散列码

   ```
   public int hashCode(){
      return Objects.hash(object1,object2,...)
   }
   ```

5. **两个对象hashcode相同，他们不一定相等（只是哈希算法烂发生了哈希碰撞），equasl方法返回ture他们才相等；equals方法返回true则hashcode一定相等。** 那为什么还要有hashcode方法？当hashcode不同时可以直接判断两个对象不同，不用再调用equals方法了。==》**所以重写equals方法也要重写hashcode方法**
6. record type自带hashCode方法（根据实例域的值）
7. 当实例域的值很接近的时候，hashcode算法要避免重复、避免哈希碰撞。

### Object类的toString()方法

1. **返回className+@+hashCode的十六进制字符串（hashcode是整数）==>建议所有对象都重写这个方法。**
2. 作用:对象跟字符串相加时自动触发对象的toString方法

   ```
   var p=new Point(10,20)
   String msg="The current position is"+p
   ```

3. ""+x取代x.toString()；因为前者x可以是基本类型或对象
4. System.out.printIn(Object)==>调用object类的toString方法；因为PrintStream类没有重写toString方法
5. 数组也用object.toString()方法

   ```
   int [] arr={2,3,5,7,11,13}
   String s=""+arr
   //s是"[@1a46e30"
   ```

   **Arrays.toString**和**Arrays.deepToString**重写了Object.toString()方法，输出"[2,3,5,7,11,13]",deepToString输出多维数组

## 5.3 Generic Array Lists 泛型数组列表

1. C++中必须在编译时就确定数组大小，Java可以在运行时确定，代码如下：

   ```
   int l=100*2;
   Employee[] staff=new Employee[l]
   ```

   上面的代码最后还是写死了数组的长度，所以用ArrayList

### 5.3.1 声明数组列

1. ArrayList<T> 即时自动调整容量===>**数组满了之后继续添加元素，数组列表会自动创建一个更大的数组并复制原先数组中的元素过来**

   ```
   ArrayList<Employee> staff=new ArrayList<Employee>()
   ArrayList<Employee> staff=new ArrayList<>()  //等号右边的类型可以省略
   ```

2. Java5之前也有ArrayList，但是没有泛型，只是一个自适应大小的集合，用来保存Object类型元素的数组表
3. staff.add(new Employee(...));add方法增加元素;
4. 如果能够明确数组列表容量如100：前100个元素就不会有1.0中重新分配空间的开销
   （1）可以用staff.ensureCapacity(100) 方法  
   （2）初始化的时候写上 ArrayList<Employee> staff=new ArrayList<>(100)  
   **但是数组列表的容量100和new Employee[100]普通数组的长度不同**：  
      普通数组就真的被划分成100个槽点用来放置元素，但数组列表只是有放置100个元素而不用重新分配空间的潜力，实际的长度可以小于100或者大于100（超过100个元素就重新分配空间）
5. 数组列表当前的实际长度 staff.size()，相当于arr.length()
6. trimToSize()方法，将数组列表的内存块减小到刚好保存当前所有元素的大小===>后续添加新元素内存块要重新移动，耗费时间，所以最终确定了再用trimToSize

### 5.3.2 Accessing Array List

1. ArrayList不是Java语言的一部分,只是标准库中的一个有用的累
2. get和set方法,**set方法是替代原有元素，不是新增元素，set方法返回之前在i位置上的元素**，原来i的位置上没有元素，不可以用arrayList.set(i,x),只能用arrayList.add(x)新增

   ```
   staff.set(i,harry)  //相当于数组的arr[i]=harry
   Employee e=staff.get(i)  //先当于Employee e=a[i]

   staff.add(x);
   staff.add(i,x);在某个位置插入元素
   ```

3. 如果一个ArrayList没有泛型，**get方法返回一个对象，需要cast强制类型转换**;**add和set方法会接受任何类型，编译器不报错，这样不好**

  ```
  Employee e=(Employee) staff.get(i);
  ```

4. remove方法,移除某个元素

   ```
   Employee e=staff.remove(n);
   ```

5. 如果数组列表里很多元素并且需要经常修改，考虑使用linked list（TODO 第九章）；
6. 既可以灵活扩展数组，又可以方便访问数组不用get和set。**Java中的普通数组在定义的时候必须给出长度，长度不可更改，否则报错**

   ```
   ArrayList<X> list=new ArrayList<>();
   while(...)
   {
      x=...;
      list.add(x)
   }
   // 先用arrayList添加元素，这个过程是可以灵活扩展的
   x[] a=new X[list.size()];
   list.toArray(a);  //然后把arrayList中的元素复制到数组a中，去数组a中获取元素很方便
   ```

### 5.3.3 泛型的数组列表和raw ArrayList原始数组列表的兼容性

1. 类型化数组列表传递给要求原始数组列表的地方不报错；
2. 把原始数组列表传递给要求类型化的地方会报错warning,cast转换也没用

   ```
   ArrayList<Employee> result = (ArrayList<Employee>) employeeDB.find(query);
   // yields another warning
   ```

3. **编译器做完类型检查之后，类型检查没问题的话就把所有类型化的数组列表转为原始数组列表**==>程序运行的时候没有虚拟机中的类型参数，都执行一样的运行时检查===>对于不严重的警告，用@SuppressWarnings注解

   ```
   @SuppressWarnings("unchecked") 
   ArrayList<T> result=(ArrayList<T>) employeeDB.find(query);

   ```

## 5.4 Object Wrappers and Autoboxing  对象包装和自动装箱（**给编译器用的，在生成类的字节码时插入必要的调用，虚拟机只负责执行字节码**）

0. todo 发现装箱其实就是调用了 包装类的valueOf()方法，拆箱其实就是调用了 xxxValue()方法。
1. 把原始类型转换成对象，所有原始类型都有对应的类:  
   Integer Long Float Double Short Byte Character Boolean  
   **前六个继承超类Number，Number是一个抽象类**
2. **所有封装类都是final类，构造出来之后就不能改变封装对象的值**，final所以也不可被继承
3. <T>泛型只能是wrappers类型不能是原始类型

   ```
   var list = new ArrayList<Integer>();
   ```

4. ArrayList<Integer>比int[]数组低效，因为每个元素都被包装成对象，当便捷比高效重要的少数元素集合的时候可以用
5. **自动装箱，就是调用封装类的valueOf()方法**

   ```
   list.add(3)
   //自动转换为
   list.add(Integer.valueOf(3))
   ```

6. **自动拆箱，就是调用封装类的实例的xxxValue()方法**

   ```
   List<Integer> list = new ArrayList<>();
   int n=list.get(i);
   //自动相当于
   int n=list.get(i).IntValue();
   ```

7. 算数计算自动装箱和拆箱

   ```
   Integer n=3;  //装箱
   n++; // 拆箱进行计算，算完之后再装箱
   ```

8. 封装类型是对象，==比较的是引用地址,值一样不是一个对象也不相等,**直接不要用==比较**

   ```
   Integer a=100;
   Integer b=100;
   if(a==b)  // false
   ```

   有时会成立，看Java实现，有时会把常用的值封装成同一个对象==>**结果不确定，应该用equals方法比较**

   简单类型，小的整数被包装到固定对象中，==成立，所以具体看规范要求的Java实现自动装箱规范要求boolean、byte、char 127， 介于-128 ~ 127 之间的short 和int 被包装到固定的对象中。例如， 如果在前面的例子中将a 和b 初始化为100， 对它们进行比较的结果一定成立。
9. 不要把他们用作locks===TODO
10. 不要用new操作符调用封装类的构造器，计划的废除：  
    自动装箱Integer a=1000或者Integer.valueOf(1000)
11. 包装类可以引用null：  Integer n=null;
12. Integer和Double掺在一起计算，Integer自动封装成Double
13. 数组封装类会存放基础方法，静态的

   ```
   int x=Integer.parseInt(s);
   ```

14. **封装类是final类的，所以虽然是对象，作为方法的参数被传入方法时，在方法内部改变封装类的值也改变不了原来的对象的值，==>这里封装类的表现跟基本类型一样。**

    ```java
      public static void triple(int x) // x是复制的外面的参数，改变不了外面的参数

      {
         x = 3 * x; // modifies local variable
      }

      public static void triple(Integer x) // won't work 对象形参和实参引用同一个地址，但是封装类immutable
      {
      . . .
      }
    ```

## 5.5 参数的数量可变的方法

1. 剩余参数Object... args自动装箱

   ```
   public class PrintStream{
      // 下面的args规定了是对象，所以相当于object[]，如果剩余参数里有基本类型，会自动封装成封装类。
      public PrintStream printf(String fmt,Object... args) {
         return format(fmt,args);
      }
   }
   ```

## 5.6 Abstract Classes 抽象类（interfaces中多见）

1. 对于子类的顶层超类，比如person类，这种类的意义一般不是要使用他的实例，而是作为其他类的基本类。===>**抽象类不能被实例化**

   ```
   new Person("Vince Vu") //Error
   ```

2. **定义抽象类的对象变量不报错**,变量实际引用非抽象的子类

   ```
   Person p=new Student();
   ```

3. 抽象类可以有（1）抽象方法（不需要具体想实现的方法）、（2）实例域和具体方法（把通用的域和方法放在超类中）。
4. **抽象方法加abstract关键字，不需要具体实现**，不会被执行，只是**占位**，在子类中写具体实现。
5. 类有一个或多个方法是abstract的，那类本身也加abstract关键字。

   ```
   public abstract class Person{
      ...
      public abstract String getDescription();
   }
   ```

6. 子类继承了抽象类，而没有重写抽象方法==>子类中有抽象方法==>**子类也是抽象类**
7. 没有抽象方法的类也可以被定义为抽象类

## 5.7 Enumeration Classes

1. 枚举类型实际上是枚举类,**SMALL,LARGE,MEDIUM是类的实例**

   ```
   public enum Size{SMALL,LARGE,MEDIUM};
   ```

2. 枚举类中可以写构造器、方法和实例域，构造器默认且只能是private，private可省略

   ```
   public enum Size
   {
   SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");
   private String abbreviation;
   // 下面是构造器 automatically private
   Size(String abbreviation) { this.abbreviation =
   abbreviation; }
   public String getAbbreviation() { return abbreviation; }
   }
   ```

3. **所有的枚举类都是抽象类Enum的子类，且所有枚举类都是final类**

   ```java
   //枚举类相当于
   public final class T extends Enum {
      ...
   }
   ```

4. 方法

   ```
   Size[] values=Size.values();// 返回全部值的数组
   Size.SMALL.toString(); //返回常量名“SMALL”
   Size.MEDIUM.ordinal();//返回MEDIUM在类中的序号，跟数组index一样从0开始算
   Size s=Enum.valueOf(Size.class,"SMALL")// 返回指定的枚举常量，赋值给s变量
   ```

5. **枚举是唯一实例，所以比较的时候可以直接使用 ==，equals方法里面也是默认用==**。
<https://medium.com/danonrockstar/comparing-enum-values-in-java-85e484a9190c>

## Sealed Classes 密封类 （Java15、17的新内容）

1. **密封类用于控制谁能做这个类/接口的直接子类/子接口，permits关键字声明密封类的直接子类/子接口**

   ```java
   public abstract sealed class JSONValue
   //只允许这些类继承JSONValue，定义其他子类会报错
      permits JSONArray, JSONNumber, JSONString, JSONBoolean,
   JSONObject, JSONNull {
      ...
   }
   ```

2. 被允许的子类必须是可使用的。不能是嵌套在其他类下的private类,或其他包下的包可见类（TODO：什么叫包可见类？子类可以是密封类自己的private类嘛）；
3. 被允许的子类如果是public类，子类必须与密封类在同一个package或module
4. 省略permits，则子类必须都定义在密封类所在的文件内。即子类都不需要被设置为public的场景下适用省略permits（一个文件只能有一个公共类，密封类能有子类，所以不是private的是public的，所以所有子类都没办法public）；
5. 使用原因（作用）：**编译运行时检查**

   ```java
   public String type()
   {
      return switch (this)
      {
      case JSONArray j -> "array";
      case JSONNumber j -> "number";
      case JSONString j -> "string";
      case JSONBoolean j -> "boolean";
      case JSONObject j -> "object";
      case JSONNull j -> "null";
      // No default needed here,因为所有被允许的子类都写了，没有其他可能了，就不用default了
      };
   }
   ```

TODO:P405 NOTE
6. 密封类的子类也可有子类，**密封类只定义他的直接子类**。

7. 密封类的子类必须明确指出是final（不能被继承），sealed（密封类被有限继承），non-sealed（公开的，不限制类的继承）

8. non-sealed关键字，java的第一个有分隔符的关键字，可能是以后的趋势。现有代码可能不支持编译

9. interfaces是抽象类的一种普遍化，也有Sealed interfaces，对应subtypes

10. Records 和 enumerations可以实现接口，但是不能继承类。

## 5.9 Reflection 反射-更好的动态操作Java代码(**主要用于构建工具，而不是编写具体的程序**)

1. 通过反射Java可以支持接口构建、对象关系映射，反射的能力：

      (1). **运行时分析类的能力，利用反射机制获取到类的类型、实例域、方法、构造器、注解、修饰符等。**

      (2). 上述对应Class类的getFields, getMethods, getConstructors，getAnnotation()，getDeclaredFields, getDeclaredMethods, getDeclaredConstructors，getDeclaredAnnotation()，getModifiers()，**Declared就是只是当前类声明的，不加Declared就是包含超类中声明的**。

      (3). **private的域不可获得。f.setAccessible(true)方法之后，f.get()就可用了。** setAccessible方法是AccessibleObject类的方法，是Field, Method, 和 Constructor三个类的公共超类。

### 5.9.1 The Class Class

1. java运行系统在运行时对所有对象进行 运行时间类型识别（runtime type identification）；用于虚拟机选择正确的方法去执行。
2. Class 类可以访问类型识别信息
3. 对象调用getClass方法返回对象的Class类型的实例; getName()方法返回类的全类名

   ```
   var generator=new Random();
   Class cl=generator.getClass();
   String name=cl.getName();// "java.util.Random" 有包的名字
   ```

4. forName方法通过类名获取对应的Class对象

   ```
   String className="java.util.Random";
   Class cl=Class.forName(className);
   ```

5. T.class (T是任意java类型,基本类型和类)，可以获取class类的实例。所以**class类是对类型type的描述**

   ```
   Class cl1=Random.class;
   Class cl2=int.class;  // int不是class，是类型
   ```

6. class类是泛型类；Employee.class的类型是Class<Employee>
7. 虚拟机为每个类型管理一个唯一的Class对象==>用==对比类的对象

   ```
   if(e.getClass()==Employee.class)...
   ```

8. 有了类型的Class对象，getConstructor方法获得Contructor类型的对象；  
   newInstance方法构造实例。

   ```
   Class cl=Class.forName("java.util.Random");
   Object obj=cl.getConstructor().newInstance();
   ```

   **有参数的构造器，getConstructor()传参数类型，newInstance()传参数值**

   ```
   Class cl = Random.class; // or any other class with a
   constructor that
   // accepts a long parameter
   Constructor cons = cl.getConstructor(long.class);
   Object obj = cons.newInstance(42L);
   ```

9. Class.toInstance相比getConstructor()方法是过时的实例化方法，编译时不会做异常检查，不会强制处理异常代码==>可能会导致运行时异常。getConstructor()会把异常封装在InvocationTargetException中，编译时会捕获.

### 5.9.2 A Primer on Declaring Exceptions("throw an exception")

1. “throw an exception”==>catch来处理,不处理则程序终止。
2. 异常的类型 unchecked exceptions和checked exceptions
3. 对于无法避免的异常，在方法名上加throws,调用这个方法的其他方法也要加throws===>方便程序快速找出是哪个方法抛出的checked exceptions。出错程序终止的时候提供一个**栈轨迹stack trace** TODO：什么是栈轨迹？

   ```
   public static void doSomething(String name)
      throws ReflectiveOperationException 
   {
      Class cl=Class.forName(name);
   }
   ```

### 5.9.3 Resources 资源（和类关联的数据文件，如图、音文件，text文件）

1. **资源文件要和资源对应的类放在同一个文件夹下。==>虚拟机知道怎么查到到某个类，借助这个功能才能查找到同一位置下的关联资源文件。**
2. T.class调用getResource()方法返回资源的url  
   T.class调用getResourceAsStream()方法返回资源的输入流
   找不到文件返回null，所以不会抛出异常

   ```
   public class ResourceTest
   {
      public static void main(String[] args) throws IOException
      {
         Class cl = ResourceTest.class;
         URL aboutURL = cl.getResource("about.gif");
         // aboutURL 是完整路径绝对路径file:/D:/javaStudy/corejava/out/production/corejava/resources/about.gif
         var icon = new ImageIcon(aboutURL);
   
         InputStream stream = cl.getResourceAsStream("data/about.txt");
         var about = new String(stream.readAllBytes(), StandardCharsets.UTF_8);
   
         InputStream stream2 = cl.getResourceAsStream("/corejava/title.txt");      
         var title = new String(stream2.readAllBytes(), StandardCharsets.UTF_8).strip();
   
         JOptionPane.showMessageDialog(null, about, title, JOptionPane.INFORMATION_MESSAGE, icon);
      }
   }
   ```

### 5.9.4 用反射分析类的能力

1. java.lang.reflect包中有**Field,Method,Constructor三个类**，用来描述类的实例域、方法和构造器。
2. Field 类 getType方法返回描述实例域类型的Class类对象。
3. Method类有方法描述参数和返回值的类型;
4. Constructor类有方法描述参数的类型;
5. 上述三个类都有getModifiers方法，返回整形，用不同的位开关描述public、static等**修饰符**的使用情况。
6. **Modifier类**（也在reflect包中）中的静态方法（isPublic,isPrivate,isFinal）分析getModifiers方法的返回值。
7. Class类中的getFields, getMethods, 和getConstructors方法**返回数组**，元素分别是类的public域、方法、构造器，**包括超类中的public部分**；Class中的getDeclaredFields、getDeclaredMethods和 getDeclaredConstructors方法分别**返回所有**声明的域、方法和构造器组成的数组，**不包括超类中的成员**。

   ```java
      public static void printFields(Class cl)
      {
         Field[] fields = cl.getDeclaredFields();

         for (Field f : fields)
         {
            Class type = f.getType();
            String name = f.getName();
            System.out.print("   ");
            String modifiers = Modifier.toString(f.getModifiers());
            if (modifiers.length() > 0) System.out.print(modifiers + " ");         
            System.out.println(type.getName() + " " + name + ";");
         }
      }
   ```

8. 可以借助上述方法分析java interpreter解释器而已加载的任何类，不仅仅是可编译的类。还能分析Java编译器自动生成的内部类。

### 5.9.5 在运行时使用反射分析对象

1. 查看域中的具体内容。反射可以查看**在编译时还不被知道的对象的域**==>Field类的get()方法

   ```java
   var harry = new Employee("Harry Hacker", 50000, 10, 1, 1989);
   Class cl = harry.getClass();
   // the class object representing Employee
   Field f = cl.getDeclaredField("name");
   // the name field of the Employee class
   Object v = f.get(harry);
   // the value of the name field of the harry object, i.e.,
   // the String object "Harry Hacker"
   ```

2. f.set(obj,value);给obj的f域设置新的值；
3. **private的域不可获得。f.setAccessible(true)方法之后，f.get()就可用了。setAccessible方法是AccessibleObject类的方法，是Field, Method, 和 Constructor三个类的公共超类。**

4. TODO:可变句柄 variable handles VarHandle
