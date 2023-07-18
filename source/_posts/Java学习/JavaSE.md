---
title: JavaSE
date: 2023-07-16 10:31:37
mathjax: true
categories: Java学习
tags: 
- Java
---
# 1 Java集合框架

<!--more-->

## 1.1 总体介绍

Java集合大致可以分为两大体系，一个是`Collection`，另一个`Map`：

- `Collection`：主要由`List`、`Set`以及`Queue`接口组成。
  - `List`：代表有序、可重复的集合。
  - `Set`：代表无序、不可重复的集合。
  - `Queue`：提供了基于队列的集合体系。
- `Map`：代表具有映射关系的键值对集合。

![](JavaSE/fig1.jpg)

![](JavaSE/fig2.jpg)

## 1.2 List集合

`List`集合的特点就是存取有序，可以存储重复的元素，可以用下标进行元素的操作。

`List`的主要实现类有`ArrayList`、`LinkedList`、`Vector`以及`Stack`：

- `ArrayList`
  - 基于动态数组实现，需要分配连续的内存空间。
  - 支持随机访问，可直接通过元素下标获取元素。
  - 由于插入与删除操作需要移动数组元素，所以插入删除效率较低。
  - 当数组满时，会进行扩容（使用`copyOf`实现）。
- `LinkedList`
  - 基于双向链表实现，不需要分配连续的内存空间。
  - 不支持随机访问，每次查询都只能从一端开始遍历，直到找到查询的对象。
  - 插入与删除只需要变更指针指向即可，效率较高。
  - 由于需要记录前驱以及后继指针，因此内存占用较高。
- `Vector`
  - 与`ArrayList`类似，但`Vector`是线程安全的。
  - 由于`Vector`的线程安全实现仅仅是在每个方法上面添加了`synchronized`关键字，因此效率低，如今已经很少使用了。
- `Stack`
  - `Stack`是`Vector`的子类，不同的是，它的数据结构是先进后出，也就是栈。
  - `Stack`同样是线程安全的，同样效率低，不推荐使用，建议使用`Deque`双端队列来替代`Stack`。

## 1.3 Map集合

`Map`中保存的是键值对，键要求唯一、不可重复，值可以重复，无下标。

`Map`的主要实现类有`HashMap`、`LindedHashMap`、`TreeMap`：

- `HashMap`：基于哈希表实现，查找对象时通过哈希函数计算其位置。
  - 由于使用的是哈希表存储元素，所以无法保证数据的输入顺序与输出顺序一致。
  - 使用链地址法解决哈希冲突，当链表长度大于8并且哈希表长度大于64时，将链表转化为红黑树。
- `LinkedHashMap`：底层原理与`HashMap`类似，只是内部多了一个双向链表，用来维护插入顺序或者LRU顺序。默认为按照插入顺序排序。
- `TreeMap`：基于红黑树实现，每个key-value都作为红黑树的一个结点。可以使用自然排序（默认升序）或者自定义比较器排序。

## 1.4 Set集合

`Set`集合的特点是存取无序、元素不重复、无下标。

`Set`的主要实现类有`HashSet`、`LinkedHashSet`和`TreeSet`。

`Set`的这几个实现类都是基于`Map`的几个实现类所实现的，key存储`Set`的元素，value置为`null`。

## 1.5 Queue集合

`Queue`是一个队列集合，队列通常是指先进先出（FiFO）的容器。新元素插入到队列的尾部，访问元素操作会返回队列头部的元素。通常，队列不允许随机访问队列的元素。

`Queue`主要实现类有`ArrayDeque`、`LinkedList`、`PriorityQueue`：

- `ArrayQueue`是一个基于数组实现的双端队列，既然是双端队列，那么即可以先进先出（FIFO队列的特性），也可以先进后出（栈的特性）。
- `LinkedList`是`List`接口的实现类，也是`Deque`（双端队列）的实现类，底层是双向链表的数据结构。
- `PriorityQueue`基于堆实现，该类能够对存储在其中的元素按照元素大小进行排序。

**参考资料**：

[【集合系列】- 初探 java 集合框架图](http://www.justdojava.com/2019/09/16/java-collection-1/)

[Java集合框架史上最详解（list set 以及map）](https://chasing987.github.io/2020/12/13/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%20list-set-map%20/)

# 2 Java函数式接口

## 2.1 概述

JDK8之后新增了`java.util.function`包，该包下都是函数式接口，其中最为基础的有四大接口：`Consumer`、`Supplier`、`Predicate`、`Function`。这四大接口的总结如下：

|    **接口**     | **参数** | **返回值** |                           **说明**                           |
| :-------------: | :------: | :--------: | :----------------------------------------------------------: |
|  `Consumer<T>`  |   `T`    |     无     |   消费型：用来接收一个泛型`T`对象并执行相关操作，无返回值    |
|  `Supplier<T>`  |    无    |    `T`     |           供给型：无参，用于生成泛型`T`对象并返回            |
| `Predicate<T>`  |   `T`    | `boolean`  | 谓词型：用于判断泛型`T`对象是否符合条件，符合返回`ture`，否则返回`false` |
| `Function<T,R>` |   `T`    |    `R`     |  函数型：接收一个泛型T对象的参数，产生泛型R对象的结果并返回  |

除此之外，`java.util.function`包下还有许多这四种基础接口的变种，不够在理解了这四种接口之后，其他的自然也就理解了。

## 2.2 Consumer接口

`Consumer`接口的源码如下：

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * 消费逻辑
     */
    void accept(T t);

    /**
     * 用于组合Consumer，相当于串联，并返回经过组合之后的Consumer
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
    
}
```

`Iterable`接口中的`forEach`方法就使用到了`Consumer`接口，相关代码如下：

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        // 执行消费逻辑
        action.accept(t);
    }
}
```

因此，我们可以向`forEach`中传入自己的处理逻辑，来对其中的每个元素都进行相应的处理。以遍历打印元素为例：

```
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
list.forEach(System.out::println);
```

注意，方法引用或者lambda表达式都可以看做实现了函数式接口的匿名内部类。

## 2.3 Supplier接口

`Supplier`接口的源码如下：

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * 获取泛型T对象
     */
    T get();
}
```

`Optional`对象中的`orElseGet`方法中就使用到了`Supplier`接口，对应代码如下：

```java
/**
 * 如果value值存在，则返回该值，否则返回Supplier生成的结果
 */
public T orElseGet(Supplier<? extends T> supplier) {
    return value != null ? value : supplier.get();
}
```

## 2.4 Predicate接口

`Predicate`接口的源码如下：

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * 具体判断逻辑，用于判断泛型T对象是否符合要求，可以理解为条件A
     */
    boolean test(T t);

    /**
     * 对当前断言进行“与”操作，如果将other视为条件B，
     * 则该方法可理解为 条件A && 条件B
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * 对当前断言进行“取非”操作，可以理解为 !条件A
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * 对当前断言进行“或”操作，如果将other视为条件B，
     * 则该方法可理解为 条件A || 条件B
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * 对当前断言进行“取等”操作，如果将other视为条件B，
     * 则该方法可理解为 条件A == 条件B
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

`Predicate`接口多用于`filter`方法中，用于筛选符合条件的元素，最为常见的就是Stream流中的`filter`方法。

## 2.5 Function接口

`Function`接口源码如下：

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * 转换逻辑
     */
    R apply(T t);

    /**
     * 组合方法，会在调用当前Function之前调用before中的代码逻辑
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * 组合方法，会在调用当前Function之后调用after中的代码逻辑
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * 静态方法，返回输入参数
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

Stream流中的`map`方法就用到了`Function`接口。