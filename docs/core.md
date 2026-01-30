---
layout: docs
---

# Symp Core Specs

> **[about document]**  
> Specification of *Symp Core*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
> Symp Core is a minimal symbolic computation framework based on S-expressions and term rewriting. Its execution model is defined entirely by structural reduction rules, without variables, environments, mutable state, or implicit control flow. This document specifies the core syntax and semantics of Symp Core, providing a formal grammar, an informal semantic model, and illustrative examples. The aim is to present Symp Core as a small, deterministic foundation for symbolic processing, language experimentation, and reasoning about computation through explicit term transformation.


## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

Symp Core is a deliberately minimal language designed to explore computation as **structural transformation of symbolic expressions**. Rather than relying on variables, environments, or mutable state, Symp Core defines all behavior in terms of S-expression reduction driven by explicit rewrite rules.

The language is intended as a *core calculus*, not a full-featured programming language. Its design favors simplicity, predictability, and semantic transparency over convenience or performance. Every construct in Symp Core has a small and well-defined meaning, and there are no hidden evaluation rules beyond those explicitly specified.

Symp Core is suitable as:

* a foundation for higher-level symbolic or functional languages,
* a target for experimentation with alternative semantics,
* a teaching tool for reduction-based computation,
* or a compact system for symbolic processing and term manipulation.

This specification describes Symp Core, focusing on its syntax, reduction semantics, and core operational behavior.

## 2. Theoretical Background

Symp Core draws inspiration from several established ideas in programming language theory, including S-expressions, term rewriting systems, and functional reduction semantics. However, it intentionally avoids many features commonly found in functional languages, such as variable binding, closures, and implicit evaluation strategies.

Instead, Symp Core adopts the following guiding principles:

* **Uniform representation**  
  Code and data share the same S-expression structure.

* **Explicit computation**  
  Only expressions whose head resolves to a reducible identifier can cause computation.

* **Structural semantics**  
  Meaning arises from term shape and position, not from environments or scopes.

* **Deterministic reduction**  
  Given the same input term and module tree, reduction always produces the same result.

This section introduces the formal syntax and the informal semantic model underlying Symp Core.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. Symp core language itself resembles a kind of S-expression. S-expressions consist of atoms or lists of other S-expressions where lists are surrounded by parenthesis. In Symp Core, the first list element to the left is called "head", and it determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
/////////////////////////////////////////////////////////
//                                                     //
//  Relaxed BNF rules for S-expr based Symp Core v0.x  //
//                                                     //
//  Notes:                                             //
//  - in this grammar, `...` reads as a terminal       //
//  - `*` marks zero or more occurrences               //
//                                                     //
/////////////////////////////////////////////////////////

<start> := (SYMP <alias>* <ident>*)
         | (FILE <ATOMIC>)

<alias> := (ALIAS <ATOMIC> <start>)

<ident> := (ID <ATOMIC> <term>)

<term> := (PARAMS ...)
        | (FUNCTION (PARAMS ...) (RESULT <ANY>))
