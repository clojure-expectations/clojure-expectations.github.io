# Before & After Tests

Sometimes you need to run code either before or after your test suite runs, perhaps to set up your test database and then tear it down, or before or after each individual test to set up data or state for the tests to run. We can write functions to do this work as wrappers around our test suite or around each test:

```clojure
(defn wrap-database [suite]
  (create-database)
  (suite) ; run the test suite
  (drop-database))

(defn wrap-data [test]
  (setup-data)
  (test) ; run the individual test
  (teardown-data))
```

Because `expectations.clojure.test` is compatible with `clojure.test`, the same test fixtures machinery should be used: [clojure.test/use-fixtures](https://clojuredocs.org/clojure.test/use-fixtures). In order to tell the various test runners to use those functions above, we install them as `:once` or `:each` fixtures:

```clojure
(require '[clojure.test :refer [use-fixtures]])

(use-fixtures :once wrap-database)

(use-fixtures :each wrap-data)
```



