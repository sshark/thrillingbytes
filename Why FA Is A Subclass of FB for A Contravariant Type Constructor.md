# Why Does `F[T] <: F[U]` Where F Is A Contravarint Type Constructor and `U <: T`?

If `U <: T` and a contravariant type constructor, `F[-R]`, then `F[T] <: F[U]` because a contravariant `F[-R]` is a consumer.

For example, given `F` is a `Box[-R]` with a single method `accept(r: R): Unit`, `U <: T` are `Apple <: Fruit` respectively, the answer is `Box[Fruit] <: Box[Apple]`. 

`Box[Fruit]` can used in places where `Box[Apple]` are because `Box[Fruit].accept(...)` accepts `Fruit`, `Apple`, and subclasses of `Fruit` e.g. `Orange` if `Orange <: Fruit` is defined. *A method accepting the class and its subclasses is in according to Java language specification, no configuration or special syntax is required.*
