# 函数式接口

## 1. 概念

*函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。*
*函数式接口可以被隐式转换为 lambda 表达式。*
*Lambda 表达式和方法引用（实际上也可认为是* *Lambda表达式* *）上。*

## 2. @FunctionalInterface

![](https://scnek46ixq9t.feishu.cn/space/api/box/stream/download/asynccode/?code=NDY3OTA5M2JmZTRkZjQ5ZTVkMjM2ZjhhMDE1NzMyOTVfZFJIajk0SlVxSU9VT2RuTkZ5UjJUdmtGWHllbGg0d3FfVG9rZW46UDdkRWJzVlJybzZnMVJ4UnRLcmNsd0tsbkM2XzE3MzM4Mzc5OTA6MTczMzg0MTU5MF9WNA)

像我们平常在创建线程时用到的Runnable接口，在他里面只有一个抽象方法run（），因此该接口就是上面提到的函数式接口，注意它的左上角有一个@FunctionalInterface的注解。

这个注解的作用是强制你的接口只有一个抽象方法。如果有多个话直接会报错。

这里当你写了第二个方法时，编译就无法通过，idea甚至在编码阶段就行了提示。

## 3. 函数式接口的使用方式

首先我们自定义一个函数式接口如下：

```Java
package Test;
@FunctionalInterface
public interface TestInterface {
    public abstract void singleMethod();

//    void multipleMethod();
}
```

然后定义一个Test封装类

```Java
public class Test {
    public void testMethod(TestInterface testInterface) {
        System.out.println("即将执行函数式接口方法");
        testInterface.singleMethod();
    }

    public static void main(String[] args) {
/**
 * 测试
 */
Test test = new Test();
        test.testMethod(new TestInterface() {
            @Override
            public void singleMethod() {
                System.out.println("执行函数式接口方法");
            }
        });
        }
        }
```

在这个类中我们写了testMethod方法，将TestInterFace当作参数传入，并打印一句话

可见执行结果如下：

> 即将执行函数式接口外部定义方法
> 执行函数式接口定义方法

根据上面的使用模式，我们会发现jdk中原本就给我们配置许多种函数式接口的使用，像这种函数式接口和lambda表达式的使用是jdk8之后的一个新的特性，因此比如jdk8中更新的stream流对象中的很多方法都需要用到相关知识

在这里我们举一个创建线程的例子如下

```Java
new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("方法执行了");
        }
    }).start();


    /**
     * 结合lambda表达式
     */
new Thread(()->{
        System.out.println("run方法被运行了");
    }).start();

}
```

对于上述表达，第一种我们用的是匿名内部类的方法去实现函数式接口，而第二种用的就是jdk8中所更新的lambda表达式，可见使用之后的代码要相对于之前简短了很多，像jdk中一些原本封装好的方法也是使用了lambda表达式的这种方式。因此lambda表达式的掌握有利于一些源码的阅读。

# 常用函数式接口

## Java封装函数式接口

> java.lang.Runnable,
> 
> java.awt.event.ActionListener,
> 
> java.util.Comparator,
> 
> java.util.concurrent.Callable
> 
> java.util.function包下的接口，如Consumer、Predicate、Supplier等

其实，jdk中给我们提供了很多的函数式接口，我们平时都会用到，只不过大家没有注意到而已，这里我结合实际代码讲解几个常用的函数式接口。想想大家平时常常用到stream流的各种方法来处理list。我看去stream类中看一下它提供的方法。可以看到有几个出镜率较高的函数式接口

* Supplier
* Comsumer
* Predicate
* Function

![](https://scnek46ixq9t.feishu.cn/space/api/box/stream/download/asynccode/?code=OTk1YTQzZDQzNmUzODBiMTg4ZGMwMzU2MDEzYjU3MGVfVXRCVDZHbnVYQjFmQjZqVHBabnRBVklnSU5hbThUdW9fVG9rZW46UnJRa2JGcWhIbzFTdkV4NTB2TmNKT3JobktmXzE3MzM4Mzc5OTA6MTczMzg0MTU5MF9WNA)

## 1. Supplier

**Supplier< T >：包含一个无参的方法**

T get()：获得结果

该方法不需要参数，他会按照某种实现逻辑（由Lambda表达式实现）返回一个数据

Supplier< T >接口也被称为生产型接口，如果我们指定了接口的泛型是什么类型，那么接口中的get方法就会产生什么类型的数据供我们使用

![](https://scnek46ixq9t.feishu.cn/space/api/box/stream/download/asynccode/?code=MTdjMTIwOWFmYmRjMTI2NzEwOTM1ZmZiYzBlNWRjNDRfTDNkR2JBRUduRXdiVUNoMUpWY3Z3bXVrYmxaazltNjVfVG9rZW46RWNqdGJ4UFJ6b2x6Ykl4WWVldGN4eDQwbnFkXzE3MzM4Mzc5OTA6MTczMzg0MTU5MF9WNA)

![](https://scnek46ixq9t.feishu.cn/space/api/box/stream/download/asynccode/?code=YTdjZjNlYWQxMGU1NWZlZGE2NTQ0NDRmNjVhY2U0NWZfaE4zdkRNck0zRXF6Q0Z0UXhmNnlBVHdUMXdIbElpdFhfVG9rZW46RGc1S2JRUTZVb0p6UWp4TW9kNmNyU0gybkdmXzE3MzM4Mzc5OTA6MTczMzg0MTU5MF9WNA)

Supplier接口的get方法没有入参，返回一个泛型T对象。我们来看几个使用 的实例。先写一个实验对象food，我们定义一个方法，创建对象，我们使用lambda的方式来调用并创建对象**。**

```Java
Food food1 = Food.create(Food::new);
Food food2 = Food.create(()-> new Food("小面包",33d,1700d));
Food food = null;
//orElseGet
Food food3 = Optional.ofNullable(food).orElseGet(Food::new);
```

需要注意的是，我每调用一次get方法，都会重新创建一个对象。orElseGet方法入参同样也是Supplier，这里对food实例进行判断，通过Supplier实现了懒加载，也就是说只有当判断food为null时，才会通过orElseGet方法创建新的对象并且返回。不得不说这个设计是非常的巧妙。

## 2. Consumer

**Consumer< T >：包含两个方法**

void accept(T t)：对给定的参数执行此操作

default Consumer < T > andThen(Consumer after)：返回一个组合的Consumer，依次执行此操作，然后执行after操作

Consumer< T >接口也被称为消费型接口，它消费的数据类型由泛型指定

```Java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

可以看到accept方法接受一个对象，没有返回值。那么我们来实战,首先使用Lambda表达式声明一个Supplier的实例，它是用来创建Food实例；再使用Lambda表达式声明一个Consumer的实例，它是用于打印出Food实例的toString信息；最后Consumer消费了Supplier生产的Food。我们常用的forEach方法入参也是Consumer。使用代码如下:

```Java
Supplier<Food> supplier1 = ()-> new Food("ld",33d,1700d);
        Consumer<Food> consumer = (Food g)->{
            System.out.println(g.toString());
        };
        consumer.accept(supplier1.get());
        ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(1, 2, 3, 4, 5));
        list.stream().forEach(System.out::println);
//        forEach(System.out::println);

        Consumer<Integer> print = x -> System.out.println("Number: " + x);
        Consumer<Integer> printSquare = x -> System.out.println("Square: " + (x * x));

        Consumer<Integer> combined = print.andThen(printSquare);
        combined.accept(5);
```

## 3. Predicate

**Predicate< T >：常用的四个方法**

boolean test(T t)：对给定的参数进行判断（判断逻辑由Lambda表达式实现），返回一个布尔值

default Predicate< T > negate()：返回一个逻辑的否定，对应逻辑非

default Predicate< T > and()：返回一个组合判断，对应短路与

default Predicate< T > or()：返回一个组合判断，对应短路或

isEqual()：测试两个参数是否相等

Predicate< T >：接口通常用于判断参数是否满足指定的条件

test(T t) 、negate()

```Java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
boolean test(T t);

default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }


default Predicate<T> negate() {
        return (t) -> !test(t);
    }

default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }


static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
: object -> targetRef.equals(object);
    }

