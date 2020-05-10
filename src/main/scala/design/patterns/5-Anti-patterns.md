1- Seq : A base trait for sequences 
-
Be Specific ! Instead of using ``Seq``, use ``List``, ``Vector``, ``LazyList`` (since Scala 2.13.0), ``Chain``, ``fs.Stream`` ... etc

2- About monad transformers :
- 
- Use monad transformations in local functions, more likely in the interpreters, but do not use them in interfaces.
- Example : Using ``OptionT`` for the API is undesirable :
* So instead of expose ``OptionT``:
````scala
trait Users[F[_]] {
  def findUser(id: UUID): OptionT[F, User]
}
````
* You wrap it with ``F`` :
````scala
trait Users[F[_]] {
  def findUser(id: UUID): F[Option[User]]
}
````
* Whn we have an abstract effect ``F`` for our API, we should never expose it (the API) in our interface.