```

The above grammar defines the syntax of Symp Core. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols (including `...`) are considered parts of the Symp Core grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. Informal Semantics

Symp Core is an S-expression–based language whose semantics are defined entirely in terms of **term reduction**. There is no notion of variables, environments, or mutable state. Computation proceeds by repeatedly rewriting expressions until no further reduction steps are possible.

At runtime, every expression is a **term**, and every term is either *reducible* or *irreducible* depending on its form and position.

#### Programs and Modules

A Symp Core program is organized as a **tree of modules**. Each module contains a set of named identifiers, where each identifier is associated with either:

* a parameter placeholder, or
* a function definition (a term with a result).

Modules may contain child modules, forming a hierarchical namespace. Identifiers always refer to definitions through an explicit relative module path; there is no global or dynamic scope. All name resolution is structural and deterministic.

Aliases introduce named subtrees into the module hierarchy. They do not alter evaluation semantics, but affect how identifiers are resolved.

#### Terms and evaluation

A term is either:

* an **atom** (identifier or literal), or
* a **list**, consisting of a head identifier and a list of argument terms.

Reduction always proceeds by attempting to evaluate the **head position** of a list. Identifiers and literals appearing outside of head position are *not evaluated automatically*.

Reduction continues until one of the following holds:

* the term is a literal,
* the term is an identifier,
* the term is a list whose head does not reduce to a reducible function.

#### Identifiers

An identifier consists of:

* a module path, and
* a local name.

Identifiers are **inert by default**. An identifier on its own does not evaluate to its definition. It only gains meaning when it appears in the *head position* of a list.

If an identifier cannot be resolved in the module tree, reduction yields an explicit `ERROR` term.

#### Reducible functions vs. irreducible parameters

Every identifier defined in a module belongs to exactly one of two semantic categories:

##### Reducible functions

A **function** is an identifier whose definition includes a result term. When such an identifier appears in the head position of a list, it is *reducible*.

Reducing a function application proceeds as follows:

1. The head of the list is reduced until it becomes an identifier.
2. If that identifier refers to a function definition:

   * the function’s result term is taken as a template,
   * the special identifier `Args` within the result is replaced by the argument list of the call,
   * all identifiers in the result are rewritten with module paths relative to the call site.

3. The resulting term replaces the original expression and reduction continues.

Function application is therefore **pure substitution**, not evaluation in an environment. Functions do not capture values, bind variables, or maintain state. They are best understood as *rewrite rules* over terms.

##### Irreducible parameters

A **parameters** is an identifier whose definition consists only of a parameter declaration and has no associated result term.

Parameters are **irreducible**:

* they do not trigger substitution,
* they cannot be applied as functions,
* and they never reduce further.

Parameters are identifiers with no rewrite rule; they exist solely to name irreducible structure. Its arguments may still be reduced, but the overall structure is preserved.

Semantically, parameters represent:

* constants,
* symbolic values,
* or opaque data constructors.

They provide a way to name values without introducing computation.

##### Consequences of the distinction

This separation between reducible functions and irreducible parameters has several important consequences:

* **No implicit evaluation**  
  Identifiers do not “evaluate to their value.” Only function application causes reduction.

* **No variables or bindings**  
  There is no variable substitution beyond the explicit handling of variadic `Args`.

* **Predictable evaluation**  
  Whether an expression reduces depends solely on whether its head resolves to a function.

* **Uniform data and code representation**  
  Functions, parameters, and data structures all share the same S-expression form.

#### Argument Handling

Arguments are positional and implicit. A function receives its arguments as a list accessible via the identifier `Args`. Arity checks, if required, are enforced explicitly by the function’s reduction logic or built-in rules.

There are no named parameters and no pattern matching at the core level.

#### Built-in reducible identifiers

Certain identifiers (such as `IsEq`, `IsAtom`, `IsEmpty`, `FAH` [*first-after-head*], `RAH` [*rest-after-head*], and `IAH` [*insert-after-head*]) are treated as built-in reducible functions with fixed reduction rules. These form the minimal operational core of the language and are not defined within the module system itself.

#### Normal forms

A term is in **normal form** if no reduction rule applies. This includes:

* literals,
* identifiers,
* lists whose head reduces to irreducible parameters,
* lists whose head is a built-in form that explicitly preserves the expression.

Normal forms are final results of computation.

#### Errors

Errors are represented as ordinary terms whose head is the identifier `ERROR`. Errors do not interrupt reduction via control flow; instead, they propagate as values. This makes error handling explicit and inspectable within the language itself.

#### Summary

Informally, Symp Core evaluates programs by:

1. Structurally resolving identifiers through a module tree,
2. Reducing expressions by repeatedly inspecting list heads,
3. Applying built-in rules or user-defined rewrite rules,
4. Producing a final term that can no longer be reduced.

The absence of implicit state, environments, or variable binding makes Symp Core a small but expressive foundation for symbolic computation and language experimentation.

## 3. Examples

This section presents a sequence of Symp Core examples, starting from simple atomic terms and gradually introducing lists, built-in operations, user-defined functions, branching, and recursion. Each example builds on concepts introduced earlier.

### 3.1. Atoms

The simplest Symp Core terms are **atoms**.

A literal atom evaluates to itself:

```
"hello"
```

Reduction result:

```
"hello"
```

Identifiers are also atoms. However, identifiers are *inert* on their own:

```
Foo
```

Reduction result:

```
Foo
```

No lookup or evaluation occurs unless an identifier appears in the *head position* of a list.

### 3.2. Lists and Head Reduction

A list represents a potential computation:

```
(Foo "bar")
```

Reduction proceeds by first reducing the head (`Foo`). If the head does not reduce to a valid identifier with defined semantics, reduction fails:

```
(ERROR "Head must reduce to identifier")
```

At this point, `Foo` has no meaning because it is not defined in any module.

### 3.3. Built-in Predicates

Some identifiers have predefined semantics when used as list heads. Examples in this subsection will assume `Foo` identifier is defined:

#### Equality: `IsEq`

The `IsEq` built-in tests atom equality of two reduced terms:

```
(IsEq "a" "a")
```

Reduction result:

```
True
```

```
(IsEq "a" (Foo "bar"))
```

Reduction result:

```
False
```

Equality is tested over fully reduced terms.

#### Atomic values: `IsAtom`

The `IsAtom` built-in tests whether its argument reduces to an atom:

```
(IsAtom "hello")
```

Reduction result:

```
True
```

Applied to a list:

```
(IsAtom (Foo "bar"))
```

Reduction result:

```
False
```

#### Lists: `IsEmpty`

The `IsEmpty` built-in tests whether a list has zero elements:

```
(IsEmpty (Foo))
```

Reduction result:

```
True
```

Applied to a list:

```
(IsEmpty (Foo "bar"))
```

Reduction result:

```
False
```

### 3.4. Lists as Data

Lists are also data structures and can be inspected using built-in operations.

Consider the list:

```
(Foo "a" "b" "c")
```

Using `FAH` (“first after head”):

```
(FAH (Foo "a" "b" "c"))
```

Reduction result:

```
"a"
```

Using `RAH` (“rest after head”):

```
(RAH (Foo "a" "b" "c"))
```

Reduction result:

```
(Foo "b" "c")
```

These operations treat lists purely structurally; the meaning of `Foo` is irrelevant here.

### 3.5. Defining Identifiers

Identifiers are introduced in modules using `ID` forms.

```
(SYMP
  (ID X
    (PARAMS ...)
    (RESULT "hello")))
