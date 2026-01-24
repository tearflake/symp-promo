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
<start> := (SYMP <alias>+ <ident>+)
         | (FILE <ATOMIC>)

<alias> := (ALIAS <ATOMIC> <start>)

<ident> := (ID <ATOMIC> <term>)

<term> := (PARAMS ...)
        | (FUNCTION (PARAMS <ATOMIC>*) (RESULT <ANY>))
```

The above grammar defines the syntax of Symp Plus. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered parts of the Symp Plus grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. Informal Semantics

Symp Plus extends **Symp Core** with a *parameterized function syntax* while preserving the same underlying execution model. All runtime semantics remain those of Symp Core; Symp Plus introduces **no new reduction rules**, **no environments**, and **no additional runtime constructs**.

Instead, Symp Plus operates by **compiling parameterized definitions into pure Symp Core terms** prior to evaluation.

#### Parameters as Compile-Time Structure

In Symp Plus, a function may declare a finite list of **named parameters**:

```
(FUNCTION (PARAMS x y z) (RESULT <term>))
```

These parameter names:

* are **purely syntactic**,
* exist **only during compilation**,
* and are **not present at runtime**.

They do not correspond to variables, bindings, or values. Instead, each parameter name is compiled into a *structural access pattern* over the implicit argument list `Args`.

At runtime, Symp Plus programs are indistinguishable from Symp Core programs.

#### Compilation to Symp Core

Before evaluation, a Symp Plus program undergoes a **parameter resolution pass** that rewrites all function bodies:

* Each occurrence of a parameter name is replaced with a term that extracts the corresponding positional argument from `Args`
* Parameter access is expressed exclusively using the built-in structural operators:

  * `FAH` (first after head)
  * `RAH` (rest after head)

For a function with parameters:

```
(PARAMS p₀ p₁ p₂ ...)
```

the parameter `pᵢ` is compiled to the structural form:

```
(FAH (RAH (RAH ... Args)))
```

where `RAH` is applied `i` times.

This transformation is purely syntactic and does not depend on runtime values.

#### Consequences of Compilation

Because parameters are compiled away:

* **There is no parameter scope at runtime**
* **No binding or rebinding occurs**
* **Parameters are positional, not symbolic**
* **Repeated parameter use duplicates structure**
* **Errors from incorrect arity emerge structurally**

All argument access is explicit, inspectable, and reducible using the existing Symp Core semantics.

#### Runtime Semantics

After compilation:

* All functions are standard Symp Core functions
* All parameter references have become `FAH` / `RAH` expressions
* Evaluation proceeds exactly as defined in the Symp Core specification

Symp Plus therefore introduces **no semantic extensions**, only **syntactic convenience**.

## 3. Examples: Differences from Symp Core

The following examples illustrate what Symp Plus adds, and how it relates to Symp Core behavior.

### Example 1: Single Parameter Function

#### Symp Plus definition

```
(ID Echo
  (FUNCTION
    (PARAMS x)
    (RESULT x)))
```

#### Symp Plus usage

```
(Echo "hello")
```

#### Compilation result (conceptual)

```
(ID Echo
  (FUNCTION
    (PARAMS ...)
    (RESULT (FAH Args))))
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
(ID Pair
  (FUNCTION
    (PARAMS a b)
    (RESULT (L a b))))
```

#### Usage

```
(Pair "x" "y")
```

#### Compilation result (conceptual)

```
(L
  (FAH Args)
  (FAH (RAH Args)))
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
(ID Dup
  (FUNCTION
    (PARAMS x)
    (RESULT (L x x))))
```

#### Usage

```
(Dup "a")
```

#### Compilation result (conceptual)

```
(L
  (FAH Args)
  (FAH Args))
```

#### Final reduction result

```
(L "a" "a")
```

**Observation:**
Each occurrence of `x` expands independently. There is no sharing or binding.

### Example 4: Undefined Parameter Error

#### Symp Plus definition

```
(ID Bad
  (FUNCTION
    (PARAMS x)
    (RESULT y)))
```

During compilation, `y` cannot be resolved to a parameter.

#### Compilation result

```
(ERROR "Undefined parameter")
```

**Difference from Core:**
This error is detected *before runtime* and does not involve reduction.

### Example 5: Symp Plus vs Symp Core Equivalence

The following two definitions are semantically equivalent.

#### Symp Plus

```
(ID Second
  (FUNCTION
    (PARAMS x y)
    (RESULT y)))
```

#### Symp Core (manual)

```
(ID Second
  (FUNCTION
    (PARAMS)
    (RESULT (FAH (RAH Args)))))
```

Both reduce identically when applied.

### Summary

Symp Plus introduces **named parameters as a compile-time abstraction**, not a semantic extension. All parameterized functions are transformed into equivalent Symp Core definitions using explicit structural access over `Args`.

As a result:

* Symp Plus programs compile into valid Symp Core programs
* The Symp Core interpreter requires no modification
* All guarantees of determinism, transparency, and structural semantics are preserved

Symp Plus should be understood as a *notation layer* that improves clarity and ergonomics while maintaining the minimalism of the underlying calculus.

## 4. Conclusion

Symp Plus demonstrates that meaningful improvements in expressiveness do not require expanding a language’s runtime semantics. By introducing named parameters as a **compile-time abstraction**, Symp Plus improves readability and maintainability while remaining fully compatible with the Symp Core execution model.

Because all parameterized definitions are translated into explicit structural operations over `Args`, Symp Plus:

* preserves the determinism and transparency of Symp Core,
* avoids introducing variables, environments, or bindings,
* keeps all computation reducible to term rewriting,
* and requires no changes to the core interpreter.

Symp Plus should therefore be understood not as a new language, but as a *conservative extension*—a notation layer that compiles away entirely. Its design illustrates how higher-level conveniences can be layered atop a minimal calculus without obscuring or weakening its underlying semantics.

As with Symp Core, the simplicity of Symp Plus makes it a suitable foundation for experimentation, formal reasoning, and the incremental exploration of alternative language designs.

