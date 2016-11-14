---
title: JavaScript async.serial + async.parallel to ClojureScript core.async
description: How to simulate the JS async library features with Clojure's core.async
---

### A Little Background

In this post, I'll show you how you can convert JavaScript async functions to use ClojureScript's `core.async`. The way Clojure handles parallelism is by using [CSP](https://en.wikipedia.org/wiki/Communicating_sequential_processes) (Communicating Sequantial Processes). In a nutshell, it uses `channels` that you can feed data into and extract data out of. Another popular technique for parallelism is the `Actor model`, popularized by Erlang, which provides special types called `Actors` that essentially have mailboxes that they can read from and can send mail to other Actors. Actors can be loosely thought of as an extra implementation on top of CSP. One of the reasons Clojure chose CSP is that it's more loosely coupled -- channels are first-class and can be passed around freely without knowing it's origin. With Actors, you need to know who you are sending the message to.

The traditional JavaScript techniques for doing async is to provide non-blocking functions that call a callback function when it's complete. This is also loosely coupled and follows the [Hollywood Principle](https://en.wikipedia.org/wiki/Inversion_of_control), but it has had a bad reputation for creating heavily nested chains of callbacks known as the "Pyramid of Doom" or "Callback Hell". 

Eventually `Promises` were developed. This solved the heavy nesting that can result from the plain callback approach, but it still requires chaining Promises together and returning a new Promise for each resolved one. An even more recent development is the use of `Observables`, or FRP, which by it's very nature is functionally pure and also handles event streams in a higher-level way. The problem with this approach is that combining Observables is a little awkward, and it introduces additional complexity with the concept of Hot and Cold Observables.

My favorite approach though is the use of `Futures`, which has been inspired and adopted by the recent developments in the functional JavaScript community ([Ramda](https://github.com/ramda/ramda), [Sanctuary](https://github.com/sanctuary-js/sanctuary), [Fantasy Land](https://github.com/fantasyland/fantasy-land), etc). The Future is like a Promise except that it's lazy and only evaluated when `fork` is called. Future libraries that implement Fantasy Land will implement the `Monad` specification (Promises are monads also), and so you can chain Futures together using Kliesli composition (`pipeK`) or with [do notation](https://github.com/Risto-Stevcev/do-notation).

In this post I'll actually use another popular JavaScript approach, which is to use the [async](https://github.com/caolan/async) library, and I'll translate `async.serial` and `async.parallel` into the ClojureScript `core.async` equivalent. The `async.serial` function takes a list of async functions and executes them in parallel -- so once the first function is done, it calls the second until that's done, which calls the third, and so on. The `async.parallel` function executes all callbacks at the same time or "in parallel" (though we'll ignore the fact that in the JS world everything is serial under the hood because of the event loop), and then returns the result in the order that it was originally given. So with `async.parallel`, for example, if the first callback finishes later than the second, it will still be the first element in the array when it gets returned (preserves ordering).


### Getting Started

I'll create a new ClojureScript project first:

```bash
$ lein new figwheel async-to-async
$ cd async-to-async/
```

And I'll include the [cljs-ajax](https://github.com/JulianBirch/cljs-ajax) (version "0.5.5") library as a dependency in `project.clj`, and start up figwheel:

```bash
$ lein figwheel
```


### Translating Async Callback Functions to Core.Async Functions

In this section I'll show you how you can translate callback-style JS functions into `core.async` style functions. I'll wrap the `ajax.core/GET` function so that it returns a channel rather than providing a callback. Here is how my `core.cljs` file looks like:

```clojure
(ns async-to-async.core
  (:require-macros [cljs.core.async.macros :refer [go]])
  (:require [cljs.core.async :as async :refer [<! put! chan]]
            [ajax.core]))

(defn GET [url]
  (let [out (chan)
        handler #(put! out %1)]
    (ajax.core/GET url {:handler handler :error-handler handler})
    out))

(defn on-js-reload [] )
```

I've just included the `core.async` utilities I'll use and the `ajax.core` library and the simple GET wrapper function. All it does is it creates and returns a channel which gets fed the result from the `ajax.core/GET` request for the given url once the handler callback returns.


### Imitating async.serial

```clojure
(ns async-to-async.core
  (:require-macros [cljs.core.async.macros :refer [go]])
  (:require [cljs.core.async :as async :refer [<! put! chan]]
            [ajax.core]))

(defn GET [url]
  (let [out (chan)
        handler #(put! out %1)]
    (ajax.core/GET url {:handler handler :error-handler handler})
    out))

(enable-console-print!)

;; Fixed number of requests

(go
  (let [users [(<! (GET "http://jsonplaceholder.typicode.com/users/1"))
               (<! (GET "http://jsonplaceholder.typicode.com/users/2"))]]
    (println users)))


;; Variable number of requests

(go
  (let [users (<! (go
                    (let [usrs (atom [])]
                      (dotimes [n 3]
                        (swap! usrs conj (<! (GET (str "http://jsonplaceholder.typicode.com/users/" (inc n))))))
                      @usrs)))] 
    (println users)))

(defn on-js-reload [])
```

The key to imitating `async.serial` is to use the `<!` function. The core.async docs state that `<!` will park if nothing is available in that channel, which by extension means that it will wait for the first request to finish and that channel to close before going to the second one, and so on. You can confirm this by opening the Dev Tools Network panel, and you'll see that the second request doesn't start until the first one finishes.

Unfortunately, there are serious deficits to `core.asyc`, the biggest pain point being that `go` stops evaluating at function boundaries. As a result, you can't really use `<!` with `map`, so you have to simulate map-like behavior. In this case I'm using an atom with `dotimes`, which doesn't allocate closures internally.


### Imitating async.parallel

```clojure
(ns async-to-async.core
  (:require-macros [cljs.core.async.macros :refer [go]])
  (:require [cljs.core.async :as async :refer [<! put! chan]]
            [ajax.core]))

(defn GET [url]
  (let [out (chan)
        handler #(put! out %1)]
    (ajax.core/GET url {:handler handler :error-handler handler})
    out))

(enable-console-print!)

;; Fixed number of requests

(go
  (let [channels [(GET "http://jsonplaceholder.typicode.com/users/1")
                  (GET "http://jsonplaceholder.typicode.com/users/2")]]
    (println (<! (async/map (comp vec list) channels)))))


;; Variable number of requests

(go
  (let [channels (mapv #(GET (str "http://jsonplaceholder.typicode.com/users/" (inc %))) (range 3))]
    (println (<! (async/map (comp vec list) channels)))))

(defn on-js-reload [])
```

The key to simulating `async.parallel` is to use `core.async/map` on a collection of channels. The function allows you to effectively convert the collection of channels into another object but returning only a single channel, in this case a vector. Then you can use `<!` to take the final result. Note that because it doesn't need to park for each request, it doesn't have to work around the limitations of the `go` macro like the equivalent serial version did, so it's implementation is simpler.
