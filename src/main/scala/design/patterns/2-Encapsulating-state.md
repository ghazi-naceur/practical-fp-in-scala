- To encapsulate state, we can use : 
``MonadeState``, ``StateT``, ``MVar`` and ``Ref``.

- One of the best approaches to manage state is to encapsulate the state in the interpreter 
and expose an abstract interface gathering the needed functionalities.

- The interface should know nothing about the state.

- When using ``MonadeState[F, AppState]`` or ``Ref[F, AppState]``, functions, potentially, can access and modify the
state of the application, unless it is used with ``classy lenses``.

- Since state is mutable, it must be handled to guarantee that it will not leak, so the interpreter's (`Ref` of example)
 constructor will private, and we use a smart constructor instead :
 ````scala
import cats.effect.Sync
import cats.implicits._

object LiveCounter {
    def make[F[_]: Sync]: F[Counter[F]] = 
        Ref.of[F, Int](0).map(new LiveCounter(_))
}
````

- The created interface is 'effectful' (because it is allocating a mutable reference : the counter), it is a good 
practise to wrap ``F``.

- We can avoid creating a class for the state, and put it directly in the smart constructor of the object ``LiveCounter``:
```
object LiveCounter {
    def make[F[_]: Sync]: F[Counter[F]] = 
        Ref.of[F, Int](0).map { ref =>
            new Counter[F] {
                def incr: F[Unit] = ref.update(_ + 1)
                def get: F[Int] = ref.get
            }
        }
}
```
 => The drawback of this method : It doesn't allow us to define local variables (unless private) inside the implementation,
    also doesn't allow to override **def**s and **val**s ::> Scale compiler won't allow it.
 As a solution for this, you need to define your interpreter inside a class.