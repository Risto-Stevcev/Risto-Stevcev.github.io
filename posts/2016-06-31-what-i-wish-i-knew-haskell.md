---
title: What I wish I knew when I was learning Haskell
description: What I wish I knew when I was learning Haskell
---

Every developer should learn Haskell, hopefully sooner rather than later. Even if you don't like Haskell or you 
don't plan on using it for work, the tools and perspectives that you leave with end up being invaluable for writing 
correct and solid code in any language.

However, since Haskell is so different from other languages and the learning curve substantially higher than most languages, 
a lot of developers eager to learn it often end up burning out. In a lot of cases, developers end up revisiting it several 
times before they fully grok it. These are some of the things I wish someone told me when I was learning Haskell. 
This post is inspired by [Stephen Diehl's article](http://dev.stephendiehl.com/hask/).

 
- <i class="fa-li fa fa-code"></i>
  **Until recently, learning resources weren't that great**  
  The most common resources for learning Haskell have been Learn you a Haskell and Real World Haskell. Both of these 
  resources are generally not complete enough to get a full picture of Haskell. In recent times this have greatly 
  improved, with many tutorials to learn category theory, Stack Overflow answers, and new books like Haskellbook. If I 
  could go back, I would have spent most of that time trying to write code, and only consulting learning materials 
  when I felt stuck.

- <i class="fa-li fa fa-code"></i>
  **Monads barely scratch the surface**  
  A lot of people learning Haskell think that monads are the most important concept, and once learned then they can 
  officially say that they know Haskell. However, there's so many other category theory types which are very common 
  in Haskell code and other languages that use category theory, such as monad transformers, free structures, 
  covariant, contravariant and invariant functors, natural transformations, leibnitz equalities, coroutines, rings, 
  heyting algebras, and much more.

- <i class="fa-li fa fa-code"></i>
  **Not all category theory is useful to programmers**  
  At some point you might have a desire to learn all of the category theory constructs because you have a feeling it 
  might come in handy someday, but some of it just isn't useful to programming at all. The yoneda lemma is one of those 
  things. Other times, developers just haven't played around with it enough to find out what it might be useful for.

- <i class="fa-li fa fa-code"></i>
  **You don't need to know category theory to use category theory**  
  You don't need to know the math behind category theory to use the category theory constructs in Haskell. You will get 
  a natural sense of how it all works when you start using them without diving into any kind of theory.

- <i class="fa-li fa fa-code"></i>
  **There is the core language and then there are language extensions**  
  Besides the core language, there are also a lot of language extensions that you can use by declaring pragmas at the 
  top of your modules. Haskell devs use lots of extensions, and Haskell has become a common language for researchers to 
  experiment with new language features. This can be both exciting and daunting. It's exciting because as a Haskell 
  user you are often at the forefront of new developments in language design, but it's also daunting to have to stay 
  on top of all of these concepts and to take the time to learn them. 

- <i class="fa-li fa fa-code"></i>
  **Functions don't have to be total**  
  Haskell is touted as a very safe language, but one of the largest and well-deserved criticms is that functions can 
  be partial, or in other words not well defined for all possible inputs to the function. What's worse is that a lot of 
  the core libraries have partial functions, such as getting the `head` of a list, which ends up throwing a runtime error for 
  empty lists. In contrast, similar languages like OCaml and Purescript enforce totality and have exhaustive pattern matching, 
  which means that your code will not compile if it's not well defined. The good news though is that Haskell developers 
  tend to keep their functions total by their own discipline, and you can turn on settings that will give you a warning 
  if your pattern matching is non-exhaustive.

- <i class="fa-li fa fa-code"></i>
  **Laziness can lead to annoying space leaks**  
  Haskell is a lazy language, which is in some sense a form of normal order evaluation. This can lead to really elegant design, 
  but it comes with a signiticant cost of space leaks. With lazy evaluation, haskell will not evaluate expressions until it 
  needs them. This can lead to large unevaluated chunks of code in memory called thunks, which can eat up a lot of 
  memory. What's worse is that it's often not easy to figure out the space complexity of your program. This is what is 
  meant when people talk about space leaks with Haskell. The bright side to this is that GHC is getting better at optimizing 
  thunks, so that space leaks are as minimal as possible.

- <i class="fa-li fa fa-code"></i>
  **Type inference fails sometimes, often on more complex types**  
  The first two things that most people end up loving is type inference and pattern matching. But eventually every dev 
  ends up coming to a situation where the type inferencer ends up inferring the wrong types. If you aren't accustomed 
  to realizing that this can happen, it can take a substantial amount of time to debug why your program isn't 
  working. Which is why it's considered good practice to keep types explicit in Haskell, but you can still take advantage 
  of the elegance of inferred types inside the function body. A good practice in Haskell which Rust enforces on the 
  language level, is to add type signatures to functions, but to use type inference inside function bodies. This seems 
  to be a pretty good sweet spot to avoid unnecessary verbosity while ensuring that your programs are well defined and 
  easy to debug.

- <i class="fa-li fa fa-code"></i>
  **Types can get pretty complex**  
  Types in Haskell have a tendency to get pretty complex. It's pretty common to see transformer stacks with 3 or more 
  layers in haskell code. You get used to this after coding in Haskell or similar languages like Purescript for a while.
