{:title "Pure function refactoring"
 :layout :post
 :tags  ["Clojure" "Refactoring" "Pure Function"]
 :toc true
 :draft? false}

## A pratical example

### The story

In my early days in Clojure I translated an application from Java, fine!
It carried on since some days ago I was committed to implement new functionalities.
Looking at the code I realized it was more a code conversion than a rewriting in the spirit and philosophy
of the new programming language. The original Java/IDE/imperative mindset leaked
through all the porting, _loops, state, functions tangled with side effects, and so on._

Here I want to focus in particular on the refactoring of an function which performed a task
in a unclear and confused manner to a version of the function that perform the same task
possibly concise and easily to reason about, a **pure function**.

### The problem

Launch a REPL and try to make some changes to the code below, just some business logic change, and you soon
will realize how heavy coupled it is with different functionalities that you have to take care of before to execute it.

- there are db calls
- there are log calls
- there is a specialized log object

all of them producing side-effects and not required for the task at point.

The gist here is the following:
- you have order lines that must be reset in order to be scheduled for next billing cycle
- each order line is subject to different business policy, shipping dates are renewed and so on...
- finally the order header' s shipping date must be updated with the soonest order line' s shipping date
- you would verify that giving an order detail to the function you get back order lines with correct
  policies applyed

Here is the code[^1]:

```clojure
(defn reset!
  "Reset order state for the next billing."
  [db elog order]

  ;; go through the order detail and reset each line
  ;; when the sequence is completed return the min shipping date

  (let [min-shipping-date
   (loop [xs order min-date (t/date-time 2039 1 1)]
    (if-let [line (first (seq xs))]
     (if (not= "ZSP" (:item-id line)) ; bypass zsp items
      (let [line (reset-line line)]

       ;; side-effects here !!!
       (log/info "Processing order line" (str  (:order_id line) "/" (:order_line_id line)))

       ;; side-effects here too !!!
       (j/update! db :ORDER_LINE line ["order_id=? AND order_line_id=?" (:order_id line) (:order_line_id line)])

	   ;; do other things here, omitted for clarity
	   ;; ...

       (recur (rest xs) (if (t/minus (:shipping_date line) min-date) (:shipping_date line) min-date)))
	 (recur (rest xs) min-date))
	min-date))])

    ;; perform post operations after order detail reset

    ;; side effects here !!!
    (j/update! db :ORDER {:shipping-date min-shipping-date} ["order_id=?" (:order_id line)])

    ;; loop again through the order lines to update zsp items
	(loop [xs order]
	  (when-let [line (first (seq xs))]
	    (when (= "ZSP" (:item-id line))
		  (j/update! db :ORDER_LINE {:shipping-date min-shipping-date} ["order_id=? AND order_line_id=?" (:order_id line) (:order_line_id line)]))
	      (recur (rest xs))))

    (let [msg (str "Resetting order shipping date to "
              (fmt-date min-shipping-date)
              " [" (mut/order-number order) "]")]

        ;; again side effects here !!!
		(.log elog (:customer_id order) :info msg)
          (log/info msg))

     ;; do other things here, omitted for clarity
     ;; ...

```


The function does several computations, it retains almost all of its procedural style roots,
generating side-effects along the way and finally save the results to database.
There is no easy way to play with it interactively at a REPL, the database calls and all other side-effects ops sprung
around made it all harder than necessary to check if the code works properly.

My first reaction was wobbling among two possible ways to bypass the hurdle,

1. setup of a testing environment fully provided with a database,
2. mocking some methods to avoid the database and company at all,

while balancing out the pros and the cons of either one or the other
the code smell was really strong... :)
both the solutions were more cumbersome to implement let alone lazily accepting the smelling code the way as given.


### The solution

The solution in the end was the simplest, as you certainly already figured it out.
Apply the principles of good software programming, keep side-effects distinct
from business logic, strive for pure functions, which incidentally are among the core tenets of functional programming.
I do not need to test the database, looking at it if the db call persists the correct results.
I just need to test the business logic.

Let's do it:

```clojure
(defn save
  [db elog min-shipping-date order]
  (doseq [line order]
    ;; call the db update fn
	;; call log
	;; and so on
))

(defn reset
  "Reset order state for the next billing."
  [order]
  (let [{lines false zsp-lines true} (group-by #(= "ZSP" (get % :item_id)) order)                 ; split resettable lines
        lines                        (map reset-line lines)                                       ; do business logic
        min-shipping-date            (apply t/min-date (map :shipping-date lines))                ; post retrieve the min shipping date
        zsp-lines                    (map #(assoc %1 :shipping_date min-shipping-date) zsp-lines) ; update zsp lines
        lines                        (reduce conj lines zsp-lines)]                               ; join order lines again

	 ;; returns a vector with min-shipping-date and updated order detail
     [min-shipping-date lines]))
```


The refactoring consisted in moving out in a new function
the database and log stuff, passing it the data to persist.

The `reset`'s function output can be provided to the `save` function for persistence.

The original function is now decoupled from db dependency,
can be growed interactively at the REPL, tests can be written
without worrying about having a db or any else impediment in the developing stage.

You can pass some data and you get back a result that can be verifyed if it is correct. Database and logs are not involved anymore.

__And last but not least code size has wonderfully shrunken by a factor, and the intents are clearer IMHO.__


### In conclusion

It is not that in Java/IDE/imperative mindset you cannot write good software.
In this example the refactoring had to be done with these tools too, _Separation Of Concerns_
and _Pure Functions_ are universal programming principles, even tough they are the workhorses of FP.

The problem IMHO is that you are more prone to adopt solutions that at first sight
look convenient - like throwing some breakpoints in the IDE before critical code
is reached to stop execution and inspect the current context, doing it endlessly -
but at the end impacting the overall design, making it hard to evolve, test and read.

FP strongly pushes you towards _separations of concerns_, and the result you come with up,
lines of code that make just one thing, **pure functions**,
on what you can concentrate clearly without being distracted by other meanings,
makes you feel more confident about the code you write.

[^1]: Disclaimer: the code presented here is not the real one as it can not be divulgated, but it is meant to represent it as close to the original as possible.