```

The identifier `X` now names the literal `"hello"`.

Using `X` alone:

```
X
```

Reduction result:

```
X
```

Using `X` in head position:

```
(X)
```

Reduction result:

```
"hello"
```

### 3.6. Functions as Rewrite Rules

A function is defined by specifying a result term that may refer to `Args`.

```lisp
(SYMP
  (ID Echo
    (FUNCTION
      (PARAMS ...)
      (RESULT (FAH Args)))))
```

Calling `Echo`:

```lisp
(Echo "hello")
```

Reduction steps:

1. `Echo` resolves to a function.
2. Its result term is substituted.
3. `Args` is replaced by the `(Args ...)` where `...` stands for the argument list.

Final result:

```
"hello"
```

### 3.7. Multiple Arguments

Arguments are positional and accessed structurally.

```
(Echo "a" "b" "c")
```

Still reduces to:

```lisp
"a"
```

The function itself does not enforce arity; argument structure is handled explicitly by the function body.

### 3.8. Building New Lists: `IAH`

The `IAH` (“insert after head”) built-in constructs a new list by inserting an element after the head of an existing list.

```
(IAH "x" (Foo "a" "b"))
```

Reduction result:

```
(Foo "x" "a" "b")
```

This allows functions to construct and transform lists.

### 3.9. A Conditional Pattern

Although Symp Core has no built-in conditionals, conditional behavior can be expressed using parametric list heads. In the following example, we define head-relative branching behavior.

```
(SYMP
  (ID True
    (FUNCTION
      (PARAMS ...)
      (RESULT (FAH Args))))

  (ID False
    (FUNCTION
      (PARAMS ...)
      (RESULT (FAH (RAH Args)))))

  (ID Zero
      (PARAMS ...))

  (ID Succ
      (PARAMS ...))

  (ID Branch
    (FUNCTION
      (PARAMS ...)
      (RESULT ((IsEq (FAH Args) Zero) "a" "b")))))
