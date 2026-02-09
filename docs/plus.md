---
layout: docs
---

# Symp Plus Specs

> **[about document]**  
> Specification of *Symp Plus*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
> Symp Plus is a conservative extension of *Symp Core* that introduces a parameterized function syntax as a compile-time convenience. Rather than extending the runtime semantics, Symp Plus translates named parameters into explicit structural access over the implicit argument list `Args`, producing programs that are semantically equivalent to Symp Core definitions.
>
> This document specifies the syntax and compilation semantics of Symp Plus, describing how parameterized definitions are reduced to pure Symp Core terms prior to evaluation. The aim of Symp Plus is to improve clarity and expressiveness while preserving the minimal, deterministic, and structural execution model of the underlying calculus.

## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

Symp Plus is a *notation layer* built on top of **Symp Core**. Its purpose is to make certain classes of Symp Core programs easier to read and write, without changing how programs execute.

In Symp Core, functions receive their arguments implicitly through the structural list `Args`, and argument access is expressed explicitly using built-in operations such as `FAH` and `RAH`. While this design is simple and uniform, it can become verbose for functions that conceptually operate on multiple positional arguments.

Symp Plus addresses this by allowing functions to declare **named parameters**. These names improve readability and local reasoning, but they do not introduce variables, bindings, environments, or new evaluation rules.

Crucially:

* Symp Plus does **not** change the runtime model.
* Symp Plus does **not** add new reducible forms.
* Symp Plus programs are compiled into **plain Symp Core programs** before evaluation.

As a result, Symp Plus preserves all semantic guarantees of Symp Core while offering a more ergonomic surface syntax for defining functions.

This document describes Symp Plus independently, but it should be read in conjunction with the **Symp Core Specification**, which defines the execution model, reduction semantics, and built-in operations that Symp Plus relies upon.

## 2. Theoretical Background

Symp Plus inherits its theoretical foundation directly from Symp Core. It does not introduce new computational principles, but instead refines how certain structures are *written* and *expanded*.

From a semantic perspective, Symp Plus can be understood as a **source-to-source transformation system**: it rewrites parameterized function definitions into equivalent, parameter-free Symp Core definitions prior to evaluation.

This section outlines the syntactic additions and the compilation semantics that enable this transformation.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. Symp Plus language itself resembles a kind of S-expression. S-expressions consist of lists of atoms or other S-expressions where lists are surrounded by parenthesis. In Symp Plus, the first list element to the left determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
/////////////////////////////////////////////////////////
//                                                     //
//  Relaxed BNF rules for S-expr based Symp Plus v0.x  //
//                                                     //
//  Notes:                                             //
//  - `*` marks zero or more occurrences               //
//  - `+` marks one or more occurrences                //
//                                                     //
/////////////////////////////////////////////////////////

<start> := (SYMP <alias>* <ident>*)
         | (FILE <ATOMIC>)

<alias> := (ALIAS <ATOMIC> <start>)

<ident> := (ID <ATOMIC> <term>)

<term> := (PARAMS <ATOMIC>*)
        | (FUNCTION (PARAMS <ATOMIC>*) (RESULT <ANY>))

///////////////////////////////////////////////////////////
// within <ANY> S-expression, write projection <proj> as //
///////////////////////////////////////////////////////////

<proj> := (PROJ (CAST <param> <intersect>) <ATOMIC>)

<param> := <ATOMIC>
         | <proj>

<intersect> := (INTERSECT <ATOMIC>+)
             | <ATOMIC>
