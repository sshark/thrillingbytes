# Looking at A Monad Through An Example
There are many articles written about Monad. I want to do it differently explaining why does a Monad do and why it is useful to use Monad. Let's us begin right away.

Using these 2 functions as my example,
``` scala
def div(a: Int, b: Int) = ???   // A
def add(a: Int, b: Int) = ???   // B
```

Function (A) divide the first integer by the second integer. Function (B) adds 2 integer together. These 2 functions are pretty straight forward. Unlike function (B), a [total function](https://www.linkedin.com/advice/0/what-difference-between-partial-total-function-functional-2jine#:~:text=A%20total%20function%20is%20a,return%20another%20number%20as%20output), function (A) cannot compute when the second parameter is 0. If the parameters were `Float`, the function will return `Infinity`, but this is `Int`, it will throw an `java.lang.ArithmeticException` exception with the message `/ by zero`. Function (A) must find a way to let its caller know it cannot provide an answer if the second parameter is zero. There are a few ways to notify the caller of the exception and fortunately, there is one better answer than all of them discussed here.

## Return An Error Code
## Return A Error And Value Pair
## Throw An Exception
## The Better Answer, Use An Effect