```

Function `True` returns the first, while function `False` returns the second parameter. Reducing the predicate `IsEq` yields either `True` or `False` at the list head, deciding will it branch to the first or the second argument.

Calling `Branch`:

```
(Branch Zero)
```

represents `"a"`, and:

```
(Branch (Succ Zero))
```

represents `"b"`

### 3.10. Recursive Pattern Example

Although Symp Core has no explicit looping constructs, **recursive behavior emerges naturally from function application and substitution**. A function may refer to itself by name, and reduction will repeatedly expand that reference until a base case is reached.

In this example, we define a recursive function that computes the **length of a list**.

#### Representing Numbers

Symp Core has no built-in numeric literals or arithmetic. For this example, we represent numbers symbolically:

* `Zero` represents zero
* `(Succ n)` represents the successor of `n`

```
(SYMP
  (ID Zero
      (PARAMS ...))

  (ID Succ
      (PARAMS ...)))
```

For example:

```
(Succ Zero)
```

represents `1`, and:

```
(Succ (Succ Zero))
```

represents `2`.

#### Recursive Function: `Length`

We now define a recursive function `Length`:

```
(SYMP
  (ID True
    (FUNCTION
      (PARAMS ...)
      (RESULT (FAH Args))))

  (ID False
    (FUNCTION
      (PARAMS ...)
      (RESULT (FAH (RAH Args)))))

  (ID Zero
      (PARAMS ...))

  (ID Succ
      (PARAMS ...))
  
  (ID Length
    (FUNCTION
      (PARAMS ...)
      (RESULT
        ((IsEmpty Args)
          Zero
          (Succ (Length (RAH Args))))))))
```

Informally:

* If the list is empty, return `Zero`
* Otherwise, return `Succ (Length rest-of-list)`

Although this resembles a conditional, it is expressed entirely using function calls and emptiness testing.

#### Calling the Recursive Function

Consider the list:

```
(L "a" "b" "c")
```

Evaluating:

```
(Length (L "a" "b" "c"))
```

Reduction proceeds as follows (informally):

1. `Length` is applied to `(L "a" "b" "c")`
2. `(RAH Args)` becomes `(L "b" "c")`
3. Recursive call: `(Length (L "b" "c"))`
4. Recursive call: `(Length (L "c"))`
5. Recursive call: `(Length (L))`
6. Base case returns `Zero`
7. Successive `Succ` applications build the result

Final reduced form:

```
(Succ (Succ (Succ Zero)))
```

#### What This Demonstrates

This example illustrates several important properties of Symp Core:

* **Recursion is structural**: each recursive call operates on a smaller term.
* **No stack or environment exists**: recursion is just repeated substitution and reduction.
* **Termination is semantic, not enforced**: incorrect base cases lead to infinite reduction.
* **Functions are rewrite rules**: recursive calls expand the term until no rules apply.

A useful way to think about recursion in Symp Core is:

> *Each recursive call rewrites the expression into a slightly simpler one, until the expression can no longer be rewritten.*

There is no notion of “returning” from a call — only of reducing a term until it stabilizes.

### 3.11. Errors as Values

Errors are ordinary terms and propagate structurally.

```lisp
(FAH "hello")
```

Reduction result:

```lisp
(ERROR "'FAH' requires list as parameter")
```

Because errors are values, they can be passed around and inspected like any other term.

### 3.12. Summary

From these examples, we can observe that:

* Computation happens only through list reduction.
* Functions are applied via substitution, not variable binding.
* Arguments are accessed structurally using `Args`.
* Lists serve both as code and data.
* Control flow branching is possible by resolving list head.
* Function body may refer to itself, thus forming a recursion.
* Errors are explicit terms, not control-flow mechanisms.

These properties make Symp Core small, predictable, and well-suited for symbolic and structural computation.

## 4. Conclusion

Symp Core defines a small but expressive computational model centered on symbolic term reduction. By eliminating variables, environments, and implicit evaluation, it exposes the mechanics of computation in a direct and inspectable way.

The language demonstrates that:

* meaningful computation can be expressed purely through structural rewriting,
* functions can be modeled as substitution rules rather than executable procedures,
* control flow can emerge from data structure and head resolution,
* and errors can be treated as values rather than exceptional control paths.

While Symp Core is intentionally minimal, it provides a foundation upon which richer abstractions—such as pattern matching, typing disciplines, macro systems, or domain-specific languages—can be built.

As a core calculus, Symp Core prioritizes clarity and determinism over convenience. Its simplicity makes it well-suited for experimentation, formal reasoning, and as a substrate for exploring alternative language designs.