```

The above grammar defines the syntax of Symp Plus. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered parts of the Symp Plus grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. Informal Semantics

This section describes the intended meaning of **Symp Plus** programs at a conceptual level. It is informal by design and focuses on *how programs behave* rather than how they are parsed or evaluated internally.

#### Overview

**Symp Plus** is a symbolic, S-expression–based language designed to describe computations through **term rewriting**. Programs consist of **modules** that define parameters and functions. Expressions are evaluated by repeatedly reducing lists whose head position denotes an operation or function.

Symp Plus itself is not executed directly. Instead, it is **compiled into Symp Core**, a minimal interpreter where all semantics are realized through a small set of primitive operations and substitution rules.

#### Programs and Modules

A Symp Plus program consists of a top-level `(SYMP …)` form or a `(FILE …)` form.

* `(SYMP …)` introduces a module containing:

  * zero or more **aliases**, and
  * zero or more **identifiers** (definitions).

* `(FILE …)` represents an atomic, implementation-defined entry point and has no internal semantics at the language level.

Modules may be **nested**, forming a tree. Identifiers are resolved by walking this module tree using explicit module paths.

#### Aliases

An alias has the form:

```
(ALIAS <name> <start>)
```

An alias introduces a **named submodule**. The aliased `<start>` form is evaluated as if it were defined inline at that location in the module tree.

Aliases do not introduce new runtime behavior; they only affect **name resolution and modular structure**.

#### Identifiers and Declarations

Identifiers have the form:

```
(ID <name> <term>)
```

Each identifier binds a name to exactly one declaration. Declarations are either:

* a **parameter declaration**, or
* a **function declaration**.

Identifiers are inert by default. They only acquire meaning when used:

* as the **head of a list**, or
* as part of a parameter projection.

#### Parameters

A parameter declaration has the form:

```
(PARAMS p₁ p₂ … pₙ)
```

This declares an ordered list of parameter names for the surrounding identifier.

Parameters represent **positional arguments** to a function. They are not values themselves; instead, they describe how arguments are accessed within the function body.

Within function bodies, parameters may be:

* referenced directly, or
* accessed structurally via projections.

#### Functions

A function declaration has the form:

```
(FUNCTION
  (PARAMS p₁ p₂ … pₙ)
  (RESULT <term>)
)
```

A function defines a symbolic rewrite rule:

* When the function identifier appears in the **head position** of a list,
* the list’s tail is treated as the function’s arguments,
* and the function’s result term is instantiated by substituting arguments for parameters.

Functions are **pure**:

* they have no side effects,
* evaluation depends only on their arguments,
* and function application is implemented by substitution, not by environment mutation.

#### Terms and Evaluation Model

A **term** represents either:

* an **atom** (identifier, literal, or parameter), or
* a **list** of the form `(head tail₁ tail₂ …)`.

Evaluation proceeds by repeatedly reducing terms until no further reduction is possible.

Key principles:

* Atoms are irreducible.
* Lists are reducible only through their head.
* Reduction always attempts to reduce the head position first.

If the head reduces to a known function identifier, the function is applied. If the head reduces to a known parameters identifier, the parameters are considered as irreducible structure. Otherwise, an error may be triggered during runtime execution.

#### Parameters as Argument Structures

At runtime, function arguments are represented as a **list structure** bound to a special internal identifier (`Args` in the Core model).

Parameters are resolved structurally:

* the *first parameter* corresponds to the first argument,
* the second parameter to the second argument,
* and so on.

Parameter access is therefore defined in terms of list navigation, not by name lookup at runtime.

#### Projection and Structural Access

Symp Plus supports **parameter projection**, allowing access to parts of an argument structure.

A projection conceptually means:

> “From this argument list, select the argument named `<projection>` as declared in the parameter list.”

Projections are resolved **at compile time**, not at runtime. Each projection is translated into a sequence of primitive list operations (e.g. “rest after head”, “first after head”) that extract the desired argument position.

Projections may involve:

* a single parameter source, or
* an intersection of multiple parameter declarations, in which case only parameters common to all declarations are valid projection targets.

Invalid projections are compile-time errors.

#### Lists as Computation

Lists are the fundamental computation mechanism in Symp Plus.

* A list whose head is a **function identifier** represents a function application.
* A list whose head is a **primitive operator** represents a built-in operation.
* Any other list is treated as a symbolic structure and remains unreduced.

There is no distinction between “data” and “code”; both are represented uniformly as terms.

#### Errors

Errors are not exceptions in the host language sense. Instead, they are represented explicitly as symbolic error terms.

Errors may arise from:

* unknown identifiers,
* invalid projections,
* incorrect argument counts,
* or malformed list operations.

Once produced, an error term is inert and propagates unchanged.

#### Compilation Boundary

All high-level features of Symp Plus — including parameters, projections, and intersections — are **eliminated during compilation**.

The compiled Symp Core program:

* contains no parameters,
* contains no projections,
* and relies only on primitive list operations and substitution.

Thus, Symp Plus can be understood as a **macro language** over a small symbolic core.

## 3. Examples: Differences from Symp Core

The following examples illustrate what Symp Plus adds, and how it relates to Symp Core behavior.

### Example 1: Single Parameter Function

#### Symp Plus definition

```
(SYMP
  (ID Echo
    (FUNCTION
      (PARAMS x)
      (RESULT x))))
