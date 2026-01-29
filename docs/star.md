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

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. Symp star language itself resembles a kind of S-expression. S-expressions consist of atoms or lists of other S-expressions where lists are surrounded by parenthesis. In Symp Star, the first list element to the left is called "head", and it determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

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

<entails> := (ENTAILS <interface> <interface>)
           | <interface>

<interface> := (INTERFACE <union>*)
             | <union>

<union> := (UNION <primary>*)
         | <primary>

<primary> := <ATOMIC>
```

In addition to the above syntax grammar, we add the following embeddable syntax for accessing projections:
 
```
<projection> := (Get <BASE> <ATOMIC>+)
```

The above grammar defines the syntax of Symp Star. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols (including `...`) are considered parts of the Symp Star grammar.

Atomic expressions may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

### 2.2. Informal Semantics

The informal semantics of Symp Star describe how syntactically valid expressions are *understood* by the validator. Unlike traditional semantics, these rules do not assign meanings or values to expressions. Instead, they describe how interface obligations are propagated, combined, and checked.

Symp Star interprets programs as symbolic structures annotated with interface claims. Validation proceeds by ensuring that every projection access, function application, and composition step is structurally justified by the declared interfaces. When an obligation cannot be satisfied, validation fails with an explicit structural error.

The following parts describe these semantics in detail, beginning with basic orientation and progressing through composition, control, parametricity, and philosophical limits.

#### Part I — Orientation

**What Symp Star Is (and Is Not)**

Symp Star is a structural interface validation system whose role is to check whether symbolic programs respect the interface obligations they declare. It operates purely at the level of structure: it does not inspect values, reason about meaning, or attempt to interpret behavior. Its purpose is to determine whether symbolic programs, plans, or workflows are structurally admissible. By design, Symp Star is a static, value-agnostic, execution-independent validator whose analysis is decidable. It is intentionally not a traditional type system, a theorem prover, or a semantic analyzer. Rather than attempting to understand what a program does, Symp Star only verifies whether the program is structurally permitted to do it.

**Why Structural Validation Matters**

Many failures in real systems arise not from incorrect values, but from incorrect assumptions about structure. These failures include accessing projections that may not exist, composing steps whose interfaces do not align, or calling functions whose outputs are incompatible with subsequent inputs. Symp Star addresses these problems by making structural assumptions explicit and mechanically checkable. By validating structure statically, it detects such errors before execution ever occurs.

**The Cost of Implicit Assumptions**

When assumptions about structure remain implicit, they silently propagate through programs and workflows. Over time, this leads to runtime errors, fragile systems, and excessive defensive programming. Symp Star forces these assumptions to be stated explicitly as interface obligations. In doing so, it replaces hidden expectations with clear, enforceable structural contracts.

**Why We Avoid Semantics on Purpose**

Semantic reasoning is inherently value-dependent, context-sensitive, and often undecidable. Because of this, systems that rely on semantic analysis tend to be unpredictable and difficult to compose reliably. Symp Star deliberately avoids semantic reasoning in order to remain predictable, decidable, and compositional. This avoidance is not a limitation, but a design choice that preserves mechanical reliability.

#### Part II — Core Concepts

**Symbols, Identifiers, and Projections**

In Symp Star, symbols name entities, while projections represent named structural capabilities that may be accessed from those entities using built-in `Get` interface. A projection denotes permission to access a particular capability, not the value that capability yields. Symp Star never inspects values themselves; it only checks whether a projection is structurally permitted.

**Interfaces as Structural Obligations**

An interface in Symp Star is defined as a set of projections that an entity must support. When an expression claims an interface, it is obligated to support every projection contained within that interface. Interfaces therefore describe what must be possible structurally, rather than what will happen at runtime or what values will be produced.

**Capabilities and Capability Sets**

Capabilities are atomic projections that represent individual structural permissions. Capability sets describe the total collection of projections that may be safely accessed from an entity. These sets may be atomic, compositional, or expressed as unions of alternatives, and they may be reduced through structural operations. Together, they form the basis of Symp Star’s structural reasoning.

**Functions as Structural Transformers**

Functions in Symp Star are treated as transformations over interfaces rather than over values. Each function declares the interfaces it requires from its parameters and the interface it guarantees for its result. Through this declaration, functions describe how structure flows and transforms across program boundaries, independent of execution.

#### Part III — Composition

**Composing Steps Safely**

Structural composition in Symp Star is permitted only when the output interface of one step satisfies the input requirements of the next. If a mismatch occurs, composition is rejected outright. This ensures that structurally invalid workflows are prevented before execution.

**Product: Combining Independent Capabilities**

The `PRODUCT` construct represents the simultaneous availability of multiple interfaces. All components of a product must be satisfied for the product itself to be valid. This allows independent capabilities to be combined into a single structural obligation.

**Union: Representing Alternatives**

The `UNION` construct represents structural uncertainty by allowing an entity to satisfy one of several alternative interfaces. When an interface is expressed as a union, only one of its alternatives is guaranteed to be present at any given time. This explicitly models uncertainty in structure.

**Why Alternatives Are Dangerous Without Commitment**

Because a union represents multiple possible structures, projections cannot be safely accessed without first committing to a specific alternative. Attempting to access a projection directly from a union is therefore rejected. In its default mode, Symp Star requires explicit narrowing before such access is allowed.

#### Part IV — Control and Responsibility

**Structural Branching**

Branching in Symp Star originates from applying functions like the `Eq` predicate at list heads, directing execution into a result branch. Each branch introduces a different possible interface, resulting in a union of potential outcomes. Symp Star tracks this divergence structurally rather than semantically.

**Explicit Narrowing with Cast**

The `Cast` construct explicitly commits a union to a specific interface. By performing a cast, the programmer declares responsibility for choosing a particular structural alternative. This commitment allows projections to be accessed safely thereafter.

**Why Cast Is Not an Interface Escape Hatch**

Casting does not invent new capabilities or weaken existing guarantees. Instead, it asserts intent and shifts responsibility to the programmer. The structural obligations remain intact; only the responsibility for correctness changes hands.

**Optimism vs Pessimism in Structural Analysis**

Optimistic analysis assumes admissibility unless proven impossible, while pessimistic analysis requires guarantees at every step. Exact analysis occupies a balance between these extremes by requiring explicit interface casting to resolve uncertainty. Symp Star defaults to exact analysis in order to enforce clarity and responsibility without unnecessary restriction.

#### Part V — Parametric Structure

**Parametric Interfaces**

Parametric interfaces abstract over interface variables rather than concrete structures. This allows interface definitions to be reused across many contexts without committing to specific capabilities. Parametricity enables generality while preserving structural rigor.

**Structural Reuse Without Types**

Through parametricity, Symp Star enables reuse without relying on a traditional type system. Structural constraints can be expressed generically, without introducing semantic commitments about values or types. This keeps abstraction purely structural.

**Generic Workflows**

Workflows in Symp Star may be expressed generically over interfaces rather than concrete capability sets. This allows plans and transformations to be reused across different structural contexts. Such workflows remain independent of the specific capabilities involved.

**Interface-Level Abstraction**

All abstraction in Symp Star occurs at the interface level rather than at the value level. This ensures that abstraction remains structural, mechanical, and decidable. Values and semantics remain outside the system’s scope.

#### Part VI — Validation in Practice

**Symp Star as a Pre-Execution Guard**

Symp Star operates as a guard that validates plans before execution begins. By checking structural admissibility early, it prevents a wide class of errors from occurring at runtime. This shifts failure detection to the earliest possible stage.

**Validating Symbolic Plans**

Symbolic plans generated by humans or machines are treated as symbolic expressions. Symp Star checks these expressions mechanically to ensure that their structural obligations are satisfied. No execution or interpretation is required.

**Diagnosing Structural Errors**

When validation fails, Symp Star reports errors in terms of violated interface obligations. These errors identify missing or unsafe projections directly. This makes structural problems explicit and diagnosable.

**What “Correct” Means in Symp Star**

In Symp Star, correctness has a narrow and precise meaning. A program is correct if it is structurally admissible, and incorrect otherwise. No claims are made beyond this structural criterion.

#### Part VII — Positioning

**Symp Star vs Type Systems**

Traditional type systems reason about values and their behaviors. Symp Star, by contrast, reasons only about structure and permitted projections. It validates structure without committing to any particular typing discipline.

**Symp Star vs Theorem Provers**

Theorem provers aim to establish truths through logical reasoning. Symp Star does not attempt to prove anything; it merely checks whether declared structural obligations are satisfied. Its role is validation, not proof.

**Symp Star vs Symbolic Programming**

Symp Star is not a symbolic programming system itself, but a complement to such systems. It validates symbolic programs without executing them. In this way, it serves as structural infrastructure rather than a programming paradigm.

**Symp Star as Infrastructure, Not Authority**

Symp Star enforces discipline but does not impose meaning. It provides structural guarantees without asserting intent, correctness of results, or semantic interpretation. Authority remains with the program author.

#### Part VIII — Limits and Philosophy

**Why We Stay Decidable**

Decidability ensures that validation always terminates and produces predictable results. This predictability enables trust, speed, and composability. Remaining decidable is therefore a foundational design goal.

**Why We Avoid Negation and Existentials**

Negation and existential reasoning introduce undecidability and semantic dependence. Including them would undermine Symp Star’s mechanical guarantees. For this reason, they are deliberately excluded.

**What Symp Star Refuses to Know**

Symp Star explicitly refuses to reason about values, truth, semantics, or intent. These concerns lie outside its mandate. By refusing to know them, Symp Star preserves clarity and mechanical reliability.

**What Responsibility Means in Formal Systems**

In Symp Star, responsibility means explicit commitment. Ambiguity is not resolved implicitly by the system but must be resolved by the author. Symp Star enforces this discipline by requiring responsibility to be stated structurally and explicitly.

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
                (RESULT (Get x name)))
            
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
            (PRAMS value)
            (PRODUCT Number)))
    
    ...)
```

