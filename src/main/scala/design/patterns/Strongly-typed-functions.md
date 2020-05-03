
- A ``case class`` provides a ``.copy`` method.
- To get rid of the ``.copy`` method, you can use a ``sealed abstract class``.
- ``case class`` can causes performance issues and a rise in memory allocation.
- It is preferable to use ```Newtype```, which is a zero-cost wrappers with no runtime overhead.
````aidl
import io.estatico.newtype.macros._

@newtype case class Username(value: String)
````
- To use ```Newtype```, we need an extra plugin, and a plugin flag : ``-Ymacro-annotations``.
- ```Newtype``` will be replaced, in Scala 3, by ``opaque types``.
- ```Newtype``` can be constructed as well using this way :
````aidl
import io.estatico.newtype.ops._

"something".coerce[Username]
````
- ```Refinement Type``` guarantees validating data at compile-time and runtime.
````aidl
import eu.timepit.refined._
import eu.timepit.refined.auto._
import eu.timepit.refined.types.string.NonEmptyString._

def create(value: NonEmptyString): F[Option[User]]
````
- We can create our custom ```Refinement Type```:
````aidl
import eu.timepit.refined.api.Refined
import eu.timepit.refined.collection.Contains

type Username = String Refined Contains['n']
// The username must contain the character 'n'
````
- Mixing ```Newtype``` with ```Refinement Type``` is so powerful, because it guarantees validating data at compile time with a no runtime overhead.