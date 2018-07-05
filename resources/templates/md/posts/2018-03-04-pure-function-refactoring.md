{:title "Pure function refactoring"
 :layout :post
 :tags  ["Clojure" "Refactoring" "Pure Function"]
 :toc true
 :draft? false}


## Towards an easier path

### The story

Some time ago an application has been ported from Java to Clojure, fine!
After that, as usually happens for living software, some changes were needed to bring in,
and ... I got stuck for a while trying to figure out what was the best path to undertake
evolving a portion of code, which retained a little bit of pollution from the original
Java/IDE/imperative mindset - _functions tangled with side effects_.

### The problem

Here is the code:

```clojure
(defn)
```

The function do several computations, and at the end save the results to database.
In order to do changes ensuring the correct result I'd need to test it, writing
some tests, and evolving the changes interacting with them at some REPL,
but the database call made it all harder than necessary to put in practice.

At first my mind started shifting among the setup of a testing environment fully provided with a database,
or mocking some methods to bypass the database needing, lazily accepting the code the way as given,
balancing out the pros and the cons of either one or the other...
But code smell was really strong... :)

### The solution

Solution in the end was the simplest, as you, dear reader, certainly already know.  
Apply the principles of good software programming, keep separate side-effects
from business logic, strive for pure functions, which incidentally are among the core tenets of functional programming.  
I do not need to test the database, looking at if the db call persists results.  
I just need to test the business logic.

Let's do it:

```clojure
```


The refactoring consisted in moving out on a new function that takes as an argument the
computed data the database call, and whose only purpose is persisting it.

The original function can be now invoked from written tests as many time as I want,
can be growed interactively from the REPL,
and I don't need to worry about cumbersome dependencies anymore.

### In conclusion

It is not that in Java/IDE/imperative mindset you cannot write good software.
In this example the refactoring should have had to be done with these tools too.

The problem IMHO is that you are more prone to adopt solutions that at first sight
look convenient - like throwing some breakpoint in the IDE before critical code
in order to stop execution and manually run the software again and again - but
at the end impact the overall design, making it hard to evolve, test and read.

FP push you towards separations of concerns, and the result you come with up,
having a piece of code that makes just one thing, a **pure function**,
on what you can concentrate
clearly without being distracted by other stuff, makes feel you more confident with the code you wrote.

