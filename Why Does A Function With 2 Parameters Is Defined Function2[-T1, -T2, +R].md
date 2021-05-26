A two-parameter Scala function is defined with contravariant parameter type and covariant return type,

   	Function2[-T, +R](t: T) : R

instead of, 

   	Function2[+T, -R](t: T) : R

Why? Let's assume a higher order function accepts a function with covariant type parameters. There is no guarantee that the given function can work with the instances provided by the higher order function. 

For example, `Fruit` is the parent of classes `Apple` and `Orange`. If the function is going to work with  `Fruit`, it must also work with `Apple` or `Orange` instances given by the higher order function. The function will crash when an `Orange` specific method is invoked on an `Apple` instance. 

Do not confuse this with the actual instances pass to the function by the higher function and the function instance. *The rule applies to the function used in the higher order  function.* 

Next, the contravariant return type does not go well with the caller because the function could return `Fruit` or `Plant` instances, assuming `Plant` is the parent of `Fruit`,  if the function return type is `Apple`. This is because there is no guarantee that the methods available in `Fruit` or `Apple` are available in `Plant`. This will blow up in the higher order function if the invoked method is unavailable.

Going back to the original definition. Remember, *we can pass subclasses to the function paraneters regardless whether it is defined as covaraint or contravariant type* i.e. `+T` or `-T`. 

And finally, the return type is covariant is because the instance has all the methods required. The higher function can only invoke methods defined in the return type of the function it accepts. Say, the higher function accepts a function with the return type of `Fruit`, the higher function can only invoke methods available in `Fruit`. Therefore, the function can return instances of `Fruit`, `Apple` or `Orange` without breaking the contract.
