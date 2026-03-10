---
layout: docs
---

# Symp Inc Specs

> **[about document]**  
> Specification of *Symp Inc*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
> Symp Inc is a minimal symbolic computation framework based on S-expressions and term rewriting. Its execution model is defined entirely by structural reduction rules, without variables, environments, mutable state, or implicit control flow. This document specifies the core syntax and semantics of Symp Inc, providing a formal grammar, an informal semantic model, and illustrative examples. The aim is to present Symp Inc as a small, deterministic foundation for symbolic processing, language experimentation, and reasoning about computation through explicit term transformation.


## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

Symp Inc is a deliberately minimal language designed to explore computation as **structural transformation of symbolic expressions**. Rather than relying on variables, environments, or mutable state, Symp Inc defines all behavior in terms of S-expression reduction driven by explicit rewrite rules.

The language is intended as a *core calculus*, not a full-featured programming language. Its design favors simplicity, predictability, and semantic transparency over convenience or performance. Every construct in Symp Inc has a small and well-defined meaning, and there are no hidden evaluation rules beyond those explicitly specified.

Symp Inc is suitable as:

* a foundation for higher-level symbolic or functional languages,
* a target for experimentation with alternative semantics,
* a teaching tool for reduction-based computation,
* or a compact system for symbolic processing and term manipulation.

This specification describes Symp Inc, focusing on its syntax, reduction semantics, and core operational behavior.

## 2. Theoretical Background

Symp Inc draws inspiration from several established ideas in programming language theory, including S-expressions, term rewriting systems, and functional reduction semantics. However, it intentionally avoids many features commonly found in functional languages, such as variable binding, closures, and implicit evaluation strategies.

Instead, Symp Inc adopts the following guiding principles:

* **Uniform representation**  
  Code and data share the same S-expression structure.

* **Explicit computation**  
  Only expressions whose head resolves to a reducible identifier can cause computation.

* **Structural semantics**  
  Meaning arises from term shape and position, not from environments or scopes.

* **Deterministic reduction**  
  Given the same input term and module tree, reduction always produces the same result.

This section introduces the formal syntax and the informal semantic model underlying Symp Inc.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. Symp core language itself resembles a kind of S-expression. S-expressions consist of atoms or lists of other S-expressions where lists are surrounded by parenthesis. In Symp Inc, the first list element to the left is called "head", and it determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
/////////////////////////////////////////////////////////
//                                                     //
//  Relaxed BNF rules for S-expr based Symp Inc v0.x  //
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

<term> := (CONSTANT (PARAMS ...))
        | (FUNCTION (PARAMS ...) (RESULT <ANY>))
```

The above grammar defines the syntax of Symp Inc. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols (including `...`) are considered parts of the Symp Inc grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. ### Informal Semantics

The **Symp Inc interpreter** evaluates expressions by repeatedly rewriting them until no further reduction is possible. Evaluation is deterministic and proceeds by recursively reducing subexpressions before attempting to apply a function.

#### Terms

A program manipulates **terms** (`IncTerm`). A term is either:

* **Literal** — an atomic value represented as a string.
* **List** — a structured expression consisting of:

  * a **head** identifier (the operation or constructor)
  * a sequence of **arguments** (`tail`)

Conceptually, a list corresponds to a function call:

```
(head arg1 arg2 ... argN)
```

#### Evaluation Strategy

Evaluation is performed by the `reduce` function, which repeatedly invokes `reduceHelper`.

The interpreter follows these general rules:

1. **Literals evaluate to themselves.**
2. **Lists evaluate by first evaluating all arguments.**
3. If any argument fails to reduce successfully, the list is returned in its partially reduced form.
4. Once all arguments are reduced, the interpreter attempts to apply the operation indicated by the list head.
5. If the head refers to a built-in operation, that operation is executed.
6. Otherwise, the interpreter attempts to resolve the head as a **user-defined function** in the module tree.
7. If the identifier cannot be resolved, evaluation produces an error term.

This strategy corresponds to a **strict evaluation model**: arguments are reduced before a function is applied.

#### Built-in Operations

The interpreter provides a small set of primitive operations.

##### Equality

`(IsEq x y)`

Returns `"true"` if both arguments are atomic literals with identical values.
If either argument is a list, the result is `"false"`.

##### Atom Test

`(IsAtom x)`

Returns `"true"` if `x` is a literal and `"false"` if `x` is a list.

##### Empty List Test

`(IsEmpty list)`

Returns `"true"` if the list has no tail elements.

##### Conditional

`(If cond thenExpr elseExpr)`

If `cond` is `"true"`, the result is `thenExpr`. Otherwise the result is `elseExpr`.

##### List Operations

These operations treat lists as a head element followed by a sequence of tail elements.

* `(FAH list)` — returns the first element after the head.
* `(RAH list)` — returns the list containing the remaining tail elements.
* `(IAH x list)` — inserts `x` as the first element after the head of `list` and returns the new list.

##### Type Inspection

`(TypeOf x)`

Returns a literal describing the type of `x`:

* `"LITERAL"` if `x` is a literal.
* Otherwise the identifier of the list head (including module path if present).

#### User-Defined Functions

If the head identifier of a list refers to a declaration in the module tree and the declaration is a **function**, evaluation proceeds by **substitution**.

The body of the function is copied and occurrences of the special literal `"ARGS"` are replaced with a list containing the arguments passed to the function.

After substitution, the resulting expression is evaluated again.

This mechanism provides a simple **macro-style function expansion** rather than environment-based variable binding.

#### Error Handling

Errors are represented as ordinary terms:

```
(ERROR "message")
```

They may arise from:

* unresolved identifiers
* incorrect number of arguments
* type mismatches
* invalid list operations (e.g., accessing elements of an empty list)

When an error occurs, evaluation stops for the current expression and the error term is returned.

#### Modules

Identifiers may reference functions within modules using a module path. During evaluation the interpreter resolves identifiers by traversing the **module tree** according to the path specified in the identifier.

If any module in the path does not exist, resolution fails and an error is produced.

## 3. Examples

This section presents a sequence of Symp Inc examples, starting from simple atomic terms and gradually introducing lists, built-in operations, user-defined functions, branching, and recursion. Each example builds on concepts introduced earlier.

### 3.1. Atoms

The simplest Symp Inc terms are **atoms**.

A literal atom evaluates to itself:

```
"hello"
```

Reduction result:

```
"hello"
```

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
true
```

