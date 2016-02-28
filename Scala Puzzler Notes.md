#Scala Puzzlers 
An Artima published book by Andrew Phillips and Nermin 

I just completed [Scala Puzzlers](http://www.artima.com/shop/scala_puzzlers) and found it good for intermediate Scala programmers. I consider myself an intermediate Scala programmer. Beginners to Scala might not find some it useful just yet because they do not understand the language and the way how ?Scala works behind the scene. Once you have gained sufficient Scala experience, the reader will find lots of "aha" moments and fun too with this book. When I started with Scala, I encountered lots of peculiar errors and warnings during compilation or run-time. These messages or explanations were not easily understood. "Googling" helped to resolve some, some were not found and shelved until I come back for them for second time round. Hopefully, by then, someone would explain them in a blog or book (well, this is the book). 

I tend to forget as of what causes the errors if I seldom see them. So, I make quick references to some of the chapters for the explanations and solutions to these errors.

There is a [companion website](http://scalapuzzlers.com/) to this book.

##Arg Arrgh!! (Puzzler 6)
A good pointer for applying a function repeatedly,

	val result = (1 to n).foldLeft(arg) {
		(acc, _) => f(acc) // f(f(...(f(arg))))
	}

This chapter answered why there was a `compile error: could not find implicit value for parameter Ops:Numeric[N]` as `N` was clearly bound to `Double`, 

	def nextNumber[N](n: N)(implicit numericOps: Numeric[N]) =
		numericOps.plus(numericOps.times(n, n), numericOps.one)
	def applyNMulti(3)(2.0, nextNumber)
where `applyNMulti` was defined,

    def applyNMulti[T](n: Int)(arg: T, f: T => T) = ...
No, the compiler looks for each type requirement "individually" and not within the **same parameter list**. Unlike in curried function,`T` was evaluated to `Double` in the earlier part of the parameter list, `arg`, in `def applyNMulti[T](n: Int)(arg: T)(f: T => T) = ...`. That's why it worked in curried function.

However, the functions can be used as uncurried variant by explicitly putting the type,

 1. `applyNMulti(3)(2.0, nextNumber[Double])` -or-
 2. `applyNMulti[Double](3)(2.0, nextNumber)`
It might not be practical solution if the first parameter type differs. It has nothing to do with Typeclass Pattern.

##A Case of Equality (Puzzler 10)
This chapter explained the inner working of case classes *implicit overrides* with the methods `hashCode()` & `equals(...)` where their short hands were `##` & `==` respectively. It showed the replacement mechanisms for `new` and `case class`  factory method. For example, how to `new` case class without causing exception or replacing the standard factory method generated by the compiler in default case class declaration. It also showed to use `abstract case class` to replace the the default `case class` factory method.

As a result, structure equality was maintained i.e. 2 instances are equal if they have the same type and equal constructor arguments and types.

`hashCode()` and `equals(...)` are independent. Implement them separately.

##To Map or Not to Map (Puzzler 12)
Words of advice, convert the original to `Seq` if stable iteration is required.

##Return to Me! (Puzzler 14)
*For Scala versions 2.11.7 or later, this might not happen because the compiler will throw and error when the statement is defined.*

	val two = (x: Int) => { return x; 2 }
will throw an exception `NonLocalReturnControl` exception to imitate `return x` and therefore the whole expression `1 + one(2) + two(3)` will return 3. Also, this will happen to `val` but not `def`.

##Count Me Now, Count Me Later (Puzzler 15)
Highlighted the difference between `add(counter)(_) ` and `add(counter) _`. Answer, `counter` in the former would NOT be evaluated immediately but the later would be evaluated immediately.

From http://scalapuzzlers.com/#pzzlr-022, `f(a) _` and `f(a)(_)` have different meanings, governed by different sections of the Scala language specification.

1. `f(a)(_)` is the placeholder syntax for anonymous functions, described in SLS §6.23. Evaluation is deferred.
2. `f(a) _` is eta expansion, described in SLS §6.26.5. Arguments given are evaluated eagerly; only the method call itself is deferred.
	
It is worth noting a couple of points about this program. First, the use of vars does not reflect idiomatic Scala style. Even though Scala supports mutable state, it encourages a functional style of programming, i.e., preferring vals and immutability.

##One Bound, Two to Go (Puzzler 16)
*TODO Try out*

##What's in A Name? (Puzzler 19)
*TODO Try `val adder = new AdderWithBonus`*

##Irregular Expression (Puzzler 20)
1. Side effect from print caused the first statement succeed. This is because `MatchIterator` initialized the matcher via `toString()` call.
2. `hasNext()` is a more reliable approach unless it is known the iterator is non-empty.
3. use `findAllMatchIn(...)` instead of `findAllIn(...)`. Otherwise, use `findAllIn(...)` with `matchData` to avoid accidental access to `start`.

##Cast Away (Puzzler 22)
A wrapped value stored in a collection will be unboxed when compared against another value type. Of the declared type of the collection is a wrapper type, by contrast, no unboxing will occur and the value type will be boxed instead.

Because of performance reasons, compiler will always try to carry out operations on primitive types instead of wrapper type. Refer to the examples.

##A Case of Strings (Puzzler 27)

 - `Null` is just above `Nothing`
 - `null` is the one and only instance of `Null`
 - `null` cannot be tested with `isInstanceOf[Null]`
 
##Pick A Value, Pick AnyVal! (Puzzler 28)
The default value of type T as follow:

 - 0 if T is `Int`
 - 0 if T is `Long`
 - 0.0f if T is `Float`
 - 0.0d if T is `Double`
 - false if T is `Boolean`
 - () if T is `Unit`
 - `null` for the rest of T types for given

		trait NutritionalInfo {
			type T <: AnyVal
			var value: T = ...
		}
		
		val containsSugar = new NutritionalInfo { 
			type T = Boolean 
		}
		
		println(containsSugar.value)     // null
	    println(!containsSugar.value)   // false
because when attempt is made to negate the value, the compiler knows that it is a `Boolean` to be able to invoke the `unary_!` method. Automatic unboxing i.e. `scala.runtime.BoxesRuntimeUnboxToBoolean` was called.

The compiler refuses to unbox if the value can be treated as an `Any` i.e. `println(...)`, accepts `Any` as parameter type. Hence, there is no unboxing and underlying null value is retrieved.

##Implicit Kryptonite (Puzzler 29)
Overriding implicit of different name but of same type will cause ambiguity and compilation error.

Follow a pattern to avoid this and find similar patterns provided by the library. Case in point,

	class OperatingMode {
		implicit val ConsoleRender: Scanner.Console {...}
		implicit val DefaultAlarmHandler: Scanner.AlarmHandler {...}
	
	object NormalMode extends OperatingMode 
	object TestMode extends OperatingMode {
		override implicit val ConsoleRender: Scanner.Console {...}
		override implicit val TestAlarmHandler: Scanner.AlarmHandler {...}
	}

##Quite The Outspoken Type (Puzzler 30)
*Function vals* e.g. `val stringToInt: String => Int = _.toInt` take precedence over *defs* during implicit search. Therefore, it caused StackOverflowException in this puzzler because it kept referring to itself instead of `augmentString` method in object `Predef`. Had it has been defined using `def` type mismatch error would have thrown.

However, if regular *defs* and *function vals* were treated equally, an ambiguous reference to the overloaded definition would cause an exception. 

*Solution:* prefer *defs* over *function vals* when defining implicit to have ambiguity exception thrown during compilation to catch easy to miss bug.

##A View to Shill (Puzzler 31)
`mapValues` return a new iterator every time a value is retrieved as in the example. Therefore, *force* each with view before iterate through the sequence using `next()`.

##Set The Record Straight (Puzzler 32)
Methods define without parenthesis may accidentally invoke the `apply(...)` method. Therefore, it is suggested that include empty parenthesis only for method invocations with side-effects.

Beware of unintended type widening caused by methods on collections that allow the element type of a returned collection to be wider than original type.

##The Devil Is in The Detail (Puzzler 33)
Use `Map.withDefaultValue` for immutable defaults only other use `Map.withDefault`.  `withDefaultValue` returns the same instance across all map entries. `withDefault { _ => Buffer(100)}` to create new instances as in this example.

##A Listful of Dollars (Puzzler 35)
	type Dollar = Int
	final val Dollar: Dollar = 1
	val a: List[Dollar] = List(1, 2, 3)
	
	println(a map { a: Int => Dollar })
	println(a.map(a: Int => Dollar))  // (2)

This entry is to highlight the difference of using curly brackets and brackets within a function e.g. `map(...)`. So, a type declaration in an "Expr" that is not enclosed in parentheses, as in our case, is treated as a type ascription to the entire expression. In other words:

- `{ a: Int => Dollar }` is parsed as `{ (a: Int) => Dollar }`, with Int being the type of a and Dollar the constant value 1
- `(a: Int => Dollar)` is parsed as `(a: (Int => Dollar)}`, with a being a function of type Int => Dollar and Dollar a type alias for Int

Therefore,  the second `a` in (2) is a `List` which was declared earlier takes an index of type `Int` and returns an `Int` (alias `Dollar`). That is why an `IndexOutOfBoundsException` was thrown at (2) when it was asked for the 4th element with '3'.

##Additional Notes
###Hard to Reason
Why is side effect is "hard to reason"? When functions with side effect are passed around it is hard to know when and how many times it is called. Basically, it is hard to debug and understand. So it is more predictable if the functions have no side effect.

###Eta Expansion
Eta expansion is the operation of automatically coercing a method into an equivalent function, for example,

	val f = foo _ // or
	val f: A => B = foo
	
###Single Method Call
The compiler can turn consecutive innovations into a single method call if all arguments are provided, for example,

	curried(x: Int)(y: Int)(z: Int) == 
		singleCurried(x: Int, y: Int, z: Int)
So basically the JVM will invoke `curried(1)(2)(3)` as `curried(1, 2, 3)` in one go instead of 3 times.
