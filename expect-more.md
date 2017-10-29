# Expecting More

While being able to expect equality or some predicate's truthiness of your actual computed value is often good enough, there are times when you want to expect several different things of that value and it would be painful if you had to write multiple tests like this:

```clojure
(defexpect my-function-works
  (expect String (my-function 42))
  (expect seq (my-function 42))
  (expect #"foo" (my-function 42)))
```

Fortunately, Expectations let you expect multiple things at once with the `more` macro:

```clojure
(defexpect my-function-works
  (expect (more String seq #"foo") (my-function 42)))

;; which can be written even more succinctly as:

(defexpect my-function-works (more String seq #"foo") (my-function 42))
```



