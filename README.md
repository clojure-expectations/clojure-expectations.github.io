# Getting Started

Expectations lets you write concise, elegant, yet powerful tests. Each expectation has an expected outcome and an actual expression to be computed. For compatibility with `clojure.test` tooling, expectations can be named.

Add Expectations to your project with `[expectations "2.2.0-rc3"]` in your `:dependencies` \(in the `:dev` profile for Leiningen, or with `:scope "test"` for Boot\) and fire up a REPL to follow along with the examples!

Let's look at some examples of named expectations:

```clojure
(require '[expectations.clojure.test :refer :all])

;; simple equality tests:

(defexpect equality
  (expect 1 (* 1 1))
  (expect "foo" (str "f" "oo")))

;; the expected outcome can be a regular expression:

(defexpect regex-1
  (expect #"foo" "It's foobar!"))

;; since that has only a single expectation, it can be written more succinctly:

(defexpect regex-2 #"foo" "It's foobar!")

;; the expected outcome can be an exception type:

(defexpect divide-by-zero ArithmeticException (/ 12 0))

;; the expected outcome can be a predicate:

(defexpect no-elements empty? (list))

;; the expected outcome can be a type:

(defexpect named String (name :foo))

;; if the actual value is a collection, the expected outcome can be an element or subset "in" that collection:

(defexpect collections
  (expect {:foo 1} (in {:foo 1 :cat 4}))
  (expect :foo (in #{:foo :bar}))
  (expect :foo (in [:bar :foo])))
```

The `defexpect` macro creates a function that contains the test\(s\). You can run each function individually:

```clojure
user=> (equality)
nil
```

If the test passes, nothing is printed, and `nil` is returned. Let's look at a failing test:

```clojure
user=> (defexpect inequality (* 2 21) (+ 13 13 13))
#'user/inequality
user=> (inequality)

FAIL in (inequality) (:1)
  wanted: 42 from (* 2 21)
produced: 39 from (+ 13 13 13)
failure is expected: 42 
                was: 39
expected: (* 2 21)
  actual: (+ 13 13 13)
nil
```

The `FAIL...` / `expected:` / `actual:` lines are produced by `clojure.test`'s standard reporting functionality. The rest is added by Expectations to provide more detail about what actually went wrong.

