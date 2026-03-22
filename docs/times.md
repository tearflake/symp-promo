---
layout: docs
---

# Symp Times Specs

> **[about document]**  
> Specification of *Symp Times*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
**[abstract]**  
> Symp Times is a statically-checked symbolic programming language designed to express computations over structured data using a uniform S-expression syntax. It introduces a structural interface system based on products and unions, enabling safe composition, projection, and pattern matching without relying on nominal typing. Programs are compiled through intermediate representations into a minimal symbolic core, ensuring both expressive power and a simple execution model. This document defines the syntax, semantics, and design principles of Symp Times, and illustrates its use through a series of progressively complex examples.

## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

Symp Times is a symbolic programming language that emphasizes structural correctness, explicit data access, and predictable execution. It is designed as the highest-level layer in a multi-stage compilation framework, where programs are progressively transformed into simpler intermediate representations until they reach a minimal symbolic execution core.

Unlike traditional programming languages that rely on nominal typing or implicit structure, Symp Times adopts a fully structural approach. Values are described in terms of the capabilities they provide, and functions specify their requirements through explicit interfaces. This allows programs to be composed based on what data *can do*, rather than what it *is called*.

The language combines three key ideas:

- **Symbolic representation** using S-expressions
- **Structural typing** via products and unions
- **Safe data access** through projection and pattern matching

Symp Times is not executed directly. Instead, it is compiled into lower-level representations:

- **Symp Plus**, which makes structure explicit and resolves projections
- **Symp Inc**, which evaluates programs through symbolic reduction

This separation allows Symp Times to remain expressive and safe, while delegating execution to a simple and uniform runtime model.

The goal of this document is to define the syntax and semantics of Symp Times, and to provide insight into its design through illustrative examples.

## 2. Theoretical Background

This section describes the formal structure and meaning of Symp Times programs. It is divided into two parts:

- **Formal Syntax**, which defines the valid structure of programs using a grammar
- **Informal Semantics**, which explains how programs are interpreted and validated

The formal syntax specifies how expressions are constructed, while the informal semantics describe how those expressions behave and how their correctness is determined.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols considered to be correctly structured statements or expressions in that language. Symp Times language itself resembles a kind of S-expression. S-expressions consist of atoms or lists of other S-expressions where lists are surrounded by parenthesis. In Symp Times, the first list element to the left is called "head", and it determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
//////////////////////////////////////////////////////////
//                                                      //
//  Relaxed BNF rules for S-expr based Symp Times v0.x  //
//                                                      //
//  Notes:                                              //
//  - `*` marks zero or more occurrences                //
//  - `+` marks one or more occurrences                 //
//                                                      //
//////////////////////////////////////////////////////////

<start> := (SYMP <alias>* <ident>*)
         | (FILE <ATOMIC>)

<alias> := (ALIAS <ATOMIC> <start>)

<ident> := (ID <ATOMIC> <decl>)

<decl> := (EXPECTS <function> <tvar-entails>)
        | (EXPECTS <constant> <tvar-product>)

<function> := (FUNCTION (PARAMS <ATOMIC>*) (RESULT <ANY>))

<constant> := (CONSTANT (PARAMS <ATOMIC>*))

<tvar-entails> := (FORALL <ATOMIC>+ <entails>)
               | <entails>

<tvar-product> := (FORALL <ATOMIC>+ <product>)
               | <product>

<entails> := (ENTAILS <product> <union>)

<product> := (PRODUCT <union>*)

<union> := (UNION <primary>+)
         | <primary>

<primary> := (TYPE <ATOMIC> <primary>*)

/////////////////////////
// occurs within <ANY> //
/////////////////////////

<match> := (MATCH <ATOMIC> <case>+)

