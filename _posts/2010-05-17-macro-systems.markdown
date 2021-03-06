---
layout: post
title: Macro systems in Scheme
tags:
- scheme
- lisp
- macros
---

There are several issues that macro systems in Scheme address, and
different kinds of solutions to these. Here are a set of mostly
orthogonal concepts that any macro system has to relate to: (I don’t
describe each concept in detail as that would take too much text to
make this list useful as an overview)

1. A macro system can allow or disallow execution of arbitrary Scheme
   code at macro expansion time. Disallowing it (like for instance
   syntax-rules does) works around a whole haze of problems, mostly
   related to separation of expansion-time and run-time
   environments. On the other hand, allowing it adds significant power
   and adds lots of exciting possibilities. Even though syntax-rules
   is a turing complete system, it’s not as powerful as a typical
   system that allows execution of arbitrary Scheme code at expansion
   time.

2. A macro system can be fully unhygienic (as define-macro in
   Gambit-C), fully hygienic (as syntax-rules is), or allow partial
   unhygiene (like syntactic closures, explicit renaming and
   syntax-case). When allowing partial unhygiene, most systems work by
   either makeing hygiene explicit (like explicit renaming) or by
   making unhygiene explicit (like syntax-case).

3. A macro system can be based on pattern matching (syntax-rules)
   and/or explicit destructuring of the input form (explicit
   renaming). You can always implement a mechanism for pattern
   matching on a system based on explicit destructuring. The reverse
   can be true, but it is not always possible (syntax-rules is an
   example).

4. A macro system can be designed so that macros’ output are plain
   s-expressions (Gambit-C’s define-macro), special “syntax” objects
   (syntax-case), or a hybrid (syntactic closures). Letting the output
   be plain s-expressions feels very natural and is easy to understand
   conceptually. Output can be constructed using quasiquotation. In
   this respect, the hybrid approach works more or less like plain
   s-expressions. Having special “syntax” objects does in my opinion
   add significant conceptual overhead. It does, however, also enable
   other very important features, the most important being that the
   syntax objects can contain source code location. Plain
   s-expressions based on lists and symbols can’t do that.

5. A macro system might or might not allow macros to preserve source
   code location. I know of three major approaches to this problem: 1)
   Gambit-C’s define-macro, explicit renaming and syntactic closures
   do not allow macros to preserve source code location. 2) Gambit has
   an internal style of macros ##define-syntax that gets the Gambit
   compiler’s internal representation of source code with location
   annotations. Macros written in this style must explicitly make sure
   to annotate the source code with location 3) syntax-rules and
   syntax-case macros can be implemented so that source code location
   is preserved automatically and implicitly.

6. Another important aspect of macro systems is that they might or
   might not provide useful tools for useful syntax error
   reporting. Explicit destructuring of the input form usually results
   in either useless errors (like cdr on a non-pair error) or lots of
   error handling code that needs to be written for every
   macro. Pattern matching is a big gain here; it makes it possible
   for the macro system to provide at least somewhat useful
   errors. syntax-case provides some good tools for this, but I think
   this could be made even better.

7. A macro system can play more or less well with others. For
   instance, syntax-rules can coexist with pretty much any other
   hygienic system. It’s often not very difficult to make non-hygienic
   systems play well with each other. Explicit renaming and syntactic
   closures seem to work well together. I don’t know of any way that
   works well to combine pure Gambit-C define-macro with any kind of
   hygienic system. syntax-case is quite exclusive since it restricts
   macros to be procedures of one argument, which is exactly the
   format that syntax-case destructures.

8. An implementation of macros can restrict itself to one single type
   of macros, for instance syntax-rules (many simpler R5RS
   implementations), syntax-case (most (all?) R6RS implementations) or
   Gambit-C’s define-macro (Gambit-C 4). Some macro systems are
   powerful enough to implement other macro system on them,
   syntax-rules can for instance be implemented in terms of
   syntax-case or syntactic closures. Explicit renaming can be
   implemented in terms of syntactic closures (and possibly also the
   other way around but I’m not sure). Some are not. For instance,
   it’s not possible to implement syntax-rules in terms of Gambit-C
   define-macro or vice versa.

   Or, a macro system implementation can choose to use some algorithm
   internally that is powerful enough to implement several macro
   systems and expose interfaces to them separately. This is sometimes
   useful, but not very often, because many algorithms are so tightly
   knit to the system that it implements that it can’t do any other
   thing; to the extent it’s possible other macro systems can just as
   easily be implemented above the macro expansion algorithm
   layer. For instance, there’s no reason to implement an algorithm
   that can do both syntax-case and syntax-rules, because syntax-rules
   is trivial to implement with only syntax-case. And the common
   (only?) algorithm for implementing syntax-case can’t be used to
   implement syntactic closures.  These are some of the most important
   design decisions that can be made when designing a macro system. In
   practise, it’s a trade-off between ease of use, ease of
   understanding, and power (and power can be measured in several
   different ways). I wish that there was a macro system that is
   hygienic, automatically preserves source code location information,
   allows execution of arbitrary Scheme code, is extensible and is
   easy to understand. But so far, no one seems to have succeeded to
   do this. I would love to be able to write macros in Common
   Lisp-style, using quasiquote or similar, with powerful but optional
   pattern matching tools.

So us Schemers have to live with compromises, having to learn several
macro systems or one highly complex one. syntax-rules is great for
simple macros but it doesn’t work very well for complex
macros. Syntactic closures is good, but it’s not always easy to avoid
hygiene mistakes. syntax-case is good, but it IMO adds many
non-Schemey concepts; one of my main gripes with it is that it
introduces a new concept of pattern variable bindings which are
distinct from macros and normal variable (“lambda”) bindings. To me
that is a significant step back from Scheme’s simplicity. Maybe I’ll
just have to live with that.