```

#### Symp Plus usage

```
(Echo "hello")
```

#### Compilation result (conceptual)

```
(SYMP
  (ID Echo
    (FUNCTION
      (PARAMS ...)
      (RESULT (FAH Args)))))
```

#### Final reduction result

```
"hello"
```

**Difference from Core:**
In Symp Core, `FAH Args` would need to be written explicitly. Symp Plus allows a named parameter `x` as syntactic sugar.

### Example 2: Multiple Parameters

#### Symp Plus definition

```
(SYMP
  (ID Pair
    (FUNCTION
      (PARAMS a b)
      (RESULT (L a b)))))
```

#### Usage

```
(Pair "x" "y")
```

#### Compilation result (conceptual)

```
(SYMP
  (ID Pair
    (Params ...)
    (RESULT
      (L (FAH Args) (FAH (RAH Args))))))
```

#### Final reduction result

```
(L "x" "y")
```

**Key point:**
Parameter order determines structure. Reordering parameters changes the generated access pattern.

### Example 3: Repeated Parameter Use

#### Symp Plus definition

```
(SYMP
  (ID L
    (PARAMS p q))
  
  (ID Dup
    (FUNCTION
      (PARAMS x)
      (RESULT (L x x)))))
```

#### Usage

```
(Dup "a")
```

#### Compilation result (conceptual)

```
(SYMP
  (ID L
    (PARAMS ...))

  (ID Dup
    (FUNCTION
      (PARAMS ...)
      (RESULT
        (L (FAH Args) (FAH Args))))))
```

#### Final reduction result

```
(L "a" "a")
```

**Observation:**
Each occurrence of `x` expands independently. There is no sharing or binding.

### Example 4: Projections and casting

#### Symp Plus definition

```
(SYMP
  (ID Projected
    (PARAMS x y z))

  (ID GetProjected
    (FUNCTION
      (PARAMS p)
      (RESULT
        (PROJ (CAST p Projected) y)))))
```

#### Usage

```
(GetProjected (Projected "a" "b" "c"))
```

#### Compilation result (conceptual)

```
(SYMP
  (ID Projected
    (PARAMS ...))

  (ID GetProjected
    (FUNCTION
      (PARAMS ...)
      (RESULT
        (FAH (RAH (FAH Args)))))))
```

#### Final reduction result

```
"b"
```

**key point**
We can access projections, but only after required casting is applied. Choice of the right casting is our responsibility.

### Example 5: Undefined Parameter Error

#### Symp Plus definition

```
(SYMP
  (ID Bad
    (FUNCTION
      (PARAMS x)
      (RESULT y))))
```

During compilation, `y` cannot be resolved to a parameter.

#### Compilation result

```
(ERROR "Unknown parameter: 'y'")
```

**Difference from Core:**
This error is detected *before runtime* and does not involve reduction.

### Summary

Symp Plus introduces **named parameters and projections after casting as a compile-time abstraction**, not a semantic extension. All parameterized functions are transformed into equivalent Symp Core definitions using explicit structural access over `Args`.

As a result:

* Symp Plus programs compile into valid Symp Core programs
* The Symp Core interpreter requires no modification
* All guarantees of determinism, transparency, and structural semantics are preserved

Symp Plus should be understood as a *notation layer* that improves clarity and ergonomics while maintaining the minimalism of the underlying calculus.

## 4. Conclusion

Symp Plus demonstrates that meaningful improvements in expressiveness do not require expanding a language’s runtime semantics. By introducing named parameters and projections as a **compile-time abstraction**, Symp Plus improves readability and maintainability while remaining fully compatible with the Symp Core execution model.

Because all parameterized and projected definitions are translated into explicit structural operations over `Args`, Symp Plus:

* preserves the determinism and transparency of Symp Core,
* avoids introducing variables, environments, or bindings,
* keeps all computation reducible to term rewriting,
* and requires no changes to the core interpreter.

Symp Plus should therefore be understood not as a new language, but as a *conservative extension*—a notation layer that compiles away entirely. Its design illustrates how higher-level conveniences can be layered atop a minimal calculus without obscuring or weakening its underlying semantics.

As with Symp Core, the simplicity of Symp Plus makes it a suitable foundation for experimentation, formal reasoning, and the incremental exploration of alternative language designs.

