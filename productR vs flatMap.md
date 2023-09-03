# *>>* vs _*>_

*>>* or flat map is a lazy operator evaluator while _*>_ or productR is a strict evalutor. Taking `IO` from `cats-effect`, as an example, 

```
val left = IO(throw new Exception("bad"))
val right = {println("right is awaken"); IO.println("Hello, World")}

left *> right   // A
left >> right   // B
```

(A) shows "bad" and "right is awaken". `left` and `right` are evaluated at the same time
(B) shows "bad" only because right is lazily evaluated i.e. `left` is in an exception state therefore `right` is skipped

For ZIO, its `*>` method behaves lazily like Cats Effect `IO`'s `>>`

*`println(...)` is used to trace code exceution*