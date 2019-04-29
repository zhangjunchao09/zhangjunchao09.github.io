---
layout:     post
title:      java 新特性
subtitle:   lambda stream
date:       2019-04-29
author:     ZJC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - lambda
    - Stream
---

# Lambda

## 简介
Lambda表达式（也称闭包），是Java8发布的新特性中最受期待和欢迎的新特性之一。
 
在Java语法层面Lambda表达式允许函数作为一个方法的参数（函数作为参数传递到方法中），
 
或者把代码看成数据。Lambda表达式用于简化Java中接口式的匿名内部类，被称为函数式接口的概念。

## 语法
(args1, args2...)->{};
 
## 函数式接口

就是一个只具有一个抽象方法的普通接口，像这样的接口就可以使用Lambda表达式来简化代码的编写。

并且它们能够被@FunctionalInterface注解注释。

@FunctionalInterface注解不是必须的，但是它能够让工具知道这一个接口是一个函数式接口并表现有意义的行为。

例如，如果你试着这编译一个用@FunctionalInterface注释自己并且含有多个抽象方法的接口，编译就会报出这样一个错Multiple non-overriding abstract methods found。

同样的，如果你给一个不含有任何方法的接口添加@FunctionalInterface注解，会得到如下错误信息,No target method found.


Java8的lambda表达式是否只是一个匿名内部类的语法糖或者函数式接口是如何被转换成字节码的？

答案是NO,Java8不采用匿名内部类的原因主要有两点：

+ + 性能影响: 如果lambda表达式是采用匿名内部类实现的，那么每一个lambda表达式都会在磁盘上生成一个class文件。当JVM启动时，这些class文件会被加载进来，因为所有的class文件都需要在启动时加载并且在使用前确认，从而会导致JVM的启动变慢。
+ + 向后的扩展性: 如果Java8的设计者从一开始就采用匿名内部类的方式，那么这将限制lambda表达式未来的使发展范围。

