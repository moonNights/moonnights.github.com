---
layout:     post
title:      "Lambda expression"
subtitle:   "迟到的学习"
date:       2018-01-29 12:00:00
author:     "宋军庆"
catalog: true
tags:
    - JAVA8
    - JAVA8
---


学习的事例是根据JAVA [官方教学地址](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) 来的,只是单纯的根据官方实例实现。


<p id="overview"></p>

## 背景

 假如你需要开发一个互联网应用产品，你想要开发一个功能能够管理各种不同的操作，比如发送消息，需要指定成员安全认证。在接下来的各个步骤中我们慢慢的查看对比是怎么演变成lambda表达式的。

 用例描述：

 | 属性|描述|
 |--|--|
 |name|成员名称|
 |primaryActor|角色|
 |preConditions|前置条件|
 |......|各种各样你期待加入的属性方法|

 可以将用例的各个属性转换成以下对象 Person 类

```java

public class Person {

    public enum Sex {
        MALE, FEMALE
    }

    String name;
    LocalDate birthday;
    Sex gender;
    String emailAddress;

    public int getAge() {
        // ...
    }

public void printPerson() {
        // ...
    }
    }
```


## 实现过程

 如果我们想要过滤成员按照年龄来做，比如大于指定年龄的我们可能会编写如下代码

```java

public static void printPersonsOlderThan(List<Person> roster, int age) {
    for (Person p : roster) {
        if (p.getAge() >= age) {
            p.printPerson();
        }
    }
}

```



 如果我们还需要更多的检索方法，如：需要根据一个年龄段做检索，可能在上一个的基础上会添加如下代码,此时我们需要在Person 中添加printPerson方法

```java
public static void printPersonsWithinAgeRange(
    List<Person> roster, int low, int high) {
    for (Person p : roster) {
        if (low <= p.getAge() && p.getAge() < high) {
            p.printPerson();
        }
    }
}
```



假设我们还需要提供 基于性别过滤、基于角色过滤、基于各种属性的组合过滤，不敢想象过滤形参是如何的多、如何的难看,并且代码也是那么的脆弱完全不符合设计的基本原则<b>对扩展开放，对修改关闭</b>，基于以上考虑是否可以将check 部分代码抽象出来如有扩展提供一个检查实例即可。进一步演变。


```java
public static void printPersons( List<Person> roster, CheckPerson tester) {
    for (Person p : roster) {
        if (tester.test(p)) {
            p.printPerson();
        }
    }
}

interface CheckPerson {
    boolean test(Person p);
}

```



在以上的一个设计的情况下假设我们想提供基于角色过滤则仅仅需要提供一个CheckPerson 的实现类，可以这么做减少耦合和修改，遵循设计原则增加代码的稳定性扩展性。

```java
 class CheckRoleService implements CheckPerson {
    public boolean test(Person p) {
        return "Admin".equals( p.role);
    }
}


//调用
printPersons(roster, new CheckRoleService());
```



基于以上的代码也依然有它的脆弱性，我们在扩展一个方法时不得不添加一系列的实现类来实现 CheckPerson 接口，为了避免过多的创建类可以考虑使用[匿名函数](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)来代替创建一个实现类，如下：

```java
printPersons( roster,
    new CheckPerson() {
        public boolean test(Person p) {
            return "Admin".equals( p.role);
        }
    }
);
```



相对于庞大的匿名函数语法来说以上的使用只是其中一种使用方式，从上面一个示例来看`CheckPerson `是一个单纯的`functional interface`，要求只能包含一个[抽象方法](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html),由于一个接口只含有一个抽象方法因此你可以忽略名称来实现，我们考虑使用lambda 表达式来实现以上步骤


```java
	
printPersons(
    roster,
    (Person p) -> "Admin".equals( p.role)
);
```



在上述步骤中我们依然提供了一个`functional interface`,在JDK8中已经给我们提供了类似于CheckPerson 这样的`functional interface`，它们在包`java.util.function`中 `Predicate<T> ` 可以替代 CheckPerson 接口可以演变成以下格式

```java
public static void printPersonsWithPredicate(
    List<Person> roster, Predicate<Person> tester) {
    for (Person p : roster) {
        if (tester.test(p)) {
            p.printPerson();
        }
    }
}

//lambda 调用
printPersonsWithPredicate(
    roster,
    p ->  "Admin".equals( p.role)
);

```




## Lambda 表达式语法

Lambda由以下几部分组成

+ 形参，如(Person p) 类型可以省略
+ 箭头，  ->
+ body  包含代码块或者一个表达式 对于返回使用return 、代码块

(Person p) -> "Admin".equals( p.role)


## Lambda 表达式知晓

+ 参数作用域完全等同于java中方法访问变量
+ 并不能提高并发处理能力
+ 对编码有要求（理解、维护）
+ 配合`java.util.function`中的 functional interface 使用减少代码量


## 参考资料

1. [lambdaexpressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) 
2. [Lambda并行能力测试](http://www.importnew.com/11113.html) 
3. [anonymousclasses](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html) 
4. [abstract](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)
