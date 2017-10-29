# Beyond Equality

In [Getting Started](/README.md) and [Running Tests](/running-tests.md), we saw how to write basic equality expectations and we hinted at some of the more advanced expectations that are supported. In this section, we'll explore predicate-based expectations and some of the shorthand forms that Expectations provides, and then look at what Expectations provides for writing tests around collections.

## Predicate Expectations

The expected outcome can be a function and Expectations will call it on the actual value and if the result is truthy, the expectation will succeed. If the result is falsey, the expectation will succeed:

```clojure
(defexpect predicates
  (expect odd? 13) ; pass
  (expect map? [1 2 3]) ; fail
  (expect (partial instance? String) "foo") ; pass
  (expect (partial re-find #"foo") "bar")) ; fail 
```

If we run these tests, we'll see the two failures as expected:

```clojure
FAIL in (predicates) (core_test.clj:13)
failure is [1 2 3] is not map?
expected: map?
  actual: [1 2 3]

FAIL in (predicates) (core_test.clj:15)
failure is "bar" is not (partial re-find #"foo")
expected: (partial re-find #"foo")
  actual: "bar"
```

Expectations provides a shorthand for type-based and regex-based tests:

```clojure
(defexpect predicates
  ...
  (expect String "foo") ; pass
  (expect #"foo" "bar")) ; fail 
```

In addition to being shorter, this also produces a better failure message:

```clojure
failure is regex #"foo" not found in "bar"
expected: #"foo"
  actual: "bar"
```

Similarly, if a type-based test fails:

```clojure
  ...
  (expect String 42)
  ...

  wanted: java.lang.String from String
expected: 42 to be an instance of class java.lang.String
     was: 42 is an instance of class java.lang.Long
expected: String
  actual: 42
```

Expectations provides a couple of predicates of its own: `approximately` and `functionally`. The former is useful when working with floating point numbers, where strict equality is not appropriate:

```clojure
(defexpect floating-point
  (expect (approximately 0.333) (/ 1 3)) ; pass
  (expect (approximately 0.333 0.001) (/ 1 3)) ; pass -- 0.001 delta is the default
  (expect (approximately 1.0 0.5) (/ 3 4)) ; pass -- 0.75 is within 0.5 of 1.0
```

The `functionally` predicate is useful when you have two different functions that should produce the same result from the same input:

```clojure
  (expect (functionally identity (comp reverse reverse)) (range 10))
```

If you are expecting your computation to throw an exception, you might think to write an expectation like this, using the type-based shorthand mentioned above:

```clojure
  (expect ArithmeticException (try (my-function 42) (catch Throwable t t)))
```

Fortunately, Expectations provides a shorthand for that:

```clojure
  (expect ArithmeticException (my-function 42))
```

## Collections

When your actual computation returns a collection, you often expect certain values to be in that collection. Expectations supports this sort of expectation for that:

```clojure
  (expect expected-value (in actual-collection))
```

This is supported for hash maps \(expecting key/value pairs to be in the map\), sets, vectors, and lists:

```clojure
(defexpect collections
  (expect {:foo 1} (in {:foo 1 :cat 4}))
  (expect :foo (in #{:foo :bar}))
  (expect :foo (in [:foo :bar]))
  (expect :bar (in (list :foo :bar)))

  (expect {:foo 2} (in {:foo 1 :cat 4}))
  (expect :baz (in #{:foo :bar}))
  (expect :baz (in [:foo :bar]))
  (expect :baz (in (list :foo :bar))))
```

The first four all pass. The last four all fail:

```clojure
failure is expected: {:foo 2} in {:foo 1, :cat 4}
in expected, not actual: {:foo 2}
in actual, not expected: {:foo 1, :cat 4}
expected: {:foo 2}
  actual: (in {:foo 1, :cat 4})

failure is val :baz not found in #{:bar :foo}
expected: :baz
  actual: (in #{:bar :foo})

failure is val :baz not found in [:foo :bar]
expected: :baz
  actual: (in [:foo :bar])

failure is val :baz not found in (:foo :bar)
expected: :baz
  actual: (in (list :foo :bar))
```

Again, we see that Expectations produces more detail beyond `clojure.test`'s basic `expected:` / `actual:` messages. In particular, for hash maps, Expectations reports the difference between the data structures.

