---
layout: post
title: "Java8新特性之Lambda表达式"
date: 2016-07-02 23:12:12 +0800
comments: true
categories: tools java
---

写久了java，总是给人一种很啰说的感觉。最近使用java 8以后，这个感觉明显变了。Java 8中增加了几个非常有用的特性，像Stream，Lambda表达式，以及函数的引用。这些新的特性，使得java的代码更加紧凑，也更容易理解。更短的代码意味着，我们只要做很少的修改就能完成很多新的问题。

#### Lambda 表达式

Java 的tutorial上给了一个很好的例子， 我们就用这个例子来介绍Lambda表达式吧。

##### 1. 简单地有一个Person类

```
public class Person {

 public enum Sex {
   MALE, FEMALE
 }

 String name;
 LocalDate birthday;
 Sex gender;
 String emailAddress;
 int age;

 public int getAge() {
   return age;
 }

 public void printPerson() {
   System.out.println(toString());
 }
 
 @Override
 public String toString() {
   return "Name :" + name + " " +
           "Birthday :" + birthday + " " +
           "Gender :" + gender + " " +
           "Email Address :" + emailAddress + " " +
           "Age :" + age;
 }
```

这个类很简单，有名字，生日，年龄和email，函数也就是打印而已。

如果我们想打印所有年龄大于一定数值的，可以这么写：

```
public static void printPersonsOlderThan(List<Person> roster, int age) {
    for (Person p : roster) {
        if (p.getAge() >= age) {
            p.printPerson();
        }
    }
}
```
这个函数很好写，但是如果我想改一下要求，打印一定年龄段的所有的人，那么我们就需要改一下。

##### 2. 稍微再通用化一点
```
public static void printPersonsWithinAgeRange(
    List<Person> roster, int low, int high) {
    for (Person p : roster) {
        if (low <= p.getAge() && p.getAge() < high) {
            p.printPerson();
        }
    }
}
```
通过代入参数，我们的函数可以稍微的好了一点。但是呢，这个还是只能操作年龄。如果我们想换一个条件，那么就麻烦了。


##### 3. 使用Local class

那么好吧，这个时候我们能想到的是定义一个接口。类似Comparable那种，然后自己去给出具体的实现。

```
interface CheckPerson {
 boolean test(Person p);
}

class CheckPersonEligibleForSelectiveService implements CheckPerson {
 public boolean test(Person p) {
   return p.gender == Person.Sex.MALE &&
           p.getAge() >= 18 &&
           p.getAge() <= 25;
 }
}

public static void printPersons(List<Person> roster, CheckPerson tester) {
 for (Person p : roster) {
   if (tester.test(p)) {
     p.printPerson();
   }
 }
}

```


##### 4. 使用Anonymous Class

通过上面的接口，如果我想进一步优化代码的话，那么我们可以使用Anonymous class.

```
printPersons(
    roster,
    new CheckPerson() {
        public boolean test(Person p) {
            return p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25;
        }
    }
);
```

Anonymous Class使我们可以不用再定义类，而是直接重写相应的函数就好了。这个基本上就是Java 8 之前的最好的办法了吧。但是呢，我们发现，我们还是重复地写了很多没有用的code。每一次都要写test的定义，难道编译器就不能帮我们把这个部分infer出来吗？这样我们不是省了很多的事。嗯，接下来就是Lambda表达式出场了。

##### 5. 使用Lambda表达式

```
printPersons(
       roster,
       (Person p) -> p.getGender() == Person.Sex.MALE
               && p.getAge() >= 18
               && p.getAge() <= 25
);

```

这个就是Lambda表达式啦。我们只要写好函数的主体部分，其他的部分编译器自动的会帮我们做好infer。那我们在什么情况下，才可以使用Lambda表达式呢？CheckPerson是一个functional interface，对于functional interface，我们就可以使用Lambda表达式了。那什么是functional interface呢？就是只有一个abstract method的interface。注意，他是可以有其他的default或者static的方法的。