<case> := (CASE <primary> <ANY>)
```

In addition to the above syntax grammar, we add the following embeddable syntax for accessing projections:
 
```
<projection> := (PROJ <BASE> <ATOMIC>+)
```

The above grammar defines the syntax of Symp Times. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols (including `...`) are considered parts of the Symp Times grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. Informal Semantics

**Symp Times** is a statically-checked, expression-based language designed to describe symbolic computations over structured data.

Programs in Symp Times are compiled into **Symp Plus**, and ultimately into **Symp Inc**, where execution is performed via term reduction. The role of Symp Times is therefore:

> to provide **typed structure, safe data access, and pattern matching**, while remaining compilable to a minimal symbolic core.

#### Values and Data

Symp Times operates over three fundamental kinds of values:

##### Literals

Atomic values represented as strings.

```
"true", "false", "hello"
```

They are treated as indivisible values.

##### Structured Values (Constructors)

Constants define **product types**, i.e. structured values with named fields.

Example intuition:

```plaintext
(Person name age)
```

* `Person` is the constructor
* `name`, `age` are fields
* The value is ordered internally, but accessed by name

##### Unions

A **union type** represents a value that may be one of several constructors:

```
(UNION Person | Company)
```

Unions are used primarily in:

* function interfaces
* pattern matching

#### Functions

Functions map input values to output values.

Each function is annotated with an **interface**:

```
(input types) -> output type
```

More precisely:

* inputs form a **product**
* output is a **union**

##### Evaluation Model

Function bodies are expressions evaluated by:

* computing subexpressions
* applying functions
* resolving projections
* evaluating matches

Evaluation is **deterministic** and **strict** (arguments are evaluated before use), except where control flow dictates otherwise (e.g. match branches).

#### Type System

The type system ensures that programs are well-formed before execution.

##### Products and Unions

* **Product**: ordered collection of input types
* **Union**: set of possible result types

##### Subtyping

A type `A` is a subtype of `B` if:

* every possible value of `A` is also a valid value of `B`

For unions:

```
A ⊆ B  if all elements of A are in B
```

##### Type Variables (Generics)

Functions may be parameterized by type variables.

These are instantiated during type checking and must remain consistent across usage.

##### Type Inference

The compiler infers:

* types of expressions
* shapes of intermediate values

Errors occur if:

* arguments do not match expected types
* projections are invalid
* match expressions are not exhaustive

#### Projection (Field Access)

Projection extracts a field from a structured value:

```
(PROJ x name)
```

##### Semantics

* The compiler determines the **position** of the field in the constructor.
* Access is only valid if:

  * the field exists
  * its position is consistent across all possible types of `x`

##### Projection on Unions

If `x` has a union type:

```
(UNION A B)
```

then:

* the field must exist in **all variants**
* the field must appear at the **same position**

Otherwise, the projection is rejected at compile time.

#### Pattern Matching

Pattern matching allows branching based on the shape of a value:

```
(MATCH x
  (CASE A expr1)
  (CASE B expr2)
  (CASE ELSE expr3)
```

##### Semantics

* The value of `x` is inspected
* Its constructor determines which branch is taken
* Only the selected branch is evaluated

##### Exhaustiveness

A match must cover all possible cases:

* every variant in the union must be handled
* or an `ELSE` branch must be provided

Errors:

* missing case
* duplicate case
* invalid constructor

##### Type of Match

The result type is the **union of all branch result types**.

#### Lists and Function Calls

A function call:

```plaintext
(f a b c)
```

is evaluated by:

1. evaluating `a`, `b`, `c`
2. applying `f` to those values

The result is determined by the function body.

#### Constants vs Functions

##### Constants

* represent structured values
* define field names and types

##### Functions

* compute values
* must satisfy their declared interface

#### Errors

Errors are detected at compile time whenever possible:

* unknown identifiers
* invalid projections
* type mismatches
* non-exhaustive matches

At runtime, execution assumes well-typed input and proceeds without additional checks.

#### Compilation Perspective

Symp Times expressions are not executed directly.

Instead:

1. Types are checked and inferred
2. Pattern matching is translated into conditionals
3. Field access is converted into positional access
4. The program is lowered into Symp Plus

Ultimately:

> All structured semantics are reduced to **symbolic list operations** in Symp Inc.

#### Design Intuition

Symp Times separates concerns:

* **structure and safety** live here
* **representation and execution** are delegated to lower layers

This allows:

* expressive high-level programming
* simple and uniform runtime behavior

## 3. Examples

The following examples illustrate Symp Times’ structural validation model in practice. Each example introduces a small fragment of the language and demonstrates how interface obligations are declared, propagated, and enforced.

The examples are intentionally minimal. They are not intended to demonstrate full programs, but rather to isolate individual structural concepts such as capability requirements, composition, unions, casting, and parametric interfaces. Together, they form a progressive introduction to the Symp Times model.

### Example 1 — The Smallest Possible Contract

This example introduces a function that requires a capability. Its purpose is to show how interfaces act as structural obligations and how projections represent capabilities that a value may or may not provide. The core idea is a function that expects its argument to expose a `name` projection. Through this, the example illustrates the notion of atomic capability sets and demonstrates how basic structural validation works.

```
(SYMP
    (ID HasName
        (EXPECTS
            (PARAMS name)
            (PRODUCT String)))
    
    (ID PrintName
        (EXPECTS
            (FUNCTION
                (PARAMS x)
                (RESULT (PROJ x name)))
            
            (ENTAILS
                (PRODUCT HasName)
                String))))
```

Here, `PrintName` is defined in terms of what it needs rather than what it receives. It only cares that `x` satisfies the `HasName` interface, meaning that it structurally provides a `name` projection.

```
(PrintName (HasName "John")) -> "John"
```

### Example 2 — Structural Failure

This example focuses on what happens when a structural obligation is not met. It exists to make structural errors concrete and to show how invalid workflows are rejected early. The scenario demonstrates what occurs when `PrintName` is called with a value that does not satisfy the `HasName` interface, highlighting how missing projections are detected and how errors are reported.

```
(SYMP
    (ID HasValue
        (EXPECTS
            (PARAMS value)
            (PRODUCT String)))
    
    ...)
```

When we call the function with:

```
(PrintName (HasValue "12")) -> Error in 'PrintName' function: expecting 'HasName' interface
```

the system rejects the call because the value provided does not expose the required structure. The error makes it explicit that `PrintName` expects a value conforming to the `HasName` interface.

### Example 3 — Composition of Parameters

This example demonstrates how independent capabilities can be combined. It introduces a function that requires capabilities drawn from multiple structural sources, showing that capability aggregation works in a cartesian, compositional way. The emphasis is on how structure can be conjunctive across parameters and how capabilities do not need to originate from a single source.

```
(SYMP
    (ID User
        (EXPECTS
            (PARAMS name email)
            (PRODUCT String String)))
    
    (ID GetEmail
        (EXPECTS
            (FUNCTION
                (PARAMS x)
                (RESULT (PROJ x email)))
            
            (ENTAILS
                (PRODUCT User)
                String))))
```

The example reinforces the idea that structure is compositional and that required capabilities can be satisfied independently.

```
(GetEmail (User "Jane" "jane@mymail.com")) -> "jane@mymail.com"
```

### Example 4 — Union and Cast

This example explores how a function can safely accept multiple structural shapes and how a value moves from possibility to commitment. It introduces structural alternatives using `UNION`, and shows the need for explicit narrowing. The function demonstrated can work with either emails or phone numbers, but it must explicitly take responsibility for narrowing before using a specific capability.

```
(SYMP
    (ID Email
        (EXPECTS
            (PARAMS address title text)
            (PRODUCT String String String)))

    (ID SMS
        (EXPECTS
            (PARAMS number text)
            (PRODUCT String String)))

    (ID User
        (EXPECTS
            (PARAMS name prefers)
            (PRODUCT String (UNION Email SMS))))

    (ID Notify
        (EXPECTS
            (FUNCTION
                (PARAMS x)
                (RESULT
                    (MATCH x
                        (CASE Email
                            (PROJ x address))
                        
                        (CASE SMS
                            (PROJ x number))
            
            (ENTAILS
                (PRODUCT User)
                String)))
```

This example makes it clear that Symp refuses silent narrowing. The programmer must explicitly choose a structure by explicitly using the `Cast` function, and accept responsibility for that choice. Details about the `Cast` function are explained in the section [Example 5].

```
(Notify (User "Jill" (Email "email" "jill@mymail.com"))) -> "jill@mymail.com"
(Notify (User "Jack" (SMS   "sms"   "+155512345"     ))) -> "+155512345"
```

### Example 5 — Parametric Interfaces

This final example introduces interfaces that abstract over structure itself. Its purpose is to demonstrate reusability through parametric structural constraints. The function shown works over any interface, illustrating structural polymorphism without relying on traditional types or concrete values.

```
(ID Identity
    (EXPECTS
        (FUNCTION (PARAMS x) (RESULT x))
        (FORALL X (ENTAILS (PRODUCT X) X))))
```

This form of polymorphism is purely structural: it operates over interfaces, not types, and does not depend on the nature of the values involved. Calling the function by: `(Identity "x")` yields `"x"`.

### Summary

Taken together, these examples demonstrate the core philosophy of Symp Times: programs are defined in terms of structural capabilities, and correctness is enforced through explicit interfaces and compile-time validation. Rather than relying on implicit assumptions about data, Symp Times requires all structural requirements to be stated and verified, resulting in programs that are both predictable and robust.

## 4. Conclusion

## 4. Conclusion

Symp Times presents a structured approach to symbolic computation, combining a simple syntactic foundation with a powerful system of structural interfaces. By separating concerns between high-level expression, intermediate representation, and low-level execution, the framework achieves both clarity and flexibility.

The language enforces correctness through compile-time validation, ensuring that all structural requirements are satisfied before execution. Features such as projection, unions, and pattern matching are designed to be both expressive and safe, while remaining compatible with a minimal execution model.

Through its compilation pipeline, Symp Times demonstrates how complex language features can be systematically reduced to simple symbolic operations. This makes it suitable not only as a programming language, but also as a foundation for experimentation in language design, type systems, and symbolic processing.

