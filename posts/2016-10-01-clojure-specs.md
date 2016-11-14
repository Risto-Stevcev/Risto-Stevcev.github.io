---
title: Clojure Specs
description: A brief summary of the powerful new specs feature in Clojure
---

The Clojure community recently introduced a powerful new language feature called 
specs, which is available for both Clojure and ClojureScript. It's intended to 
replace the need for core.typed, and introduces some powerful features into the 
language to ensure type safety. Specs provide a flexible way to write type signatures. 

- <i class="fa-li fa fa-code"></i>**Instrumentation**  
  With `instrument`, you can have your specs type check at runtime. So if you try 
  to call a function that takes a single parameter that's a `string?`, and you give 
  it a number, this will throw an error instead of attempting to execute the 
  function body.

- <i class="fa-li fa fa-code"></i>**Generative tests for free**  
  Specs also give you generative tests for free. This is very convenient because 
  specs are predicate based, and so this saves you from writing the extra boilerplate 
  for generative tests.

- <i class="fa-li fa fa-code"></i>**Simulating compile-time type checking**  
  You can simulate compile-time type checking by adding generative tests for each 
  of the specs and having them run with figwheel. These tests will run immediately. 
  It's like compile-time type checking in some ways because the tests will fail if 
  the types don't match without having to run your application's code.

- <i class="fa-li fa fa-code"></i>**Simulating refinement and dependent types**  
  Since specs are based on predicate expressions, they are a lot like refinement 
  types such as Liquid Haskell. And since the types are values, you get something 
  like dependent types. It should be possible to write Idris-like proofs to some 
  capacity.
