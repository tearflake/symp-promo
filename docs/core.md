---
layout: docs
---

# Symp Core Specs

> **[about document]**  
> Specification of *Symp core*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
> Symp is a symbolic processing framework built around S-expressions, explicit module structures, and substitution-based evaluation. Unlike traditional programming languages that rely on environments, closures, and implicit scoping rules, Symp models computation as a sequence of structural rewrites over symbolic terms. Its core design emphasizes transparency, compositionality, and explicit semantics, making it well-suited for symbolic computation, metaprogramming, domain-specific languages, and program transformation tasks. This document specifies the Symp core language, describing its syntax, informal semantics, and representative usage patterns.


## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

Symp is a symbolic computation framework designed around a small but expressive core. Its primary goal is to provide a minimal, well-defined foundation for building higher-level computational systems that manipulate structured expressions rather than executing imperative instructions.

At its core, Symp treats programs as trees of symbolic terms. There is no strict separation between code and data: functions, expressions, and values are all represented using the same underlying S-expression structure. Computation proceeds by *rewriting* these structures according to explicitly declared rules, rather than by executing statements in a mutable environment.

Several design principles guide Symp:

* **Explicit structure over implicit behavior**  
  Name resolution, parameter access, and module boundaries are all explicit and structural.

* **Substitution instead of environments**  
  Function application is defined as syntactic substitution rather than environment-based evaluation.

* **Minimal core semantics**  
  The core language is intentionally small, enabling predictable behavior and extensibility.

* **Symbolic-first computation**  
  Symp is intended to manipulate expressions symbolically, even when modeling evaluative or functional behavior.

This specification focuses exclusively on the *Symp core*. Extensions such as imperative constructs, functional enrichments, rewriting systems, or device-oriented components are intentionally out of scope and may be defined in separate documents.

## 2. Theoretical Background

Symp is influenced by several well-established ideas in programming language theory, symbolic computation, and formal systems. This section outlines the conceptual foundations that motivate its design.

First, Symp adopts the **S-expression** as its fundamental syntactic representation. Originating in early symbolic computation systems, S-expressions provide a uniform and recursive structure that naturally represents trees, lists, and nested expressions. This uniformity allows Symp programs to be easily analyzed, transformed, and generated.

Second, Symp draws from **term rewriting systems**. Rather than viewing computation as a sequence of state transitions, Symp models evaluation as the repeated transformation of symbolic terms until a stable form is reached. Function definitions act as rewrite rules, and evaluation corresponds to their systematic application.

Third, Symp deliberately avoids **environment-based semantics** common in functional languages. There are no closures, lexical environments, or dynamic variable bindings. Instead, Symp relies on explicit module paths and substitution. This approach eliminates many sources of ambiguity —such as variable capture or hidden dependencies— at the cost of requiring more explicit structure.

Finally, Symp incorporates ideas from **module systems** found in statically structured languages. Names are resolved through a hierarchical module tree rather than through lexical scope. This design choice emphasizes clarity, compositionality, and predictability in large symbolic systems.

Together, these foundations position Symp as a framework for *symbolic structure manipulation* rather than a conventional general-purpose programming language. The following sections formalize this perspective by describing its syntax, semantics, and usage patterns.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. Symp core language itself resembles a kind of S-expression. S-expressions consist of lists of atoms or other S-expressions where lists are surrounded by parenthesis. In Symp core, the first list element to the left determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
<start> := (SYMP <alias>+ <ident>+)
         | (FILE <ATOMIC>)

<alias> := (ALIAS <ATOMIC> <start>)

<ident> := (IDENT <ATOMIC> <term>)

<term> := (PARAMS <ATOMIC>+)
        | (PARAMS ...)
        | (RESULT <ANY>)
        | (FUNCTION (PARAMS <ATOMIC>+) (RESULT <ANY>))
