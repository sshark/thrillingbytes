
# Looking at A Monad Through An Example
There are many articles written about Monad. I want to do it differently explaining why does a Monad do and why it is useful to use Monad. Let's us begin right away.

Using these 2 functions as my example,
``` scala
def div(a: Int, b: Int) = ???   // A
def add(a: Int, b: Int) = ???   // B
```

Function (A) divide the first integer by the second integer. Function (B) adds 2 integer together. These 2 functions are pretty straight forward. Unlike function (B), a [total function](https://www.linkedin.com/advice/0/what-difference-between-partial-total-function-functional-2jine#:~:text=A%20total%20function%20is%20a,return%20another%20number%20as%20output), function (A) cannot compute when the second parameter is 0. If the parameters were `Float`, the function will return `Infinity`, but this is `Int`, it will throw an `java.lang.ArithmeticException` exception with the message `/ by zero`. Function (A) must find a way to let its caller know it cannot provide an answer if the second parameter is zero. There are a few ways to notify the caller of the exception and fortunately, there is one better answer than all of them discussed here.

## Returning An Error Code
For some cases returning an error code is the simplest solution. When the function fails, it will am error code to indicate what went wrong. Case in point, for function (A), if parameter `b` is zero, it will fail and return `-1` given the following implementation,
``` scala
def  div(a: Int, b: Int): Int = if (b == 0) -1 else a / b  // Bad implementation
```
However, this implementation is bad because `-1` could be a legit value for `div(-8, 8)`. To be honest, there is no integer we can use to indicate an error. This method is not viable.

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

The second part of the extraction logic conflicts with DRY principle. Yes, we can refactor (C) into,
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
In Scala, all exceptions are unchecked unlike Java, where exceptions are split into checked `Exception` and unchecled `RuntimeException`. In Java, for checked exceptions, the developer has to enclose the function a `try-catch` block. On the other hand, Scala developers use the `try-catch` block if they  want to catch the exception. 

In retrospective, it looks like throwing an exception seems to be the way forward. In fact, it is easy to implement function (A) using exception,
``` scala
def  div(a: Int, b: Int): Int = a / b
```
The responbility lies with the caller to catch the exception if anything goes bad. The downside of this is, the developer does not know that function (A) can and will throw an exception when `b` is zero. This will cause the application that uses function (A) to be unusable or unstable at best if the exception is not caught in its place. 

## The Better Answer, Use An Effect

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNDYzMTM5MV19
-->