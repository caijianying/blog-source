---
title: java8系列专栏之Lambda与Stream API
tags:
  - 笔记
originContent: ''
categories:
  - java
toc: false
date: 2019-04-29 20:14:42
---

## 前言
写这篇文章的目的是为了记录一下学习笔记，其次为了能够在复习的时候快速掌握相关知识。本篇记录java8系列专栏之Lambda与Stream API
## 正文
### Lambda
#### 什么是Lambda

```
List<String>list = Arrays.asList("a","c","b");
Collections.sort(list, (o1,o2)->o1.compareTo(o2));
```
以下方式都是常用的lambda使用方式
```
str->str.toLowerCase()
(o1,o2)->o1.compareTo(o2)
(o1,o2)->{return o1.compareTo(o2)}
(String o1,String o2)->{return o1.compareTo(o2)}
```
#### 怎么用，哪里用
函数接口声明既可使用。例如Runnable，Comparator都是函数接口。用@FunctionalInterface声明的都是函数接口
#### 实现原理
首先需要明确说明的是lambda没有使用匿名内部类去实现。

Java 8设计人员决定使用在Java 7中添加的`invokedynamic`指令来推迟在运行时的翻译策略。当javac编译代码时，它捕获lambda表达式并生成`invokedynamic`调用站点(称为lambda工厂)。调用`invokedynamic`调用站点时，返回一个函数接口实例，lambda将被转换到这个函数接口


使用invokedynamic指令，运行时调用LambdaMetafactory.metafactory动态的生成内部类，实现了接口，内部类里的调用方法块并不是动态生成的，只是在原class里已经编译生成了一个静态的方法，内部类只需要调用该静态方法

