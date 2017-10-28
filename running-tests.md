# Running Tests

It is common practice to place your tests under a `test` folder, in a structure that matches your source code, with test files having a suffix of `_test`:

```
project.clj ; or build.boot
src/
  myapp/
    core.clj
test/
  myapp/
    core_test.clj
```

Let's put a \(broken\) function into `src/myapp/core.clj` that we wish to test \(and fix\):

```clojure
(ns myapp.core)

(defn sum
  "I try to sum up a collection of numbers"
  [coll]
  (loop [total 1 input coll]
    (if (next input)
      (recur (+ total (first input)) (next input))
      total)))
```

Now we'll write some tests for `sum` in `test/myapp/core_test.clj`:

```clojure
(ns myapp.core-test
  (:require [expectations.clojure.test :refer :all]
            [myapp.core :refer :all]))

(defexpect empty-sum 0 (sum []))

(defexpect simple-sums
  (expect 6 (sum [1 2 3]))
  (expect 10 (sum [1 2 3 4])))
```

See the sections below for how to run your tests with Boot or Leiningen. When you run these tests, they'll fail:

```
FAIL in (simple-sums) (core_test.clj:8)
produced: 4 from (sum [1 2 3])
failure is expected: 6 
                was: 4
expected: 6
  actual: (sum [1 2 3])

FAIL in (simple-sums) (core_test.clj:9)
produced: 7 from (sum [1 2 3 4])
failure is expected: 10 
                was: 7
expected: 10
  actual: (sum [1 2 3 4])

FAIL in (empty-sum) (core_test.clj:5)
produced: 1 from (sum [])
failure is expected: 0 
                was: 1
expected: 0
  actual: (sum [])

Ran 2 tests containing 3 assertions.
3 failures, 0 errors.
```

When we see that `(sum [])` produced `1` instead of `0`, we quickly realize that we should be initializing the `total` to `0` in the loop! We fix that and run the tests again:

```
FAIL in (simple-sums) (core_test.clj:8)
produced: 3 from (sum [1 2 3])
failure is expected: 6 
                was: 3
expected: 6
  actual: (sum [1 2 3])

FAIL in (simple-sums) (core_test.clj:9)
produced: 6 from (sum [1 2 3 4])
failure is expected: 10 
                was: 6
expected: 10
  actual: (sum [1 2 3 4])

Ran 2 tests containing 3 assertions.
2 failures, 0 errors.
```

We see that `empty-sum` no longer fails, but we don't seem to be counting the last element of our collection. Ah, we're calling `next` in the `if` test _and_ in the loop! We'll change the `if` test to `(seq coll)` and run our tests again:

```
Ran 2 tests containing 3 assertions.
0 failures, 0 errors.
```

Success!

## Boot

Boot doesn't come with a test runner out-of-the-box so you will need to add the following to your `build.boot` file \(in addition to adding the `expectations` dependency\):

```clojure
(merge-env! :source-paths #{"test"} ; source paths are not added to build/deploy artifacts
            :dependencies '[[adzerk/boot-test "RELEASE" :scope "test"]])
(require '[adzerk.boot-test :refer [test]])
```

If you now run `boot test` you'll see that it looks for tests in all of your namespaces, including your source namespaces. We only want to look in namespaces that end in `-test`, so we'll use the `-I` \(`--include`\) option to limit the namespaces:

```
boot test -I -test$
```

The `--include` option takes a regular expression and uses it to include only matching namespaces in its search for tests. There is also a `-X` \(`--exclude`\) option to exclude matching namespaces.

## Leiningen

Leiningen has a built-in task `test` to run your tests: `lein test`

## Atom/ProtoREPL

## IntelliJ