@SuppressWarnings("unchecked")
    static <T> Predicate<T> not(Predicate<? super T> target) {
        Objects.requireNonNull(target);
        return (Predicate<T>)target.negate();
    }
}
```

可以看到test方法接受一个对象，返回boolean类型，这个函数显然是用来判断真假。那么我们用这个接口来构造一个判断食物条件的示例，还有我们常用的stream流中，filter的入参也是Predicate,代码如下:

```Java
Supplier<Food> supplier2 = ()-> new Food("小零食",33d,1700d);
Predicate<Food> food1 = (Food g)-> Objects.equals(g.getSize(),36d);
Predicate<Food> food2 = (Food g)-> Objects.equals(g.getSize(),33d);
boolean test33 = food2.test(supplier2.get());
boolean test36 = food1.test(supplier2.get());
System.out.println(supplier2.get().getName() +"是否为[36] ："+test36);
System.out.println(supplier2.get().getName() +"是否为[33] ："+test33);
```

```Java
ArrayList<Food> list1 = new ArrayList<>();
list1.add(new Food("1", 33d, 2000d));
list1.add(new Food("2", 36d, 3000d));
list1.add(new Food("3", 28d, 1500d));
list1.add(new Food("4", 31d, 1800d));
Predicate<Food> greaterThan30 = (Food g)-> g.getSize()>=30d;
list1.stream().filter(greaterThan30d).forEach(System.out::println);
```

## 4. Function

**Runction<T,R>：常用的两个方法**

R apply(T t)：将此函数应用于给定的参数

default< V >：Function andThen(Function after)：返回一个组合函数，首先将该函数应用于输入，然后将after函数应用于结果

Function<T,R>：接口通常用于对参数进行处理，转换（处理逻辑由Lambda表达式实现），然后返回一个新值

```Java
@FunctionalInterface
public interface Function<T, R> {

R apply(T t);

default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

这个看到apply接口接收一个泛型为T的入参，返回一个泛型为R的返回值，所以它的用途和Supplier还是略有区别。还是一样我们举个例子用代码来说明:

```Java
Supplier<Food> supplier = ()-> new Food("5",33d,1700d);
Function<Food,String> foodMark = (Food g)-> g.getName()+"的含量是"+g.getSize()+"，每次利润"+g.getPrice()+"。";
System.out.println(foodMark.apply(supplier.get()));
```

我们可以看到funtion接口接收Food对象实例，对food的成员变量进行拼接，返回food的描述信息。其基本用法就是如此。

# 总结

* 通过使用函数式接口和 Lambda 表达式，可以用更简洁、易读的方式表达行为，减少代码冗余。
* 函数式接口可以作为参数传递给方法，这使得代码更加灵活，可以在运行时改变行为。
* Java 8 引入的 Stream API 大量依赖于函数式接口，允许对集合进行更高效、更易读的数据操作，例如通过 `filter`、`map` 和 `reduce` 等操作。
* 函数式接口可以与方法引用结合使用，提供了一种方便的方式来扩展现有类的功能。
* 函数式编程风格通常与不可变性和无状态操作相关，这样的代码更容易进行单元测试。
