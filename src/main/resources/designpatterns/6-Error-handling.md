1- MonadError and ApplicativeError : 
- 
- Use ``MonadError`` / ``ApplicativeError`` (Cats Effect) and its methods ``attempt``, ``handleErrorWith``, ``rethrow``
... etc, since ``IO`` Monad implements ``MonadError[F, Throwable]``.

2- Either Monad :
-
Example :
````scala
trait Categories[F[_]] {
  def maybeFindAll: F[Either[RandomError, List[Category]]]
}

class Program[F[_]: Functor](categories: Categories[F]) {
  def findAll: F[List[Category]] = 
    category.maybeFindAll.map {
      case Right(c) => c
      case Left(RandomError) => List.empty[Category]
    } 
}
````
3- Classy prisms :
-
- Using ``classy prisms`` or ``classy optics`` using the ``Meow MTL`` library : It gives back typed errors
and exhausting pattern matching without polluting our interface ``F[Either[E, A]]`` no using monad transformers.
- It consists of defining a hierarchy of errors an ADT.
````scala
sealed trait UserError extends NoStackTrace
final case class UserAlreadyExists(username: String) extends UserError
final case class UserNotFound(username: String) extends UserError
final case class InvalidUserAge(age: Int) extends UserError
````

- Example 