##### 6. 使用标准的Functional Interface

```
interface Predicate<T> {
    boolean test(T t);
}

public static void printPersonsWithPredicate(
    List<Person> roster, Predicate<Person> tester) {
    for (Person p : roster) {
        if (tester.test(p)) {
            p.printPerson();
        }
    }
}
```

像CheckPerson这么简单的functional interface，JDK已经帮我们预先写好了一个，就是Predict接口。在我们使用的时候，可以直接使用Predict就好了。

##### 7. 写更多的Lambda表达式

```
public static void processPersons(
    List<Person> roster,
    Predicate<Person> tester,
    Consumer<Person> block) {
        for (Person p : roster) {
            if (tester.test(p)) {
                block.accept(p);
            }
        }
}
processPersons(
     roster,
     p -> p.getGender() == Person.Sex.MALE
         && p.getAge() >= 18
         && p.getAge() <= 25,
     p -> p.printPerson());
```
通过上面的方法，我们不仅可以使用printPerson函数，我们甚至可以使用Person中任意一个函数。更进一步，我们先通过map的方式，将Person对象转化成为另外一个string对象。然后再对map的对象进行操作。
```
public static void processPersonsWithFunction(
    List<Person> roster,
    Predicate<Person> tester,
    Function<Person, String> mapper,
    Consumer<String> block) {
    for (Person p : roster) {
        if (tester.test(p)) {
            String data = mapper.apply(p);
            block.accept(data);
        }
    }
}
processPersonsWithFunction(
    roster,
    p -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25,
    p -> p.getEmailAddress(),
    email -> System.out.println(email)
);
```

##### 8.使用泛型编程
看到上面的定义，我们就会嘀咕，如果我们map出来的对象不是string怎么办？假如我想使用的函数不是println怎么办？那我们是不是又要重写啦？这个时候，我们又想到了泛型编程这个老话题。Lambda表达式+泛型编程，简直完美。
```
public static <X, Y> void processElements(
    Iterable<X> source,
    Predicate<X> tester,
    Function <X, Y> mapper,
    Consumer<Y> block) {
    for (X p : source) {
        if (tester.test(p)) {
            Y data = mapper.apply(p);
            block.accept(data);
        }
    }
}
processElements(
    roster,
    p -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25,
    p -> p.getEmailAddress(),
    email -> System.out.println(email)
);
```
##### 9. 终极完全版

其实上面的一切都是假的。忘记吧。我们直接使用stream就好了。不过通过上面这样一轮迭代，我们知道了stream到底是如何实现的。这样我们在使用的时候也更加地明白。我相信在我们自己定义的时候，我们也能够定义出更加灵活的Lambda表达式。

```
roster
    .stream()
    .filter(
        p -> p.getGender() == Person.Sex.MALE
            && p.getAge() >= 18
            && p.getAge() <= 25)
    .map(p -> p.getEmailAddress())
    .forEach(email -> System.out.println(email));
```

是的，通过就这么一行的代码，我们几乎可以完成之前10行甚至20行代码可以完成的事情。而且，在我们需要有什么改变的时候，重写代码也是非常的方便。如果使用Method inference，上面的代码甚至可以更简单，如下:

```
roster
    .stream()
    .filter(
        p -> p.getGender() == Person.Sex.MALE
            && p.getAge() >= 18
            && p.getAge() <= 25)
    .map(Person::getEmailAddress)
    .forEach(System.out::println);
```



总体上来看，Java 8完全是进入到了一个新的台阶，但是还是太慢了，只是在初步阶段。Scala已经都走出去这么远了，Java 8需要更多新的特性。

### 参考资料

[pipelines](http://docs.oracle.com/javase/tutorial/collections/streams/index.html#pipelines)

[streams](http://docs.oracle.com/javase/tutorial/collections/streams/index.html)

[method references](http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

[Lambda Expressions](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