When we call the function with:

```
(PrintName (HasValue 12)) -> Error in 'PrintName' function: expecting 'HasName' interface
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
                (RESULT (Get x email)))
            
            (ENTAILS
                (PRODUCT User)
                String))))
```

The example reinforces the idea that structure is compositional and that required capabilities can be satisfied independently.

```
(GetMail (User "Jane" "jane@mymail.com")) -> "jane@mymail.com"
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
                    ((Eq (Get x prefers tag) "email")
                        (Get (Cast (Get x prefers) (Email "abc" "abc")) email )
                        (Get (Cast (Get x prefers) (SMS   "abc" "abc")) number)))))
            
            (ENTAILS
                (PRODUCT User)
                (UNION String Number))))
```

This example makes it clear that Symp refuses silent narrowing. The programmer must explicitly choose a structure by explicitly using the `Cast` function, and accept responsibility for that choice. Details about the `Cast` function are explained in the next section.

```
(Notify (User "Jill" (Email "email" "jill@mymail.com"))) -> "jill@mymail.com"
(Notify (User "Jack" (SMS   "sms"   "+1555123456"    ))) -> "+155512345"
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

The following parametric function defines very important casting function:

```
(ID Cast
    (EXPECTS
        (FUNCTION
            (PARAMS x i)
            (RESULT x))
        
        (PARAMETRIC X I
            (ENTAILS
                (PRODUCT X I)
                I))))
```

Here, `Cast` is shown not as a built-in primitive but as an ordinary function. It takes a value and a witness expression of an interface, using that witness to justify the cast. Casting allows us to repurpose a value declaring it complies to the new interface. Projections of the new interface `I` have to be a subset of the old interface `X`, otherwise an interface error is reported. 

## 4. Conclusion

Symp Star provides a disciplined framework for reasoning about structure in symbolic systems. By focusing exclusively on interface obligations and projections, it avoids the complexity and undecidability associated with value-level or semantic analysis. This makes it predictable, compositional, and suitable for use as infrastructure in larger systems.

Rather than replacing type systems, theorem provers, or execution engines, Symp Star complements them. It occupies a narrow but realistic niche: validating that symbolic programs are structurally admissible before they are executed or interpreted. In doing so, it shifts responsibility from implicit assumptions to explicit declarations.

Ultimately, Symp Star enforces a simple principle: if a program claims it can do something structurally, it must prove that claim upfront. Everything else—meaning, intent, correctness of results—remains the responsibility of the author and the surrounding system.

