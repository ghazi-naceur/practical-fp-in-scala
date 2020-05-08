1- State Monad :
- 
- If the state is sequential, it is safer to use ```State``` monad, which has a ``run: S => (S, A)`` function, with 
``S`` is the old state, and the result contains the new state with the result of the state transition (if any).
- The ``State`` monad is inherently sequential, so there is no way to run 2 ``State`` actions in parallel.
For example :
````scala
val nextInt: State[Int, Int] = 
    State(s => (s + 1, s * 2)

def seq: State[Int, Int] = 
    for {
        n1 <- nextInt
        n2 <- nextInt
        n3 <- nextInt
    } yield n1 + n2 + n3
````
In this example, ``State`` is threaded sequentially after each flatMap call that returns the new state that is used to 
run on the following instruction => This will definitely not work in a parallel setup.

2- Atomic Ref :
-
Our counter is wrapper in an interface that could be invoked from anywhere at the same, so we have a concurrent situation.
``State`` won't work in this case.
As a solution, we can use ``Ref`` (Cats Effect), which is a pure functional model of a concurrent mutable reference. It provides
``update`` and ``modify`` **atomic** functions. (Internally, it uses *compare_and_set* loop)