```

The above grammar defines the syntax of Symp core. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered parts of the Symp core grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

We will be using the same syntax definition nomenclature throughout the accompanying specification documents of imperative, functional, and rewriting extensions, and device components.

### 2.2. Informal Semantics

This section describes how Symp programs are *interpreted* and how expressions *behave*, without relying on formal evaluation rules. The goal is to explain how Symp “thinks” and how users should reason about programs written in it.

#### Programs as Symbolic Structures

Symp is a **symbolic, expression-oriented framework** based on S-expressions.
Every program, definition, and computation in Symp is represented as a tree of terms. There is no distinction between “code” and “data”: expressions are manipulated as structured values.

Evaluation in Symp proceeds by **rewriting expressions** until they reach a stable form. Rather than executing instructions step by step, Symp repeatedly replaces identifiers and function applications with their corresponding definitions.

#### Modules and Namespaces

All definitions in Symp live inside a **module tree**.

A module:

* Maps names to *definitions*
* May contain child modules
* Is addressed explicitly via module paths

Identifiers are resolved **structurally**, not lexically. Each identifier carries its own module path, and name lookup follows that path through the module tree. There is no implicit global scope and no dynamic environment.

This design ensures:

* Fully explicit name resolution
* No hidden captures or shadowing
* Deterministic interpretation of identifiers

#### Definitions and Their Roles

A name in a module may correspond to one of three conceptual roles:

1. **Parameters**  
   Declares that the name is callable and specifies its parameter structure.

2. **Result**  
   Associates the name with a value or expression. Such identifiers behave like constants.

3. **Function**  
   Combines parameters and a result expression, forming a transformation rule.

These roles determine how an identifier behaves during evaluation:

* A *Result* is automatically replaced by its value when encountered.
* A *Function* is expanded only when applied to arguments.
* A *Parameters-only* definition acts as a symbolic operator and does not reduce on its own.

#### Expressions and Evaluation

Symp expressions are either **atoms** or **lists**.

* **Atoms** evaluate to themselves unless they are function parameters or identifiers bound to results.
* **Lists** represent applications or structured expressions.

Evaluation always proceeds by first reducing the **head** of a list. The meaning of a list depends on what its head evaluates to.

If the head resolves to:

* A special identifier (e.g. `Eq`), special evaluation rules apply.
* A function definition, the function is expanded.
* A parameter declaration, the list is preserved as a symbolic structure with evaluated arguments.

Evaluation continues until no further rewriting steps are possible or an error is encountered.

#### Function Application as Substitution

Functions in Symp are not applied via environments or closures. Instead, they operate by **substitution**.

When a function is applied:

1. Its result expression is copied.
2. Formal parameters are replaced with the corresponding argument expressions.
3. Module paths inside the function body are adjusted relative to the call site.
4. The resulting expression is evaluated further.

This makes function application:

* Purely structural
* Referentially transparent
* Free of hidden state

Functions therefore behave more like **rewrite rules** than traditional runtime procedures.

#### Parameters and Projections

Parameters in Symp are not just placeholders; they may include **projections**, which extract parts of argument expressions.

Two kinds of projections exist:

1. **Named projections**  
   If a parameter corresponds to a fixed-arity definition, its fields can be accessed by name.

2. **Structural projections**  
   For variadic arity definitions, `head` and `tail` allow positional access.

Projections are resolved during substitution and allow functions to deconstruct arguments without requiring explicit pattern-matching syntax.

#### Variadic Expressions and Expansion

Symp supports variadic parameter lists and a special expansion form. Variadic lists are declared by writing a single `...` symbol at the place of the arguments.
Later, before evaluation of variadic parameters, we may optionally inline special anonymous variadic lists identified by `...` at the list head place.

During evaluation:

* Variadic arguments are expanded inline.
* Each expanded element is evaluated individually.
* Expansion occurs before the surrounding list is finalized.

This allows Symp to express flexible, macro-like constructs while keeping the core semantics simple.

#### Errors and Invalid Expressions

Errors in Symp are represented explicitly as expressions. Common error conditions include:

* Unknown identifiers
* Invalid module paths
* Incorrect number of parameters
* Invalid projections
* Empty lists

Because errors are values, they propagate naturally through evaluation unless explicitly handled.

#### Evaluation Strategy Summary

In informal terms, Symp evaluation can be summarized as:

> “Repeatedly replace names with what they mean, expand functions by substitution, and reduce lists by interpreting their head.”

This strategy favors:

* Clarity over performance
* Structure over state
* Explicit semantics over implicit behavior

## 3. Examples

This section introduces Symp by example. Each example builds on the previous ones and highlights a specific aspect of the framework.

### Example 1: A Constant Definition

The simplest Symp module defines a constant result.

```
(SYMP
    (IDENT Pi
        (RESULT 3.14159)))
```

Using `Pi` in an expression causes it to reduce automatically:

```
Pi
// → 3.14159
```

In Symp, identifiers bound to results behave like constants: they are replaced by their associated expressions whenever they are encountered.

### Example 2: Symbolic Operators

An identifier may declare parameters without defining a result.

```
(SYMP
    (IDENT Add
        (PARAMS x y)))
```

Applying `Add` does **not** compute anything:

```
(Add 1 2)
// → (Add 1 2)
```

Here, `Add` is treated as a symbolic operator. Its arguments are evaluated, but the expression itself is preserved. This is useful for building symbolic expressions, DSLs, or intermediate representations.

### Example 3: Defining a Function

A function combines parameters and a result expression.

```
(SYMP
    (IDENT Inc
        (FUNCTION
            (PARAMS x)
            (RESULT (Add x 1)))
    
    (IDENT Add
        (PARAMS x y))))
```

Now applying `Inc` rewrites the expression:

```
(Inc 4)
// → (Add 4 1)
```

Note that `Add` is still symbolic here. Symp has expanded the function body by substituting `x` with `4`, but no numeric evaluation occurs.

### Example 4: Nested Reduction

Functions may reference other functions and constants.

```
(SYMP
    (IDENT Two
        (RESULT 2))

    (IDENT Double
        (FUNCTION
            (PARAMS x)
            (RESULT (Add x x))))

    (IDENT Quad
        (FUNCTION
            (PARAMS x)
            (RESULT (Double (Double x)))))
    
    (IDENT Add
        (PARAMS x y)))
