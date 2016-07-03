---
layout: post
title: "Java8新特性之stream"
date: 2016-07-03 16:44:02 +0800
comments: true
categories: java tools
---

#### Stream 

一直都很羡慕scala里面集和的操作，map, reduce, filter，现在我们可以在java中实现了。

##### 1. 对整数列表进行处理

比如，有一个整数的list，我们想先过滤掉可以整除2的数，在对每一个数进行平方。

```
public class IntegerSupplier implements Supplier<Integer> {
  private int value = 0;
  @Override
  public Integer get() {
    return value++;
  }
}

Stream<Integer> stream = Stream.generate(new IntegerSupplier());
stream.limit(100).filter( x-> x % 2 != 0 ).map( x -> x * x ).forEach( x -> System.out.print( x + " "));
```

通过Stream的generate函数，我们只要提供一个supplier的类，就可以很容易的得到一个stream的对象。其他collection，比如List也都提供了stream()函数。这样我们就可以很容易地，通过stream来对collection进行访问。



##### 2. 生成无限大小的list

利用这个我们还可以生成很多有意思的东西，比如生成一个无限大小的数组。

```
class NatureSupplier implements Supplier<Long> {
  long value = 0;
  @Override
  public Long get() {
    value += 1;
    return value;
  }
}
Stream<Long> naturalValues = Stream.generate(new NatureSupplier());
naturalValues.map( x -> x * x).limit(100).forEach( x -> System.out.print(x + " ") );
```

这里我们将stream的大小限制到了100个，否则的话，他会一直打印下去。



##### 3. 读取文件内容

再比如我们要读入一个文件的所有行到一个set里面，如果是之前的话，我们可能需要这样:

```
BufferedReader reader = new BufferedReader(new FileReader(new File(fileName)));

String line = reader.readLine();

while( line != null ) {

   lineSet.add( line );

   line = reader.readLine();

}
```

需要5行代码！这么简单的一件事情，在python里面就一行代码，为什么我需要5行代码呢？现在好一些了，可以这样：

```
BufferedReader reader = new BufferedReader(new FileReader(new File(fileName)));
reader.lines().forEach(lineSet::add );

```

##### 4. 计算平均数

之前我们定义了Person这个类([Lambda表达式](http://www.bigbear2017.com/blog/2016/07/02/java8xin-te-xing-zhi-lambdabiao-da-shi/))，如果我们想计算所有人的年龄的平均数，我们可以直接如下计算：

```
double average = roster
    .stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .mapToInt(Person::getAge)
    .average()
    .getAsDouble();
```

是的，只要一行，我们就可以得到所有人年龄的平均数。类似地，我们还可以计算sum，使用其他reduce function。

### 参考资料

[streams](http://docs.oracle.com/javase/tutorial/collections/streams/index.html)