```
(IsEq "a" (Foo "bar"))
```

Reduction result:

```
false
```

Equality is tested over fully reduced terms.

#### Atomic values: `IsAtom`

The `IsAtom` built-in tests whether its argument reduces to an atom:

```
(IsAtom "hello")
```

Reduction result:

```
true
```

Applied to a list:

```
(IsAtom (Foo "bar"))
```

Reduction result:

```
false
```

#### Lists: `IsEmpty`

The `IsEmpty` built-in tests whether a list has zero elements:

```
(IsEmpty (Foo))
```

Reduction result:

```
true
```

Applied to a list:

```
(IsEmpty (Foo "bar"))
```

Reduction result:

```
false
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

### 3.5. Functions as Rewrite Rules

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

### 3.6. Multiple Arguments

Arguments are positional and accessed structurally.

```
(Echo "a" "b" "c")
```

Still reduces to:

```lisp
"a"
```

The function itself does not enforce arity; argument structure is handled explicitly by the function body.

### 3.7. Building New Lists: `IAH`

The `IAH` (“insert after head”) built-in constructs a new list by inserting an element after the head of an existing list.

```
(IAH "x" (Foo "a" "b"))
```

Reduction result:

```
(Foo "x" "a" "b")
```

This allows functions to construct and transform lists.

### 3.8. A Conditional Pattern

Although Symp Inc has no built-in conditionals, conditional behavior can be expressed using parametric list heads. In the following example, we define head-relative branching behavior.

```
(SYMP
  (ID Branch
    (FUNCTION
      (PARAMS ...)
      (RESULT (If (IsEq (FAH ARGS) "zero") "a" "b")))))
```

Function `true` returns the first, while function `false` returns the second parameter. Reducing the predicate `IsEq` yields either `true` or `false` at the list head, deciding will it branch to the first or the second argument.

Calling `Branch`:

```
(Branch "zero")
```

represents `"a"`, and:

```
(Branch (Succ "zero"))
```

represents `"b"`

### 3.9. Recursive Pattern Example

Although Symp Inc has no explicit looping constructs, **recursive behavior emerges naturally from function application and substitution**. A function may refer to itself by name, and reduction will repeatedly expand that reference until a base case is reached.

In this example, we define a recursive function that computes the **length of a list**.

#### Representing Numbers

Symp Inc has no built-in numeric literals or arithmetic. For this example, we represent numbers symbolically:

* `"zero"` represents zero
* `(Succ n)` represents the successor of `n`

```
(SYMP
  (ID Succ
    (CONSTANT
      (PARAMS ...))))
```

For example:

```
(Succ "zero")
```

represents `1`, and:

```
(Succ (Succ "zero"))
```

represents `2`.

#### Recursive Function: `Length`

We now define a recursive function `Length`:

```
(SYMP
  (ID Succ
    (CONSTANT
      (PARAMS ...)))
  
  (ID Length
    (FUNCTION
      (PARAMS ...)
      (RESULT
        ((IsEmpty Args)
          "zero"
          (Succ (Length (RAH ARGS))))))))
```

Informally:

* If the list is empty, return `"zero"`
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
6. Base case returns `"zero"`
7. Successive `Succ` applications build the result

Final reduced form:

```
(Succ (Succ (Succ "zero")))
```

#### What This Demonstrates

This example illustrates several important properties of Symp Inc:

* **Recursion is structural**: each recursive call operates on a smaller term.
* **No stack or environment exists**: recursion is just repeated substitution and reduction.
* **Termination is semantic, not enforced**: incorrect base cases lead to infinite reduction.
* **Functions are rewrite rules**: recursive calls expand the term until no rules apply.

A useful way to think about recursion in Symp Inc is:

> *Each recursive call rewrites the expression into a slightly simpler one, until the expression can no longer be rewritten.*

There is no notion of “returning” from a call — only of reducing a term until it stabilizes.

### 3.10. Errors as Values

Errors are ordinary terms and propagate structurally.

```
(FAH "hello")
```

Reduction result:

```
(ERROR "'FAH' requires argument 0 to be list")
```

Because errors are values, they can be passed around and inspected like any other term.

## 4. Conclusion

Symp Inc defines a small but expressive computational model centered on symbolic term reduction. By eliminating variables, environments, and implicit evaluation, it exposes the mechanics of computation in a direct and inspectable way.

The language demonstrates that:

* meaningful computation can be expressed purely through structural rewriting,
* functions can be modeled as substitution rules rather than executable procedures,
* control flow can emerge from data structure and head resolution,
* and errors can be treated as values rather than exceptional control paths.

While Symp Inc is intentionally minimal, it provides a foundation upon which richer abstractions—such as pattern matching, typing disciplines, macro systems, or domain-specific languages—can be built.

As a core calculus, Symp Inc prioritizes clarity and determinism over convenience. Its simplicity makes it well-suited for experimentation, formal reasoning, and as a substrate for exploring alternative language designs.

