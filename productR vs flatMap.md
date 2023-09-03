# `>>` vs `*>`

`>>` is a *lazy* evaluator while `*>` or `productR` is a *strict* evaluator. Taking `IO` from `cats-effect`, as an example, 

```scala
lazy val left = IO(throw new Exception("bad"))
lazy val right = {println("right is awakened"); IO.println("Hello, World")}

left *> right  // A
left >> right  // B
```

(A) prints "bad" and "right is awakened". Both `left` and `right` are evaluated\
(B) prints "bad" only. `right` is a `lazy` value and it is not evaluated immediately. `right` skips evaluation because `left` has thrown an exception

For ZIO, its `*>` method behaves lazily like Cats Effect `IO`'s `>>`

## Observations
1. Both `println(...)` and `lazy` are not required in actual use
2. `println(...)` for tracing code execution
3. `lazy` to prevent `right is awaken` message from printing if not invoked
