1- Regions of sharing :
- 
- To  understand the shared state, you need to understand ``regions of sharing``, which can be denoted by a simple
``flatMap`` call.
- ``Semaphore`` is another concurrent structure provided by Cats Effect, which work in a situation when we have 2 programs
that need a permit to perform an expensive task.

````scala
import cats.effect._
import cats.effect.concurrent.Semaphore
import cats.effect.implicits._
import cats.implicits._

object SharedState extends IOApp {
    
    def someExpensiveTask: IO[Unit] = 
        IO.sleep(1.second) >>
            putStrLn("expensive task") >>
            someExpensiveTask

    def p1(sem: Semaphore[IO]): IO[Unit] = 
        sem.withPermit(someExpensiveTask) >> p1(sem)

    def p2(sem: Semaphore[IO]): IO[Unit] = 
            sem.withPermit(someExpensiveTask) >> p2(sem)

    def run(args: List[String]): IO[ExitCode] = 
        Semaphore[IO](1).flatMap { sem =>
            p1(sem).start.void *>
                p2(sem).start.void
        } *> IO.never.as(ExitCode.Success)
}
````
- In this example, we have 2 programs handling the execution of the expensive task by using the same Semaphore.
- The Semaphore is created in the ``run`` function, calling its apply method with an argument '1', that indicates 
the number of permits, and then calling ``flatMap`` to share between ``p1`` and ``p2``.
- The enclosing ``flatMap`` bloc is what denotes our ``region of sharing``.
- ``Region of sharing`` presents the location where we control and share our data structure.
- This example explains why almost all concurrent structures (``Ref``, ``Semaphore``, ``Deferred``, HTTP Client (http4s)
 ... etc) are wrapped in ``F``
 
 2- State leak :
 -
 - To illustrate the state leak, we can imagine the last example, with Semaphore, without the wrapper ``IO``:
````scala
object LeakeySharedState extends IOApp {
  
  // Global access
  val sem: Semaphore[IO] = 
    Semaphore[IO](1).unsafeRunSync()
  
  
  def someExpensiveTask: IO[Unit] = 
    IO.sleep(1.second) >>
      putStrLn("expensive task") >>
      someExpensiveTask

  new LaunchMissiles(sem).run  // Unit

  def p1: IO[Unit] = 
    sem.withPermit(someExpensiveTask) >> p1(sem)  

  def p2: IO[Unit] = 
    sem.withPermit(someExpensiveTask) >> p2(sem)  

  def run(args: List[String]): IO[ExitCode] = 
        p1.start.void *> p2.start.void
           *> IO.never.as(ExitCode.Success)
}
````

- In this example, we lost the control on sharing our data structure, because we don't know what ``LaunchMissiles``
    is doing internally, so it can't take the permit and never releases it, which will be block our 2 programs
    p1 and p2.