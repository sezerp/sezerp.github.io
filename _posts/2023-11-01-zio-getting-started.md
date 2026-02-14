---
layout: single
classes: wide
title:  "ZIO basic concepts"
date:   2023-11-01 22:22:22 +0100
categories: computer-science
excerpt_separator: <!--more-->
excerpt: zio
tags: zio zio2 scala basic
author: Paweł Zabczyński
---

This article is my attempt to explain the first concept that a developer meets when starting the journey with functional programming, specifically with ZIO.

[ZIO](https://github.com/zio/zio) is a library and also a type within that library. Its main purpose is to provide a way to create maintainable software. The main feature is that everything runs asynchronously. However, in most cases it is not required to use advanced multithreading techniques. Additionally, it provides functional programming concepts. The core concept is the `effect`, which allows dealing with side effects as with pure code.


`Effect` is a description of a program. That means during creation of a `ZIO` expression, it will not be evaluated.


Examples

```scala
import zio.ZIO

val someVal: Unit = println("hello world") // will be printed immediately
val someEffect: ZIO[Any, Nothing, Unit] = ZIO.succeed(println("Hello World from effect")) // just constructs ZIO structure, will not perform println
```


```scala
val fut: Future[Int] = Future(10) // the future starts immediately after creating the instance
val effect: ZIO[Any, Nothing, Int] = ZIO.succeed(10) // creates only ZIO container
```

That introduces referential transparency, which means we can substitute any value with an expression that returns the same type. As long as a program does not contain any side effects, this is straightforward.

```scala

def add(a: Int, b: Int): Int = a + b // expression
val value1: Int = add(10, 3)
val value2: Int = 10 + 3
val value3: Int = 13
```
The above example shows that all of the values are equivalent — it doesn't matter which one is used. All of them have the same value and effect, because those expressions are pure and do not run any side effects.

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

In this example the expressions and values are not equivalent and cannot be substituted. All of them have the `Unit` type. However, when they are instantiated, the first sends data to system out, and the last sets a variable that is outside the expression scope. Although they return the same type/value, the effects of evaluation are different: print and set variable. The solution is `IO`. In ZIO there is the `ZIO` type, which describes computation instead of evaluating it immediately on creation. That means the construction is separated from evaluation.


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

Now all of the values contain only a computation description. During instantiation all of them will do the same thing: create a `ZIO` with the same types without any side effects. Even though two of them have side effects, they are now equivalent. But what is the difference? At first glance it does not give any advantages over normal expressions. The answer is the ability to construct smaller programs by describing them using the `ZIO` type and compose them somewhere else. If they contain any side effects, those are run on demand during evaluation, not construction, as many times as required.


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
On the console we will not see any messages because the program just creates programs, it does not evaluate them.

```scala
object Application {
  def main(args: Array[String]): Unit = {
    val program1 = println("Hello world!")
    val program2 = println("Hello")
  }
}
```
When we use pure Scala expressions, the result will be different — messages will appear on the console:

```shell
Hello world!
Hello
```


Let's try to compose

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

Now let's check what happens when we try to do the same with plain Scala:

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
The output is slightly different, the same as when we just create values

```shell
Hello world!
Hello
```

This happens because creation is not separated from evaluation, and it is not possible to re-evaluate it. When an expression that evaluates to a value contained a side effect, then that side effect was produced at the creation stage and never again — the val now contains only the resulting value. That means if instead of `println` there was an insert into a database, it would happen only once when creating the val.

#### Summary

The effect has three basic properties:

- describes the kind of computation using types, moreover tells that a side effect will possibly be performed on execution
- separates creation and evaluation process — in my opinion, the most important point
- produces a value of the described type on success

Effects were designed to describe effectful operations. To be honest, they provide more capabilities such as an async API, but that is a different topic.