[参考1](https://github.com/shekhargulati/java8-the-missing-tutorial/blob/master/02-lambdas.md#using-invokedynamic)
[参考2](https://blog.csdn.net/raintungli/article/details/54910152)
#### 内置函数接口

| 函数式接口 | 函数描述符 | 原始类型特化 |
|:-----:|:--------|:-------|
| `Predicate<T>` | `T->boolean` | `IntPredicate,LongPredicate, DoublePredicate` |
| `Consumer<T>` | `T->void` | `IntConsumer,LongConsumer, DoubleConsumer` |
| `Function<T,R>` | `T->R` | `IntFunction<R>, IntToDoubleFunction,` <br/> `IntToLongFunction, LongFunction<R>,` <br/> `LongToDoubleFunction, LongToIntFunction, ` <br/> `DoubleFunction<R>, ToIntFunction<T>, ` <br/> `ToDoubleFunction<T>, ToLongFunction<T>` |
| `Supplier<T>` | `()->T` | `BooleanSupplier,IntSupplier, LongSupplier, DoubleSupplier` |
| `UnaryOperator<T>` | `T->T` | `IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator` |
| `BinaryOperator<T>` | `(T,T)->T` | `IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator` |
| `BiPredicate<L,R>` | `(L,R)->boolean` |  |
| `BiConsumer<T,U>` | `(T,U)->void` | `ObjIntConsumer<T>, ObjLongConsumer<T>, ObjDoubleConsumer<T>` |
| `BiFunction<T,U,R>` | `(T,U)->R` | `ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>` |

### Stream API

| 操作 | 类型 | 返回类型 | 使用的类型/函数式接口 | 函数描述符 |
|:-----:|:--------|:-------|:-------|:-------|
| `filter` | 中间 | `Stream<T>` | `Predicate<T>` | `T -> boolean` |
| `distinct` | 中间 | `Stream<T>` |  |  |
| `skip` | 中间 | `Stream<T>` | long |  |
| `map` | 中间 | `Stream<R>` | `Function<T, R>` | `T -> R` |
| `flatMap` | 中间 | `Stream<R>` | `Function<T, Stream<R>>` | `T -> Stream<R>` |
| `limit` | 中间 | `Stream<T>` | long  | |
| `sorted` | 中间 | `Stream<T>` | `Comparator<T>` | `(T, T) -> int` |
| `anyMatch` | 终端 | `boolean` | `Predicate<T>` | `T -> boolean` |
| `noneMatch` | 终端 | `boolean` | `Predicate<T>` | `T -> boolean` |
| `allMatch` | 终端 | `boolean` | `Predicate<T>` | `T -> boolean` |
| `findAny` | 终端 | `Optional<T>` | |  |
| `findFirst` | 终端 | `Optional<T>` | |  |
| `forEach` | 终端 | `void` | `Consumer<T>` | `T -> void` |
| `collect` | 终端 | `R` | `Collector<T, A, R>` |  |
| `reduce` | 终端 | `Optional<T>` | `BinaryOperator<T>` | `(T, T) -> T` |
| `count` | 终端 | `long` | | |

### Collector 收集
**Collectors 类的静态工厂方法**

| 工厂方法 | 返回类型 | 用途 | 示例 |
|:-----:|:--------|:-------|:-------|
| `toList` | `List<T>` | 把流中所有项目收集到一个 List | `List<Project> projects = projectStream.collect(toList());` |
| `toSet` | `Set<T>` | 把流中所有项目收集到一个 Set，删除重复项 | `Set<Project> projects = projectStream.collect(toSet());` |
| `toCollection` | `Collection<T>` | 把流中所有项目收集到给定的供应源创建的集合 | `Collection<Project> projects = projectStream.collect(toCollection(), ArrayList::new);` |
| `counting` | `Long` | 计算流中元素的个数 | `long howManyProjects = projectStream.collect(counting());` |
| `summingInt` | `Integer` | 对流中项目的一个整数属性求和 | `int totalStars = projectStream.collect(summingInt(Project::getStars));` |
| `averagingInt` | `Double` | 计算流中项目 Integer 属性的平均值 | `double avgStars = projectStream.collect(averagingInt(Project::getStars));` |
| `summarizingInt` | `IntSummaryStatistics` | 收集关于流中项目 Integer 属性的统计值，例如最大、最小、 总和与平均值 | `IntSummaryStatistics projectStatistics = projectStream.collect(summarizingInt(Project::getStars));` |
| `joining` | `String` | 连接对流中每个项目调用 toString 方法所生成的字符串 | `String shortProject = projectStream.map(Project::getName).collect(joining(", "));` |
| `maxBy` | `Optional<T>` | 按照给定比较器选出的最大元素的 Optional， 或如果流为空则为 Optional.empty() | `Optional<Project> fattest = projectStream.collect(maxBy(comparingInt(Project::getStars)));` |
| `minBy` | `Optional<T>` | 按照给定比较器选出的最小元素的 Optional， 或如果流为空则为 Optional.empty() | `Optional<Project> fattest = projectStream.collect(minBy(comparingInt(Project::getStars)));` |
| `reducing` | 归约操作产生的类型 | 从一个作为累加器的初始值开始，利用 BinaryOperator 与流中的元素逐个结合，从而将流归约为单个值 | `int totalStars = projectStream.collect(reducing(0, Project::getStars, Integer::sum));` |
| `collectingAndThen` | 转换函数返回的类型 | 包含另一个收集器，对其结果应用转换函数 | `int howManyProjects = projectStream.collect(collectingAndThen(toList(), List::size));` |
| `groupingBy` | `Map<K, List<T>>` | 根据项目的一个属性的值对流中的项目作问组，并将属性值作 为结果 Map 的键 | `Map<String,List<Project>> projectByLanguage = projectStream.collect(groupingBy(Project::getLanguage));` |
| `partitioningBy` | `Map<Boolean,List<T>>` | 根据对流中每个项目应用断言的结果来对项目进行分区 | `Map<Boolean,List<Project>> vegetarianDishes = projectStream.collect(partitioningBy(Project::isVegetarian));` |
## 参考
1. [Lambdas](https://github.com/shekhargulati/java8-the-missing-tutorial/blob/master/02-lambdas.md#lambdas-)
2. [Lambda expressions](https://github.com/winterbe/java8-tutorial#lambda-expressions)
3. [Java 8 动态类型语言Lambda表达式实现原理分析](https://blog.csdn.net/raintungli/article/details/54910152)
4. [Stream](https://github.com/biezhi/learn-java8/blob/master/java8-stream/README.md#stream)