A two-parameter Scala function is defined with contravariant parameter type and covariant return type,

   	Function2[-T1, -T2, +R](t1: T1, t2: T2) : R

instead of, 

   	Function2[+T1, +T2, -R](t1: T1, t2: T2) : R

Why? Let's assume a higher order function accepts a function with covariant type parameters. There is no guarantee that the given function can find the methods in the instances provided by the higher order function. 

For example, `Fruit` is the parent of classes `Apple` and `Orange`. If the function is going to work with  `Fruit`,   it can work with `Apple` or `Orange` instances from the higher order function. The function will crash when the function invoked a method specific to `Orange` with an instance of `Apple`. 

Do not confuse this with the actual instances pass to the function by the higher function and the function instance. *The rule applies to the function instance pass to the higher function.* 

Next, the contravariant return type does not go well with the caller because the function could return `Fruit` or `Plant` instances, assuming `Plant` is the parent of `Fruit`,  if the function return type is `Apple`. This is because there is no guarantee that the method available in `Fruit` or `Apple` is also available in `Plant`. and it will cause the higher function to blow up if the method is not there.

Going back to the original definition. Remember, *we can pass subclasses but not the parents of the intended parameter type regardless of how the parameter types are defined* i.e. `+T` or `-T`. When the higher order function invokes the given function, it knows for sure that the instances it passes can handle what the function demands because the function is expecting the parent or the exact type declared in the parameter.

And finally, the return type is covariant is because the instance returned can manage what is asked by the higher function. The higher function can only invoke methods defined in the return type of the function it accepts. Say, the higher function accepts a function with the return type of `Fruit`, the higher function can only invoke methods available in `Fruit`. Therefore, the function can return instances of `Fruit`, `Apple` or `Orange` without breaking the contract.

