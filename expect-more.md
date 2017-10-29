# Expecting More

While being able to expect equality or some predicate's truthiness of your actual computed value is often good enough, there are times when you want to expect several different things of that value and it would be painful if you had to write multiple tests like this:

```clojure
(defexpect my-function-works
  (expect String (my-function 42))
  (expect seq (my-function 42))
  (expect #"foo" (my-function 42)))
```

## more

Fortunately, Expectations let you expect multiple things at once with the `more` macro:

```clojure
(defexpect my-function-works
  (expect (more String seq #"foo") (my-function 42)))

;; which can be written even more succinctly as:

(defexpect my-function-works (more String seq #"foo") (my-function 42))
```

## more-of

Or perhaps you need more than just predicates? Perhaps you need multiple expectations on some or all of the computed value? The `more-of` macro allows you to write pairs of expected / actual expressions, based on the computed value. For example, rewriting the above using `more-of`:

```clojure
(defexpect my-function-works
  (more-of value
    String value
    seq value
    #"foo" value) (my-function 42))
```

In this case, the `more` form is simpler, but `more-of` is like a let binding so full destructuring forms are possible:

```clojure
(defexpect pieces
  (more-of [a b]
    1 a
    even? b)
  (range 1 5))

(defexpect elements
  (more-of {:keys [x y]}
    seq x
    :a (in y))
  {:x [1 2] :y [:a :b]})
```

These can also be rewritten using `let` and a series of `expect` forms:

```clojure
(defexpect pieces
  (let [[a b] (range 1 5)]
    (expect 1 a)
    (expect even? b)))

(defexpect elements
  (let [{:keys [x y]} {:x [1 2] :y [:a :b]}]
    (expect seq x)
    (expect :a (in y))))
```

## more-&gt;

In addition to `more` and `more-of`, there is a threading version `more->` that would let us write those tests like this:

```clojure
(defexpect pieces
  (more-> 1 first
          even? second)
  (range 1 5))

(defexpect elements
  (more-> seq :x
          :a (-> :y (in)))
  {:x [1 2] :y [:a :b]})
```

## from-each

So far, all of the expectations have worked with a single computed value. Sometimes we might want to write expectations over a series of computed values:

```clojure
(defexpect my-function-works-a-lot
  (more String seq #"foo")
  (from-each [n (range 42 50]
    (my-function n)))
```

This allows us, very succinctly, to expect that calling `my-function` on the range of values still satisfies all of the predicates specified in `more`.

`from-each` behaves much like Clojure's `for` comprehension and supports both `:when` and `:let` so you can write very expressive tests:

```clojure
(defexpect oddities
  (expect odd? (from-each [num [1 2 3]
                           :when (not= num 2)]
                 num))

  (expect odd? (from-each [num [1 2 3]
                           :when (not= num 2)
                           :let [numinc1 (inc num)]]
                 (inc numinc1))))
```