```

Evaluating:

```
(Quad Two)
```

proceeds conceptually as:

```
(Double (Double Two))
→ (Double (Add 2 2))
→ (Add (Add 2 2) (Add 2 2))
```

Symp repeatedly expands functions and replaces constants until no further rewriting applies.

### Example 5: Fixed Parameters and Projections

Parameters can be projected by name when their structure is known.

```
(SYMP
    (IDENT Pair
        (PARAMS a b))

    (IDENT First
        (FUNCTION
            (PARAMS p)
            (RESULT p.a)))

    (IDENT Second
        (FUNCTION
            (PARAMS p)
            (RESULT p.b))))
```

Using these definitions:

```
(First (Pair 10 20))
// → 10

(Second (Pair 10 20))
// → 20
```

Here, `p.a` and `p.b` refer to the first and second arguments of the `Pair` expression. Projection is structural and resolved during substitution.

### Example 6: Variadic Parameters and Expansion

Symp supports variadic parameter lists.

```
(SYMP
    (IDENT List
        (PARAMS ...)))
```

`List` accepts any number of parameters:

```
(List 1 2 3)
// → (List 1 2 3)
```

The special expansion form `...` flattens lists during evaluation:

```
(List 1 2 (... 3 4))
// → (List 1 2 3 4)
```

Expansion happens before the final expression is returned, allowing flexible macro-like constructs.

### Example 7: Structural Projections (`head` and `tail`)

For variadic expressions, Symp provides positional projections.

```
(SYMP
    (IDENT Head
        (FUNCTION
            (PARAMS x)
            (RESULT x.head)))

    (IDENT Tail
        (FUNCTION
            (PARAMS x)
            (RESULT x.tail)))
    
    (IDENT List
        (PARAMS ...)))
```

Using them:

```
(Head (List 1 2 3))
// → 1

(Tail (List 1 2 3))
// → (List 2 3)
```

These projections allow recursive and structural manipulation without explicit pattern-matching syntax.

### Example 8: Modules and Aliases

Modules may be nested using aliases.

```
(SYMP
    (ALIAS Math
        (SYMP
            (IDENT Zero
                (RESULT 0))

            (IDENT Inc
                (FUNCTION
                    (PARAMS x)
                    (RESULT (Add x 1))))

            (IDENT Add
                (PARAMS x y)))))
```

Identifiers can refer to module-qualified names:

```
Math/Zero
// → 0

(Math/Inc 5)
// → (Math/Add 5 1)
```

Module paths are explicit and preserved during evaluation.

### Example 9: Equality as a Built-in Form

Symp includes special identifiers with custom evaluation rules.

```
(Eq 1 1)
// → TRUE

(Eq (Inc 1) 2)
// → False
```

`Eq` evaluates both arguments and compares their resulting expressions structurally.

### Example 10: Putting It All Together

A symbolic length function for lists:

```
(SYMP
    (IDENT True
        (FUNCTION
            (PARAMS x y)
            (RESULT x)))

    (IDENT False
        (FUNCTION
            (PARAMS x y)
            (RESULT y)))

    (IDENT Len
        (FUNCTION
            (PARAMS xs)
            (RESULT
                ((Eq xs nil)
                    0
                    (Inc (Len xs.tail)))))))
```

While still symbolic, this example demonstrates:

* Recursive function expansion
* Structural projections
* Built-in comparison
* Parametric list head
* Explicit termination conditions

### Summary

Through these examples, we have seen that:

* Symp programs are rewritten, not executed
* Functions expand by substitution
* Parameters support structural access
* Modules control name resolution
* Symbolic and evaluative constructs coexist naturally

Symp encourages thinking in terms of **structure and transformation**, making it especially well-suited for symbolic computation, metaprogramming, and domain-specific languages.

## 4. Conclusion

This document has presented the specification of the Symp core language, including its formal syntax, informal semantics, and a sequence of illustrative examples. Symp defines computation as structural transformation over symbolic expressions, emphasizing explicitness, compositionality, and simplicity of the core model.

By avoiding implicit environments and runtime state, Symp provides a deterministic and transparent evaluation strategy based on substitution and rewriting. Its module system enables explicit name resolution and controlled composition, while projections and variadic constructs allow expressive manipulation of structured data.

The Symp core is intentionally minimal. It is not intended to compete with general-purpose programming languages, but rather to serve as a foundational layer upon which richer symbolic, functional, imperative, or domain-specific systems can be built. Future specifications may extend this core with additional evaluation rules, typing disciplines, or computational effects, while preserving the principles outlined here.

In summary, Symp offers a compact yet expressive framework for reasoning about and transforming symbolic structures, making it a suitable foundation for advanced metaprogramming and symbolic computation systems.