Java8的设计者决定采用在Java7中新增的动态启用来延迟在运行时的加载策略。当javac编译代码时，它会捕获代码中的lambda表达式并且生成一个动态启用的调用地址(称为lambda工厂）。当动态启用被调用时，就会向lambda表达式发生转换的地方返回一个函数式接口的实例。

## 常见的函数式接口
Comparator
Runnable

java8默认也带有许多可以直接在代码中使用的函数式接口。它们位于java.util.function包中，下面简单介绍几个:

java.util.function.Predicate<T>

此函数式接口是用来定义对一些条件的检查，比如一个predicate。Predicate接口有一个叫test的方法，它需要一个T类型的值，返回值为布尔类型。例如，在一个names列表中找出所有以s开头的name就可以像如下代码这样使用predicate。
```
Predicate<String> namesStartingWithS = name -> name.startsWith("s");
```
java.util.function.Consumer<T>

这个函数式接口用于表现那些不需要产生任何输出的行为。Consumer接口中有一个叫做accept的方法，它需要一个T类型的参数并且没有返回值。例如，用指定信息发送一封邮件：
```
Consumer<String> messageConsumer = message -> System.out.println(message);
```
java.util.function.Function<T,R>
这个函数式接口需要一个值并返回一个结果。例如，如果需要将所有names列表中的name转换为大写，可以像下面这样写一个Function:
```
Function<String, String> toUpperCase = name -> name.toUpperCase();
```
java.util.function.Supplier<T>

这个函数式接口不需要传值，但是会返回一个值。它可以像下面这样，用来生成唯一的标识符
```
Supplier<String> uuidGenerator= () -> UUID.randomUUID().toString();
```
## 原理
在lambda表达式中，我们不需要明确指出参数类型，javac编译器会通过上下文自动推断参数的类型信息。由于我们是在对一个由String类型组成的List进行排序并且compare方法仅仅用一个T类型，所以Java编译器自动推断出两个参数都是String类型。根据上下文推断类型的行为称为类型推断。Java8提升了Java中已经存在的类型推断系统，使得对lambda表达式的支持变得更加强大。javac会寻找紧邻lambda表达式的一些信息通过这些信息来推断出参数的正确类型。

在大多数情况下，javac会根据上下文自动推断类型。假设因为丢失了上下文信息或者上下文信息不完整而导致无法推断出类型，代码就不会编译通过。例如，下面的代码中我们将String类型从Comparator中移除，代码就会编译失败。
```
Comparator comparator = (first, second) -> first.length() - second.length(); // compilation error - Cannot resolve method 'length()'
```
## 示例

示例1 无参无返回：
```
public class Demo01 {

    public static void main(String[] args) {

        // 1.传统方式 需要new接口的实现类来完成对接口的调用
        ICar car1 = new IcarImpl();
        car1.drive();

        // 2.匿名内部类使用
        ICar car2 = new ICar() {
            @Override
            public void drive() {
                System.out.println("Drive BMW");
            }
        };
        car2.drive();

        // 3.无参无返回Lambda表达式
        ICar car3 = () -> {System.out.println("Drive Audi");};
        // ()代表的就是方法，只是匿名的。因为接口只有一个方法所以可以反推出方法名
        // 类似Java7中的泛型 List<String> list = new ArrayList<>(); Java可以推断出ArrayList的泛型。
        car3.drive();

        // 4.无参无返回且只有一行实现时可以去掉{}让Lambda更简洁
        ICar car4 = () -> System.out.println("Drive Ferrari");
        car4.drive();

        // 去查看编译后的class文件 大家可以发现 使用传统方式或匿名内部类都会生成额外的class文件，而Lambda不会
    }
}

interface ICar {
    
    void drive();
}

class IcarImpl implements ICar{
    
    @Override
    public void drive() {
        System.out.println("Drive Benz");
    }
}
```
示例2 有参有返回值：
```
public class Demo02 {

    public static void main(String[] args) {

        // 1.有参无返回
        IEat eat1 = (String thing) -> System.out.println("eat " + thing);
        eat1.eat("apple");

        // 参数数据类型可以省略
        IEat eat2 = (thing) -> System.out.println("eat " + thing);
        eat2.eat("banana");

        // 2.多个参数
        ISpeak speak1 = (who, content) -> System.out.println(who + " talk " + content);
        speak1.talk("xinzong", "hello word");

        // 3.返回值
        IRun run1 = () -> {return 10;};
        run1.run();

        // 4.返回值简写
        IRun run2 = () -> 10;
        run2.run();
    }

}

interface IEat {

    void eat(String thing);
}

interface ISpeak {
    void talk(String who, String content);
}

interface IRun {

    int run();
}
```
示例3 final类型参数：
```
public class Demo03 {

    public static void main(String[] args) {

        // 全写
        IAddition addition1 = (final int a, final int b) -> a + b;
        addition1.add(1, 2);
        // 简写
        IAddition addition2 = (a, b) -> a+b;
        addition2.add(2, 3);
    }

}

interface IAddition {
    
    int add(final int a, final int b);
}
```

示例4 线程实战：
```
public class Demo4 {

    public static void main(String[] args) {
        // 使用Lambda启动线程
        // 1.传统方式使用
        Thread t1 = new Thread(new MyRunnable());
        t1.start();

        // 2.匿名内部类使用
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("t2 running...");
            }
        });
        t2.start();

        // 3.Lambda使用
        Thread t3 = new Thread(() -> System.out.println("t3 running..."));
        t3.start();
        // 更简写法1
        new Thread(() -> System.out.println("t4 running...")).start();
        // 更简写法2
        Process.process(()-> System.out.println("t5 running..."));
    }
}

class MyRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println("t1 running...");
    }
}


interface Process {
    // Java8中允许接口中定义非抽象方法 前提该方法必须为 default 或 static
    static void process(Runnable runnable) {
        new Thread(runnable).start();
    }
}
```
示例5 List.sort()排序：
```
public class Demo5 {

    public static void main(String[] args) {
        // List.sort() 排序
        List<Person> list = new ArrayList<>();
        list.add(new Person(3));
        list.add(new Person(1));
        list.add(new Person(8));
        list.add(new Person(7));
        list.add(new Person(5));
        // 1.传统方式使用 - 升序
        list.sort(new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                // o1和o2比较 1:大于 0:相等 -1:小于
                // 颠倒结果就是降序
                return o1.getHeight().compareTo(o2.getHeight());
            }
        });
        // 2.Lambda使用 - 升序
        list.sort((o1, o2) ->  o1.getHeight().compareTo(o2.getHeight()));
        // 打印结果
        for (Person person : list) {
            System.out.println(person.getHeight());
        }
    }
}

class Person {

    public Person(Integer height){
        this.height = height;
    }

    private Integer height;

    public Integer getHeight() {
        return height;
    }

    public void setHeight(Integer height) {
        this.height = height;
    }
}
```
示例6 List过滤指定元素：
```
public class Demo6 {

    public static void main(String[] args) {
        List<People> list = new ArrayList<>();
        list.add(new People("zs", 22));
        list.add(new People("ls", 27));
        list.add(new People("ww", 29));
        list.add(new People("zl", 21));
        // 集合遍历查找 年龄在20-25岁的人

        List<People> search = new ArrayList<>();
        // 1.传统方式
        for (People people : list) {
            if (people.getAge() >= 20 && people.getAge() <= 25){
                search.add(people);
            }
        }

        // 2.Lambda使用
        // Stream是Java 8 提供的高效操作集合类（Collection）数据的API。
        search = list.stream().filter((People people) -> people.getAge() >= 20 && people.getAge() <= 25).collect(Collectors.toList());

        // 打印结果
        for (People people : search) {
            System.out.println(people.getName() + "-" + people.getAge());
        }
    }

}

class People {

    public People(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    private String name;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```
# stream

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

## stram基本概念

Stream（流）是一个来自数据源的元素队列并支持聚合操作

元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。

数据源 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。

聚合操作 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。

内部迭代： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## 生成流

在 Java 8 中, 集合接口有两个方法来生成流：

stream() − 为集合创建串行流。

parallelStream() − 为集合创建并行流。

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```

## forEach
Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：
```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```
## map
map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：
```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```
## filter
filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：
```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
int count = strings.stream().filter(string -> string.isEmpty()).count();
```
## limit
limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：
```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```
## sorted
sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：
如果流中的元素的类实现了 Comparable 接口，即有自己的排序规则，那么可以直接调用 sorted() 方法对元素进行排序，如 Stream<Integer>

反之, 需要调用 sorted((T, T) -> int) 实现 Comparator 接口
```
根据年龄大小来比较：
list = list.stream()
           .sorted((p1, p2) -> p1.getAge() - p2.getAge())
           .collect(toList());
```
```
list = list.stream()
           .sorted(Comparator.comparingInt(Person::getAge))
           .collect(toList());
```
## 并行（parallel）程序
parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：
```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
int count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```
## Collectors
Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：
```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```
## 统计
另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。
```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```

## distinct()
去除重复元素，这个方法是通过类的 equals 方法来判断两个元素是否相等的
如例子中的 Person 类，需要先定义好 equals 方法，不然类似[Person{name='jack', age=20}, Person{name='jack', age=20}] 这样的情况是不会处理的

## skip(long n)
去除前 n 个元素
```
list = list.stream()
            .skip(2)
            .collect(toList());
```
## flatMap(T -> Stream<R>)
将流中的每一个元素 T 映射为一个流，再把每一个流连接成为一个流
```
List<String> list = new ArrayList<>();
list.add("aaa bbb ccc");
list.add("ddd eee fff");
list.add("ggg hhh iii");

list = list.stream().map(s -> s.split(" ")).flatMap(Arrays::stream).collect(toList());
```
上面例子中，我们的目的是把 List 中每个字符串元素以" "分割开，变成一个新的 List<String>。

首先 map 方法分割每个字符串元素，但此时流的类型为 Stream<String[ ]>，因为 split 方法返回的是 String[ ] 类型；所以我们需要使用 flatMap 方法，先使用Arrays::stream将每个 String[ ] 元素变成一个 Stream<String> 流，然后 flatMap 会将每一个流连接成为一个流，最终返回我们需要的  Stream<String>
## anyMatch(T -> boolean)
流中是否有一个元素匹配给定的 T -> boolean 条件

是否存在一个 person 对象的 age 等于 20：
```
boolean b = list.stream().anyMatch(person -> person.getAge() == 20);
```
## allMatch(T -> boolean)

流中是否所有元素都匹配给定的 T -> boolean 条件

##  noneMatch(T -> boolean)

流中是否没有元素匹配给定的 T -> boolean 条件

##  findAny() 和 findFirst()

findAny()：找到其中一个元素 （使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素）

findFirst()：找到第一个元素

值得注意的是，这两个方法返回的是一个 Optional<T> 对象，它是一个容器类，能代表一个值存在或不存在，这个后面会讲到

##  reduce((T, T) -> T) 和 reduce(T, (T, T) -> T)
用于组合流中的元素，如求和，求积，求最大值等

计算年龄总和：
```
int sum = list.stream().map(Person::getAge).reduce(0, (a, b) -> a + b);
```
与之相同:
```
int sum = list.stream().map(Person::getAge).reduce(0, Integer::sum);
```

其中，reduce 第一个参数 0 代表起始值为 0，lambda (a, b) -> a + b 即将两值相加产生一个新值
同样地：
计算年龄总乘积：
```
int sum = list.stream().map(Person::getAge).reduce(1, (a, b) -> a * b);
```
当然也可以
```
Optional<Integer> sum = list.stream().map(Person::getAge).reduce(Integer::sum);
```
即不接受任何起始值，但因为没有初始值，需要考虑结果可能不存在的情况，因此返回的是 Optional 类型

## Stream 完整实例
将以下代码放入 Java8Tester.java 文件中：
```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.IntSummaryStatistics;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;
import java.util.Map;
 
public class Java8Tester {
   public static void main(String args[]){
      System.out.println("使用 Java 7: ");
        
      // 计算空字符串
      List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
      System.out.println("列表: " +strings);
      long count = getCountEmptyStringUsingJava7(strings);
        
      System.out.println("空字符数量为: " + count);
      count = getCountLength3UsingJava7(strings);
        
      System.out.println("字符串长度为 3 的数量为: " + count);
        
      // 删除空字符串
      List<String> filtered = deleteEmptyStringsUsingJava7(strings);
      System.out.println("筛选后的列表: " + filtered);
        
      // 删除空字符串，并使用逗号把它们合并起来
      String mergedString = getMergedStringUsingJava7(strings,", ");
      System.out.println("合并字符串: " + mergedString);
      List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
        
      // 获取列表元素平方数
      List<Integer> squaresList = getSquares(numbers);
      System.out.println("平方数列表: " + squaresList);
      List<Integer> integers = Arrays.asList(1,2,13,4,15,6,17,8,19);
        
      System.out.println("列表: " +integers);
      System.out.println("列表中最大的数 : " + getMax(integers));
      System.out.println("列表中最小的数 : " + getMin(integers));
      System.out.println("所有数之和 : " + getSum(integers));
      System.out.println("平均数 : " + getAverage(integers));
      System.out.println("随机数: ");
        
      // 输出10个随机数
      Random random = new Random();
        
      for(int i=0; i < 10; i++){
         System.out.println(random.nextInt());
      }
        
      System.out.println("使用 Java 8: ");
      System.out.println("列表: " +strings);
        
      count = strings.stream().filter(string->string.isEmpty()).count();
      System.out.println("空字符串数量为: " + count);
        
      count = strings.stream().filter(string -> string.length() == 3).count();
      System.out.println("字符串长度为 3 的数量为: " + count);
        
      filtered = strings.stream().filter(string ->!string.isEmpty()).collect(Collectors.toList());
      System.out.println("筛选后的列表: " + filtered);
        
      mergedString = strings.stream().filter(string ->!string.isEmpty()).collect(Collectors.joining(", "));
      System.out.println("合并字符串: " + mergedString);
        
      squaresList = numbers.stream().map( i ->i*i).distinct().collect(Collectors.toList());
      System.out.println("Squares List: " + squaresList);
      System.out.println("列表: " +integers);
        
      IntSummaryStatistics stats = integers.stream().mapToInt((x) ->x).summaryStatistics();
        
      System.out.println("列表中最大的数 : " + stats.getMax());
      System.out.println("列表中最小的数 : " + stats.getMin());
      System.out.println("所有数之和 : " + stats.getSum());
      System.out.println("平均数 : " + stats.getAverage());
      System.out.println("随机数: ");
        
      random.ints().limit(10).sorted().forEach(System.out::println);
        
      // 并行处理
      count = strings.parallelStream().filter(string -> string.isEmpty()).count();
      System.out.println("空字符串的数量为: " + count);
   }
    
   private static int getCountEmptyStringUsingJava7(List<String> strings){
      int count = 0;
        
      for(String string: strings){
        
         if(string.isEmpty()){
            count++;
         }
      }
      return count;
   }
    
   private static int getCountLength3UsingJava7(List<String> strings){
      int count = 0;
        
      for(String string: strings){
        
         if(string.length() == 3){
            count++;
         }
      }
      return count;
   }
    
   private static List<String> deleteEmptyStringsUsingJava7(List<String> strings){
      List<String> filteredList = new ArrayList<String>();
        
      for(String string: strings){
        
         if(!string.isEmpty()){
             filteredList.add(string);
         }
      }
      return filteredList;
   }
    
   private static String getMergedStringUsingJava7(List<String> strings, String separator){
      StringBuilder stringBuilder = new StringBuilder();
        
      for(String string: strings){
        
         if(!string.isEmpty()){
            stringBuilder.append(string);
            stringBuilder.append(separator);
         }
      }
      String mergedString = stringBuilder.toString();
      return mergedString.substring(0, mergedString.length()-2);
   }
    
   private static List<Integer> getSquares(List<Integer> numbers){
      List<Integer> squaresList = new ArrayList<Integer>();
        
      for(Integer number: numbers){
         Integer square = new Integer(number.intValue() * number.intValue());
            
         if(!squaresList.contains(square)){
            squaresList.add(square);
         }
      }
      return squaresList;
   }
    
   private static int getMax(List<Integer> numbers){
      int max = numbers.get(0);
        
      for(int i=1;i < numbers.size();i++){
        
         Integer number = numbers.get(i);
            
         if(number.intValue() > max){
            max = number.intValue();
         }
      }
      return max;
   }
    
   private static int getMin(List<Integer> numbers){
      int min = numbers.get(0);
        
      for(int i=1;i < numbers.size();i++){
         Integer number = numbers.get(i);
        
         if(number.intValue() < min){
            min = number.intValue();
         }
      }
      return min;
   }
    
   private static int getSum(List numbers){
      int sum = (int)(numbers.get(0));
        
      for(int i=1;i < numbers.size();i++){
         sum += (int)numbers.get(i);
      }
      return sum;
   }
    
   private static int getAverage(List<Integer> numbers){
      return getSum(numbers) / numbers.size();
   }
}
```