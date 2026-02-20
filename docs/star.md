---
layout: docs
---

# Symp Star Specs

> **[about document]**  
> Specification of *Symp Star*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
> Symp Star is a symbolic processing framework for validating the structural admissibility of programs, plans, and workflows. Rather than reasoning about values or execution, Symp Star operates solely on declared interface obligations and projection capabilities, ensuring that symbolic expressions are composed in structurally valid ways. The system is intentionally value-agnostic, execution-independent, and decidable by design. By making structural assumptions explicit and mechanically checkable, Symp Star prevents a broad class of errors arising from invalid composition, missing capabilities, and unsafe structural assumptions. This specification defines the formal syntax of Symp Star, explains its informal structural semantics, and illustrates its use through progressively refined examples.

## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

Symp Star is a symbolic processing framework designed to validate the *structural admissibility* of programs, plans, and workflows. Rather than reasoning about values or execution, Symp Star focuses exclusively on whether symbolic expressions respect the interface obligations they declare. This makes it suitable as a pre-execution validation layer for systems that generate, transform, or compose symbolic programs.

The primary motivation behind Symp Star is the observation that many failures in complex systems arise from incorrect structural assumptions rather than incorrect values. These failures are often detected late—at runtime—or handled defensively through ad-hoc checks. Symp Star addresses this problem by making structure explicit and mechanically verifiable.

This specification describes Symp Star as a formal system. It defines the syntax of the language, explains its informal semantics, and illustrates its behavior through worked examples. The document is intended for advanced programmers and system designers who are interested in structural validation, symbolic computation, and formally disciplined composition.

## 2. Theoretical Background

Symp Star is grounded in a small set of principles: structure is distinct from semantics, validation should be decidable, and responsibility should be explicit. These principles inform both the formal syntax of the language and its informal interpretation.

This section provides the conceptual foundation necessary to understand how Symp Star works. First, it introduces the formal syntax used to represent symbolic programs. Then, it explains the informal semantics that govern how these structures are interpreted and validated.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols considered to be correctly structured statements or expressions in that language. Symp star language itself resembles a kind of S-expression. S-expressions consist of atoms or lists of other S-expressions where lists are surrounded by parenthesis. In Symp Star, the first list element to the left is called "head", and it determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
/////////////////////////////////////////////////////////
//                                                     //
//  Relaxed BNF rules for S-expr based Symp Star v0.x  //
//                                                     //
/////////////////////////////////////////////////////////

<start> := (SYMP <alias>* <ident>*)
         | (FILE <ATOMIC>)

<alias> := (ALIAS <ATOMIC> <start>)

<ident> := (ID <ATOMIC> <term>)

<term> := (EXPECTS <params> <expects>)
        | (EXPECTS <function> <expects>)

<params> := (PARAMS <ATOMIC>*)

<function> := (FUNCTION (PARAMS <ATOMIC>*) (RESULT <ANY>))

<expects> := <parametric>

<parametric> := (PARAMETRIC <ATOMIC>+ <entails>)
              | <entails>

<entails> := (ENTAILS <product> <product>)
           | <product>

<product> := (PRODUCT <union>*)
           | <union>

<union> := (UNION <primary>*)
         | <primary>

<primary> := <ATOMIC>

///////////////////////////////////////////////////////////
//  within <ANY> S-expression, write:                    //
//  - projection: <proj>                                 //
//  - casting: <cast>                                    //
///////////////////////////////////////////////////////////

<proj> := (PROJ <base> <ATOMIC>)

<base> := <ATOMIC>
        | <cast>

<cast> := (CAST <param> <intersect>)

<param> := <ATOMIC>
         | <proj>

<intersect> := (INTERSECT <ATOMIC>+)
             | <ATOMIC>
