---
title: My top 4 compilation targets
description: My top 4 languages to target when writing a compiler
---

There's no real magic to compilers, it's all about translating from one language to another. For compilers that compile 
to machine code, that means translating code to assembly language, a human-readable format of the machine language that 
the computer's hardware understands that gets compiled and linekd by a type of compiler called an assembler. 

Here are my top 4 compilation targets:


- <i class="fa-li fa fa-code"></i>
  **JVM** - Probably the most obvious one, but the JVM bytecode is freely available, and it allows languages to leverage 
  years of work that has been written in Java. The startup time of the JVM is slow, but the bytecode and the interpreter 
  have been heavily optimized to run fast.

- <i class="fa-li fa fa-code"></i>
  **JS** - Javascript is probably the most popular compilation target these days, primarily because the rapid growth of 
  the frontend in recent years due to progress in computer power and the standalone app market shifting towards the web 
  and mobile. Javascript is already surprisingly fast for an interpreted language due to clever optimizations that 
  modern JS engines such as V8 have implemented. And with WebAssembly on the horizon, JS has the potential to reach 
  speeds as fast and possibly faster than the JVM.

- <i class="fa-li fa fa-code"></i>
  **LLVM** - One of the biggest challenges to writing compilers that compile to machine code is being able to target 
  many different machine architectures. Before LLVM, compiler authors had to convert the language AST into the 
  appropriate assembly, link and assemble that, and then spit out the binary. This took up a lot of the time in writing 
  compilers. Another time consuming task was writing algorithms that apply performance optimizations to the code.
  LLVM is revolutionary because compiler authors are only required to target LLVM, which can then be compiled to the 
  architecture that the machine is running (MIPS, x86, etc) via LLVM. LLVM also comes with a lot of the most common compiler 
  optimizations, as well as a set of backends to convert from LLVM to a target language.

- <i class="fa-li fa fa-code"></i>
  **BEAM** - This is the virtual machine that Erlang runs on. Writing a language that targets BEAM means that you can take 
  advantage of a lot of the benefits of Erlang: easy distributed programming and clustering, fault tolerance, actors, 
  and the ability to spawn fast lightweight processes.  

Out of all of these, LLVM is by far the most powerful, but it's still relatively new. There is already an LLVM to JS 
backend called Emscripten which compiles to Asm.js, which the initial implementation of Web Assembly will be based on.
