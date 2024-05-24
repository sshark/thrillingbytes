
# Looking at A Monad Through An Example
Though many articles were written about Monad, this article shows how a Monad is used in practice with an example and then explains why a Monad is needed. Let's begin right away.

*The code snippets in this article are using Scala 3 syntax*

This article uses these 2 functions as an example,
``` scala
def div(a: Int, b: Int) = ???           // A
def add(a: Int, b: Int): Int = a + b   	// B
```

Function (A) divides the first integer by the second integer. Function (B) adds 2 integers together. These 2 functions are pretty straightforward. Unlike function (B), a [total function](https://www.linkedin.com/advice/0/what-difference-between-partial-total-function-functional-2jine#:~:text=A%20total%20function%20is%20a,return%20another%20number%20as%20output), function (A) cannot compute when the second parameter is 0. If the parameter types are `Float`, the function will return `Infinity`, but this is `Int`, throwing a `java.lang.ArithmeticException` exception with the message `/ by zero`. Function (A) must find a way to let its caller know it cannot compute if the second parameter is zero. There are a few ways to handle the error and fortunately, there is a clear way to do this.

Function (A) actual implementation is determined by how the value and error are handled in each scenario.

## Returning An Error Code
For some cases returning an error code is the simplest solution. When the function fails, it returns an error code to indicate failure. Case in point, for function (A), if parameter `b` is zero, it fails and returns `-1` given the following implementation,
``` scala
def  div(a: Int, b: Int): Int = if (b == 0) -1 else a / b  // Bad implementation
```
This is a bad implementation because `-1` is a legit value for `div(-8, 8)`. There is no integer the function can return to indicate an error. Therefore, this method is not viable.

## A Error And Result Pair
Next, we split the error and result into a pair,
``` scala
def  div(a: Int, b: Int): (String, Int) = if (b == 0) ("/ by zero", 0) else (null, a / b)

val (error1, value1) = div(10, 2)   // (null, 5)

if (error1 == null) {
    println(s"The result of 10 / 2 is $value1")
} else {
    println("Cannot be divided by zero")
}

val (error2, value2) = div(10, 0)   // ("/ by zero", 0)

// repeat the if-else block with error1 replace by error2
```

The second part of the extraction logic conflicts with [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle. Yes, we can refactor the `if-else` block  into a function,
``` scala
def extract[A](result: (String, Int), ifOk: A => Unit, ifNOK: A => Unit) =
    if (result._1 == null) {
        println(s"The result of 10 / 2 is ${result._2}")
    } else {
        println("Cannot be divided by zero")
    }
```
Unfortunately, `extract (...)` is so restrictive. This will cause writing `extract(...)` to suit simple needs in many ways. Also, what if I want to compose the result of the function `div(...)` with `add(...)` i.e. `add(1, div(10, 0))`. The permutation will explode. This method is workable but soon it becomes a maintenance nightmare once the requirements get complicated.

## Throwing An Exception
In Scala, all exceptions are unchecked unlike Java, where exceptions are split into checked `Exception` and unchecked `RuntimeException`. In Java, functions that throw checked exceptions are enclosed in a `try-catch` block. On the other hand, Scala developers use the `try-catch` block to catch the exception they want to catch. 

In retrospect, throwing an exception seems to be the way forward. It is easy to implement the function (A) using the exception,
``` scala
def  div(a: Int, b: Int): Int = a / b
```
The responsibility lies with the caller to catch the exception but the developer does not know if a function can throw an exception when certain parameteric values are met. Consequently, any function that the application uses can cause the application to be unusable or unstable at best if the exception is not caught in its place. This makes the job of the developers very unpleasant. Worse, we are back to writing our code the Java-style with `try-catch` or `try-catch-finally` blocks everywhere blindfolded[^canThrow].

[^canThrow]: Scala 3 is experimenting with the selective checked exception capture using the [`CanThrow`](https://docs.scala-lang.org/scala3/reference/experimental/canthrow.html) capability.

## The Better Answer, Use An Effect
The effect in this context is a *container* or a container with some capabilities. The effect is not a *side effect*. A simple container like `Option` is a list with the maximum capacity of 1 element or empty. With the `Option` effect the function can let its caller know its return status. The function will return `Some` or None to indicate success or failure. Function (A) implementation and usage would look like this
``` scala
def  div(a: Int, b: Int): Option[Int] = if (b == 0) None else Some(a / b)

val result1: Option[Int] = div(10, 2)	// Some(5)
val result2: Option[Int] = div(10, 0)	// None

// print the result
def extract(result: Option[Int]): Unit = result match {
  case Some(x) => println(s"The result is $x")
  case None    => println("Cannot be divided by 0")
}

extract(result1)	// The result is 5
extract(result2)	// Cannot be divided by 0
```
Please do not eagerly extract values from the effect unless this is the final call. 

Let's say, we have a function to add 10 to the result if the result is even otherwise cancel the whole calculation. `None` represents the cancelation in the composed function. 

It is not recommended to write `addIfEven(10, div(10, 2).get)` because when `div(...)` returns a `None` instead of a `Some[Int]`, invoking a `get` on `None` would cause an exception. 

Using `Option` inside the function i.e. `addIfEven(a: Int, b: Option[Int]): Option[Int]` is bad in this case because it makes the code difficult to work with the values inside `Option`.

Instead, define as `addIfEven(a: Int, b: Int): Option[Int]` and write the code the following way,
``` scala
val result3: Option[Int] = for {
  x <- div(10, 2)
  y <- addIfEven(10, x)
} yield y

extract(result3)	// The result is 15

val result4: Option[Int] = for {
  x <- div(10, 0)
  y <- addIfEven(10, x)
} yield y

extract(result4)	// Cannot be divided by zero
```
It is not necessarily for the developer to check if the call to `div(...)` is a success or failure before moving on to the next function. The result will be fed to the next function and continue to do so as long as there are functions to call, the final result from the [*for-comprehension*](https://docs.scala-lang.org/tour/for-comprehensions.html) loop will return `Some` of a value or `None`.

Had the function (B) returned the result of type `Option[Int]` instead of `Int`, we can rewrite the for-comprehension loop as,
``` scala
def add(a: Int, b: Int): Option[Int] = Some(a + b)

val result5: Option[Int] = for {
  x <- div(10, 2)
  y <- add(10, x)
} yield y
```
What if we want the function to provide the error message instead? We can use `Either[String, Int]`.
``` scala
def div(a: Int, b: Int): Either[String, Int] = 
  if (b == 0) Left("/ by zero") else Right(a / b)

def extract(result: Either[String, Int]): Unit = result match {
  case Right(x)  => println(s"The result is $x")
  case Left(err) => println(s"Error: $err")
}

val result6: Either[String, Int] = for {
  x <- div(10, 2)
} yield add(10, x)

extract(result6)	// The result is 5

val result7: Either[String, Int] = for {
  x <- div(10, 0)
} yield add(10, x)  

extract(result7)	// Error: / by zero
```

Use exception to simply `div(...)`,
``` scala
def div(a: Int, b: Int): Either[String, Int] = 
  try {
    Right(a / b)
  } catch {case e: ArithmeticException => Left(e.getMessage)}
```

> **Sidebar**\
> As we discover later on, we can use other data types like [`Try`](https://www.scala-lang.org/api/3.3.3/scala/util/Try.html) from the standard Scala library or `IO` from a 3rd party library [Cats Effect](https://typelevel.org/cats-effect/). 
>As mentioned before, an effect is a container with capabilities: -
>1. `Option` provides a value or no value (empty) capability.
>2. `List` provides a list of values or no value (empty) capability.
>3. `Try`, like the `try-catch` block, catches any exception thrown within it.
>4. `IO` is an IO Monad which has many capabilities which include handling side-effects, error handling, parallel computation, and many more.

## And The Point Is...
Using effect is a good approach to resolve this issue. But, what does this have to do with Monads? This is one of many ways using Monads to simplfy branching between actual and unexpected (bad) events without deeply nested `if-else-then` branches in the flow.  The same monadic approach can be used to solve other issues in a similar fashion like how bad parameter or input is handled. However, this topic requires more reading and practice before it can be truly useful. We have to start somewhere. The payoff is making the code highly manageable as more code is added to tackle new requirements. Thank you for reading.

### For-Comprehension And Typeclass (Optional)
A Monad is a typeclass[^tc] that has a few functions. In the interest of this article, the focus is on the `map` and `flatMap` functions.  `map` is inherited from the Functor.  Strictly speaking, a Monad is a subclass of *Applicative* which in turn a subclass of *Functor*.

[^tc]: Typeclass is like Java `interface`. However, It is imperative to understand how typeclass functions. Please refer to  https://dev.to/jmcclell/inheritance-vs-generics-vs-typeclasses-in-scala-20op for an introduction.
 
In Scala, the for-comprehension loop is a synatic sugar for a series of `flatMap` and `map`e.g.,
 ``` scala
val result8: Option[Int] = for {
  x <- div(10, 2)
  y <- Option(x - 10)
} yield add(10, y)

// loosely converted to

val result8: Option[Int] = 
  div(10, 2)
    .flatMap(x => Option(x - 10)
    .map(y => add(10, y)))
```

Classes like `Option`, `List`, and `Either` can work right out of the box with for-comprehension because these classes have `map` and `flatMap` methods defined. If a random class `MyBox` without these 2 methods defined, it would not work. The developer could add these methods to `MyBox` if the developer owns the source. If he does not, then he has to use adhoc polymorphism a.k.a typeclassing which is very useful for extending the class capabilities. Please refer to [here](https://gist.github.com/sshark/6a169bedfa97718dd72eb0738fbb046f) for the `MyBox` Monad typeclass implemention and example.

Classes must conforms to the [Monad Law](https://devth.com/monad-laws-in-scala) to be a Monad. For example, `Option`, `List`, and `Either` are monads because they passed the Monad Law test. Classes like `Set` and `Try` are not because they failed the test even though they have `map` and `flatMap` methods defined.


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMxNTczMzE5Myw4ODY2NDg1MDUsMTc2Nz
AwNTkzNywxODA0Njg2ODA2LDEwNjUzMzc0MTMsMTE4OTM1Njc1
NywtMTkyMTY5NzY4MiwtMzYzODEyMDIyLDE4ODgwMDUzNjYsMT
g3MTU3NTE2NywyNDI3NTU0ODIsLTg1MTAzNjU2MywyNTMzNzgy
NzksLTI1NDg0NzkxNiwtMTE3MjY4NDY5OSwtMTA3Mzk3MTg4MS
wyMTQ0Nzc3Mzc0LC03MDU1NjY5MzEsLTIwNDAyNzU2NzUsMzU2
NzU3NTc2XX0=
-->