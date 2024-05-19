
# Looking at A Monad Through An Example
There are many articles written about Monad. I want to do it differently explaining why does a Monad do and why it is useful to use Monad. Let's us begin right away.

*The code snippet in this article is using Scala syntax*

Using these 2 functions as my example,
``` scala
def div(a: Int, b: Int) = ???   		// A
def add(a: Int, b: Int): Int = a + b   	// B
```

Function (A) divide the first integer by the second integer. Function (B) adds 2 integer together. These 2 functions are pretty straight forward. Unlike function (B), a [total function](https://www.linkedin.com/advice/0/what-difference-between-partial-total-function-functional-2jine#:~:text=A%20total%20function%20is%20a,return%20another%20number%20as%20output), function (A) cannot compute when the second parameter is 0. If the parameters were `Float`, the function will return `Infinity`, but this is `Int`, it will throw an `java.lang.ArithmeticException` exception with the message `/ by zero`. Function (A) must find a way to let its caller know it cannot provide an answer if the second parameter is zero. There are a few ways to notify the caller of the exception and fortunately, there is one better answer than all of them discussed here.

## Returning An Error Code
For some cases returning an error code is the simplest solution. When the function fails, it will am error code to indicate what went wrong. Case in point, for function (A), if parameter `b` is zero, it will fail and return `-1` given the following implementation,
``` scala
def  div(a: Int, b: Int): Int = if (b == 0) -1 else a / b  // Bad implementation
```
However, this implementation is bad because `-1` could be a legit value for `div(-8, 8)`. To be honest, there is no integer we can use to indicate an error. Therefore, this method is not viable.

## A Error And Value Pair
Next, we split the error and result into a pair,
``` scala
def  div(a: Int, b: Int): (String, Int) = if (b == 0) ("/ by zero", 0) else (null, a / b)

val (error1, value1) = div(10, 2)   // (null, 5)

if (error1 == null) {   // C
    println(s"The result of 10 / 2 is $value1")
} else {
    println("Cannot be divied by zero")
}

val (error2, value2) = div(10, 0)   // ("/ by zero", 0)

// repeat the same code as (C) with error1 replace by error2 and so forth
```

The second part of the extraction logic conflicts with [DRY] (https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle. Yes, we can refactor (C) into,
``` scala
def extract[A](result: (String, Int), ifOk: A => Unit, ifNOK: A => Unit) =
    if (result._1 == null) {   // C
        println(s"The result of 10 / 2 is ${result._2}")
    } else {
        println("Cannot be divided by zero")
    }
```
Unfortunately, `extact(...)` is so restrictive. This will cause writing `extract(...)` in myraid of ways to suit some common needs. Also, what if I want to compose the result of the function `divide(...)` with `add(...)` ie `add(1, div(10, 0))`. The permutation will explode. This method is workable but it will quickly becomes a maintenace nightmare once the requirements get complicated.

## Throwing An Exception
In Scala, all exceptions are unchecked unlike Java, where exceptions are split into checked `Exception` and unchecked `RuntimeException`. In Java, functions that throw checked exceptions are enclosed in a `try-catch` block. On the other hand, Scala developers use the `try-catch` block to catch the exception they want to catch. 

In retrospective, it looks like throwing an exception seems to be the way forward. In fact, it is easy to implement function (A) using exception,
``` scala
def  div(a: Int, b: Int): Int = a / b
```
The responbility lies with the caller to catch the exception but the developer does not know if a function can throw an exception when certain parameteric values are met. Consequently, any function that the application uses can cause the application to be unusable or unstable at best if the exception is not caught in its place. This makes the job of the developers very unpleasant. Worse, we are back to writing our code the Java style with `try-catch` or `try-catch-finally` blocks everywhere blindfolded[^canThrow].

[^canThrow]: Scala 3 is experimenting with the selective checked exception capture using the [`CanThrow`](https://docs.scala-lang.org/scala3/reference/experimental/canthrow.html) capability.

## The Better Answer, Use An Effect
Effect in this context is a *container* or a container with some capabilities. Effect is not *side effect*. Simple container like `Opton` is a list with the maximum capacity of 1 element or empty. With `Option` effect the function can let its caller know its return status. The function will return `Some` or None to indicate success or failure. Function (A) implementation and usage would look like this
``` scala
def  div(a: Int, b: Int): Option[Int] = if (b == 0) None else Some(a / b)

val result1: Option[Int] = div(10, 2)	// Some(5)
val result2: Option[Int] = duv(10, 0)	// None

// print the result
result1 match {
  case Some(x) => println(s"The result of 10 / 2 is $x")
  case None    => println("Cannot be divided by zero")
}

result2 match {
  case Some(x) => println(s"The result of 10 / 0 is $x")
  case None    => println("Cannot be divided by zero")
}
```

Like before we can apply DRY principle here,
```  scala
def extract(result: Option[Int]): Unit = result match {
  case Some(x) => println(s"The result is $x")
  case None    => println("Cannot be divided by 0")
}

extract(result1)	// The result is 5
extract(result2)	// Cannot be divided by 0
```
One thing to take note is don't be eager to extract the values from the effect unless this is the final call. Let's say, we want to add 10 to the result get from `divide`. We cannot write our code as such `add(10, divide(10, 2)` because `divide(...)` returns an `Option` instead of a `Int`. We have to write our code this way,
``` scala
val result3: Option[Int] = for {
  x <- divide(10, 2)
} yield Some(add(10, x))

extract(result3)	// The result is 15

val result4: Option[Int] = for {
  x <- divide(10, 0)
} yield Some(add(10, x))

extract(result4)	// Cannot be divided by zero
```
It is not necessarily for the developer to check if  the call to `divide(...)` is a success or failure before moving on to the next function. The result will be fed to next function and contunue to do as long as there are functions to call, the final result from the [*for-comprehesion*](https://docs.scala-lang.org/tour/for-comprehensions.html) loop will return `Some` of a value or `None`.

Had function (B) return the result of type `Option[Int]` instead of `Int`, we can rewrite the for-comprehesion loop as,
``` scala
def add(a: Int, b: Int): Option[Int] = Some(a + b)

val result5: Option[Int] = for {
  x <- divide(10, 2)
} yield add(10, x)
```
What if we want the function to provide the error message instead. We can use `Either[String, Int]`.
``` scala
def divide(a: Int, b: Int): Either[String, Int] = 
  if (b == 0) Left("/ by zero") else Right(a / b)

def extract(result: Either[String, Int]): Unit = result match {
  case Right(x)  => println(s"The result is $x")
  case Left(err) => println(s"Error: $err")
}

val result6: Either[String, Int] = for {
  x <- divide(10, 2)
} yield add(10, x)

extract(result6)	// The result is 5

val result7: Either[String, Int] = for {
  x <- divide(10, 0)
} yield add(10, x)  

extract(result7)	// Error: / by zero
```

We can make use of exception and `Either` to simply `divide(...)`.
``` scala
def divide(a: Int, b: Int): Either[String, Int] = 
  try {
    Right(a / b)
  } catch {case e: ArithmeticException => Left(e.getMessage)
```

> **Sidebar**
> As we discover later on, we can use other data types like [`Try`](https://www.scala-lang.org/api/3.3.3/scala/util/Try.html) from the standard Scala library or `IO` from a 3rd party library [Cats Effect](https://typelevel.org/cats-effect/). 
>As metioned before, an effect is a container with capabilities: -
>1. `Option` provides a value or no value (empty) capability.
>2. `List` provides a list of values or no value (empty) capability.
>3. `Try`, like the `try-catch` block, catches any exception thrown within it.
>4. `IO` is an IO Monad which has many capabilities which include handling side-effects, error handling, parallel computation, and many more.

This is the best solution compares to the other solutions presented here albeit it is more complicated and require more reading into the topic. It makes the code better managed as more code is added to handle new requirements. However, this is not the point of this article.

## And The Point Is...
I do hope you agree using effect is the best approach to this issue. But, what does this approach has to do with Monad? Monad is a typeclass[^tc]) that has the`map` and `flatMap` methods. I want to discuss these methods and how they work in the for-comprehension loop.

> **Sidebar**
> Functor is a typeclass too which contains the method `map` while Monad has the method `flatMap`. Since Monad is a subclass of Functor, Monad has `map` and `flatMap` methods.

[^tc]: It is imperative to understand what typeclass is and how does it functions. Please refer to  https://dev.to/jmcclell/inheritance-vs-generics-vs-typeclasses-in-scala-20op for an introduction.
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjA3NDI2MSw3NTgxMzI3MTUsMTczNzEyMj
A5MSwxMzMwNTI0NDM0LC05NTAyMzU4ODYsMTM4MTQ4MjI3MSwt
MjExODQ0NDgxNl19
-->