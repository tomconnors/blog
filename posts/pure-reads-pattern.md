Title: A Pattern for Removing Impure Reads from Otherwise Pure Functions
Date: 2026-06-29
Tags: clojure

Sometimes I find I've got a function like this:

```clojure
(defn calculate-something [a]
  (if (some-test? a)
    (let [b (read-from-db a)]
      (some-transform a b))
    (some-other-transform a)))
```

Basically, I need to read from the database (or any other impure place) in the middle of the function, after checking whether the read is necessary/possible.

That function has a signature like `[& params] => result`. A quick way to make that function pure is to give it a signature of `[memo & params] => effect`, where `effect` is either `[:result some-data]` or `[:query & query-params]`.

So we'd write the above as:

```clojure
(defn calculate-something [memo a]
  (let [q [:read-from-db a]
        a? (some-test? a)]
    (cond
      (and a? (contains? memo q))
      (let [b (get memo q)]
        [:result (some-transform a b)])

      a? [:query q]

      :else [:result (some-other-transform a)])))
```

Then in another layer of the app where purity isn't required, call the function in a loop until it returns a result, and interpret :query forms as needed.

```clojure
(defn get-something! [conn a]
  (loop [memo nil i 0]
    (when (> i 10) (throw (ex-info "Too many queries")))
    (let [[cmd data] (calculate-something memo a)]
      (case cmd
        :result data
        :query  (let [memo (assoc memo data (execute-query! conn data))]
                  (recur memo (inc i))))))
```

You could make this support getting multiple queries to execute per call of the `calculate-something` function.

`execute-query!` could be a multimethod or a simple `case` expression.

You could also make the impure wrapper agnostic of the pure function it calls:

```clojure
(defn run-with-queries [conn f & params]
  (loop [memo nil i 0]
    (when (> i 10) (throw (ex-info "Too many queries")))
    (let [[cmd data] (apply f memo params)]
      (case cmd
        :result data
        :query  (let [memo (assoc memo data (execute-query! conn data))]
                  (recur memo (inc i)))))))
```
