---
layout: single
classes: wide
title:  "ZIO basic concepts"
date:   2023-11-01 22:22:22 +0100
categories: computer-science
excerpt_separator: <!--more-->
excerpt: zio
tags: zio zio2 scala
---

The article is my try to explain first concept that developer meet when start the journey with functional programming, not only ZIO and usually .

The [ZIO](https://github.com/zio/zio) is a library and also a type in library. The main purpose is provide way to create maintainable software. 
The mian future is that everything run in asynhronous simultaneously. However, in most cases not required use advanced multithreading techniques. Additionally, provide functional programming concepts. The core concept is `effect` that allow dill with side effects as with pure code.


`Effect` is an description of program. Thats mean during creation od `ZIO` expression will not be evaluated.


Examples

```scala
import zio.ZIO

val someVal: Unit = println("hello world") // will print be immeditialy
val someEffect: ZIO[Any, Nothing, Unit] = ZIO.succeed(println("Hello World from effect")) // just construct ZIO structure, will not perform println
```


```scala
val fut: Future[Int] = Future(10) // the future start immeditially after create instance
val effect: ZIO[Any, Nothing, Int] = ZIO.successd(10) // create only ZIO container
```

That introduce referentaial transparency what mean we can substitute any value with expression that return the same type. As long as program does not contains any side effect is simple.

```scala

def add(a: Int, b: Int): Int = a + b // expression
val value1: Int = add(10, 3)
val value2: Int = 10 + 3
val value3: Int = 13
```
Above example shows that all of the values are equivalent, it doesn't matter which one will be used. All of them has the same value and effect, but those expressions are pure, does not runn any side effects.

[Side effect definition](https://en.wikipedia.org/wiki/Side_effect_(computer_science))
> In computer science, an operation, function or expression is said to have a side effect if it modifies some state variable value(s) outside its local environment, which is to say if it has any observable effect other than its primary effect of returning a value to the invoker of the operatio

```scala
def printAddResult(a: Int, b: Int): Unit = {
    val result = a + b

    println(result)
}

val val1: Unit = printAddResult(10, 3)
val value2: Unit = ()
var someVariable: Int = 0; 
val value3: Unit = someVariable += 1
```

In this example the expressions and values are not equivalents and cannot be substitute. All of them has `Unit` type. However, when they are instantioniated firs send data into system out and last set varialbe that is out of expression scope. Despite they return the same type/value as effect of evaluation are diffrent result, print and set variable. The solution are `IO`s. In zio is `ZIO` type which describe computation instead of evauating then immeditially on creation. Thats mean the construction is separated from evaluation.


```scala

def printAddResult(a: Int, b: Int): ZIO[Any, Nothing, Unit] = {
    ZIO.succeed {
        val result = a + b
        println(result)
    }
}
val zio: ZIO[Any, Nothing, Unit] = printAddResult(10, 3)
val zio2: ZIO[Any, Nothing, Unit] = ZIO.unit
var someVariable = 0
val zio3: ZIO[Any, Nothing, Unit] = ZIO.succeed(someVariable += 1)
```

Now all of values contains only computation description. During instantionization all of them will do the same, create `ZIO` with same types without any side effects. Despite two of them have side effects they now are equivalent. But what is a difference? On first look it does not give any advanatages over normal expressions. The answer is ability to construct smaller programs by describing them using `ZIO` type and compose them somwhere else and if contains any side effect run them on demand during evcaluation not construction, as many time as required.


```scala
import zio.{Scope, ZIO, ZIOAppArgs, ZIOAppDefault}

object Application extends ZIOAppDefault {
  override def run: ZIO[Any with ZIOAppArgs with Scope, Any, Any] = {
    val program1 = ZIO.succeed(println("Hello World"))
    val program2 = ZIO.succeed(println("Hello"))

    ZIO.unit
  }
}
```
On console we will not see any messages because the program just create a programs, not evalute them.

```scala
object Application {
  def main(args: Array[String]): Unit = {
    val program1 = println("Hello world!")
    val program2 = println("Hello")
  }
}
```
When we use pure scala expressions then result will be diffrent, on console will appera messages:

```shell
Hello world!
Hello
```


Lets try compose

```scala
import zio.{Scope, ZIO, ZIOAppArgs, ZIOAppDefault}
object Application extends ZIOAppDefault {
  override def run: ZIO[Any with ZIOAppArgs with Scope, Any, Any] = {
    val program1 = ZIO.succeed(println("Hello World"))
    val program2 = ZIO.succeed(println("Hello"))

    for {
      _ <- program1
      _ <- program2
      _ <- program1
      _ <- program2
    } yield ()
  }
}

```

The result now will be messages on console:

```shell
Hello World
Hello
Hello World
Hello
```

now lets check what happend when we try do the same with plain scala:

```scala
object Application {
  def main(args: Array[String]): Unit = {
    val program1 = println("Hello world!")
    val program2 = println("Hello")

    program1
    program2
    program1
    program2
  }
}
```
The output is sligthlyu different, same as when we just create values

```shell
Hello world!
Hello
```

This happend due to creation is not separated from evaluation and is not possible revaluate it, when expression that evaluate to value contaned side effect, then  side effect was produced on creation stage and never agian, val now contains only resulting value. Thts mean if instead of `println` will be insert int data base then will happen only once when create val. 

#### Summary

The effect has basic three property

- describe kind of computation using type, moreover tells that posibly side effect will be performedwhen on egzecution
- separate creation and evaluation process, imo the most important point
- produce value of described type on success

Effects was designed to describe effectful operations, to be honest provides more capabilities such as async api but is different topic.