```

In addition to the above syntax grammar, we add the following embeddable syntax for accessing projections:
 
```
<projection> := (PROJ <BASE> <ATOMIC>+)
```

The above grammar defines the syntax of Symp Star. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols (including `...`) are considered parts of the Symp Star grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. Informal Semantics

The **Symp Star** framework extends *Symp Plus* with explicit interface annotations and static validation. Star programs are not executed directly; instead, they are **checked, erased, and projected** into plain Symp Plus programs. The Star layer exists purely to enforce correctness and does not affect runtime behavior.

Semantically, a Star program is processed in **three distinct passes**.

#### Interface Extraction

In the first pass, the compiler traverses the Star module tree and extracts **interfaces** from every declaration.

Each module produces an *interface environment* mapping declaration names to their declared interfaces. These environments mirror the module hierarchy of the program and are used for name resolution and validation in later passes.

No expressions are analyzed in this pass. The goal is solely to make all interfaces globally available before any semantic checking occurs.

#### Interface Validation and Term Checking

The second pass enforces the semantic correctness of the Star program. It validates both the *shape* of declared interfaces and the *admissibility* of all terms.

##### Top-level interface shape

Every declaration must have a top-level interface of the correct form:

* Parameter declarations must describe a **product**.
* Function declarations must describe an **entailment** (`input ⟹ output`), optionally wrapped in a parametric interface.

Any deviation is a static error.

##### Term interpretation

Every Star term is interpreted as producing an **interface value**.

* **Literals** evaluate to a literal interface.
* **Parameters** evaluate to a parameter interface.
* **Identifiers** evaluate to an identifier interface referring to a named declaration.
* **PROJ expressions** project a labeled field from a product interface.
* **Applications** evaluate by applying a function or product interface to argument interfaces.

Term checking is recursive and compositional: the interface of a term is determined entirely by the interfaces of its subterms.

##### Application semantics

When a list expression is encountered, the head must resolve to a known identifier. Its interface determines how the application is interpreted.

###### Product application

If the head resolves to a product interface:

* The number of arguments must match the number of product fields.
* Each argument interface must satisfy the corresponding field interface.
* The result of the application is the same product interface.

This models structural construction or validation of a product value.

###### Entailment application

If the head resolves to an entailment interface:

* The number of arguments must match the input arity.
* Each argument must satisfy the corresponding input interface.
* Parametric variables are instantiated during this process.
* The output interface is computed by substituting parameters and resolved variables into the entailment’s output.

The result is a fully inferred interface representing the application’s value.

##### Parametric interfaces and variable resolution

Parametric interfaces introduce universally quantified variables. These variables are instantiated during application by matching actual arguments against required interfaces.

Variable resolution follows unification-like rules:

* A variable may be bound once to a concrete interface.
* Subsequent uses must be consistent with the existing binding.
* Cyclic bindings are rejected.

All substitutions are explicit and statically enforced.

##### Union interfaces

Union interfaces represent alternatives.

* A value must satisfy **exactly one** union alternative.
* If no alternative matches, the application is invalid.
* If more than one alternative matches, the result is ambiguous and rejected.
* Union values cannot be implicitly projected; explicit casting is required.

This ensures deterministic interface inference.

##### Projections

A projection extracts a labeled field from a product interface.

* Projections are only allowed on product interfaces.
* Attempting to project from a union or non-product interface is an error.
* Nested projections are evaluated step by step.

#### Erasure and Projection to Symp Plus

The final pass erases all Star-specific constructs:

* Interfaces are removed.
* Only parameters, identifiers, and pure expressions remain.

The resulting program is a valid **Symp Plus** module tree with identical runtime behavior to the original Star program.

The Star layer thus functions as a **static correctness discipline** with no operational effect.

#### Summary

Conceptually, Symp Star adds a **logical interface layer** on top of Symp Plus:

* Interfaces describe the admissible shapes of values and applications.
* Programs are checked via entailment, unification, and projection.
* All interface information is erased before execution.

A Star program is valid if and only if its interfaces can be consistently satisfied across all terms.

## 3. Examples

The following examples illustrate Symp Star’s structural validation model in practice. Each example introduces a small fragment of the language and demonstrates how interface obligations are declared, propagated, and enforced.

The examples are intentionally minimal. They are not intended to demonstrate full programs, but rather to isolate individual structural concepts such as capability requirements, composition, unions, casting, and parametric interfaces. Together, they form a progressive introduction to the Symp Star model.

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
            (PARAMS tag email)
            (PRODUCT String String)))

    (ID SMS
        (EXPECTS
            (PARAMS tag number)
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
                    ((Eq (PROJ (PROJ x prefers) tag) "email")
                        (PROJ (CAST (PROJ x prefers) Email) email )
                        (PROJ (CAST (PROJ x prefers) SMS) number)))))
            
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
        (PARAMETRIC X (ENTAILS (PRODUCT X) X))))
```

This form of polymorphism is purely structural: it operates over interfaces, not types, and does not depend on the nature of the values involved. Calling the function by: `(Identity "x")` yields `"x"`.

## 4. Conclusion

Symp Star provides a disciplined framework for reasoning about structure in symbolic systems. By focusing exclusively on interface obligations and projections, it avoids the complexity and undecidability associated with value-level or semantic analysis. This makes it predictable, compositional, and suitable for use as infrastructure in larger systems.

Rather than replacing type systems, theorem provers, or execution engines, Symp Star complements them. It occupies a narrow but realistic niche: validating that symbolic programs are structurally admissible before they are executed or interpreted. In doing so, it shifts responsibility from implicit assumptions to explicit declarations.

Ultimately, Symp Star enforces a simple principle: if a program claims it can do something structurally, it must prove that claim upfront. Everything else—meaning, intent, correctness of results—remains the responsibility of the author and the surrounding system.

