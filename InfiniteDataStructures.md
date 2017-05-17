This is a post about finite vs infinite collections. At a presentation about Haskell, recently, I was surprised to learn that Haskell's list type is possibly-infinite. Of course, with a few moments' consideration, this can be seen as the obvious corollary of lazy evaluation: if the tail is lazy, the list may be infinite. The same is true of scala's `Stream` type and Java's also (`java.util.stream.Stream` introduced in Java 8).

I've been working a lot with Java's streams lately and I think many of that library's key design decisions are superb. Let's clarify them:

A stream is a sequence of transformative operations on input data, on which a *single* terminal operation can be made to reduce the stream to some calculation. The terminal operation may not need to traverse the whole stream (`findAny`, `allMatch` etc). The input data may be finite of known length, finite of unknown length or infinite. What are terminal operations?
 - `collect`
 - `allMatch`, `anyMatch`
 - `findAny`, `findfirst`
 - `count`
 - `reduce`
 - `forEach`

What are the non-terminal transformations? `map`, `flatMap`, `filter`, `peek` etc

**Aside**: stream exposes a `sorted` operation which is non-terminal. In my opinion this is a mistake. Any operation which can only be performed in general by reading the entire stream into memory should be terminal and should expose the stream as a sorted "bag" (i.e. a `SortedSet` where repetitions are allowed) or at the least a sorted `java.util.List` or `Array`. I digress.

**Another aside**: it's also my opinion that the stream library could have more effectively signaled the terminal and non-terminal operations differ fundamentally by insisting that the latter are all implemented via a collect. For example;

 - `collect(Collector)`
 - `collect(ShortCircuitingCollector)`

I'm still digressing, sorry.

But the type enthusiast in me thinks this is crazy. If I have a Stream of data which is infinite, should I even be *allowed* to perform a `count`, or even a `findAny` (what useful program might not terminate on such a line?). I'm led to think a little more about which terminal operations are legitimate on an infinite stream? The answer? Only one of them; `forEach`.

Conclusion; to write a library which reuses code but has concrete classes `InfiniteStream` and `FiniteStream` which share the transformative operations on some common base type but which expose only those terminal operations that make sense for themselves. But is it even possible to do this? How to reuse or modify `Spliterator` (atop which Java's streams are concstructed; basically a potentially parallelizable iterator) such that we can guarantee that a finite stream is indeed finite? There is a big difference between finite in the sense of "of known size" (creating a `Stream` from a `java.util.List`) and finite in the sense of "unknown": for example, when making a SQL query.

But now we come full circle: **if our tail is lazily computed, our data type is potentially infinite**. *There is no way to square this one*.
