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
> Symp core is a minimal symbolic computation engine built upon a strict S-expression grammar and a compact set of primitive operations. Its purpose is to provide a referentially transparent environment for expressing deterministic computations using function definitions and pure symbolic evaluation. This document formally defines the syntax, informal semantics, and fundamental computational model of Symp core. It also outlines the relationship between core functions, imported modules, and the built-in primitives that together form a complete and usable foundation for higher-level Symp components.


## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Theoretical Background](#2-theoretical-background)
    - [2.1. Formal Syntax](#21-formal-syntax)
    - [2.2. Informal Semantics](#22-informal-semantics)  
- [3. Examples](#3-examples)  
- [4. Conclusion](#4-conclusion)  

## 1. Introduction

The Symp core is designed as a small, self-contained language for symbolic computation. It provides only the essential constructs needed to define functions, structure data, and evaluate expressions. The intention is not to recreate a full programming language, but to offer a foundational substrate upon which more elaborate paradigms can be layered.

The design emphasizes simplicity, referential transparency, and structural clarity. Each program is a collection of named constants and functions, each with fixed arity and a single result expression. Evaluation proceeds through nested S-expressions, with no state, mutation, or side effects. The resulting system is predictable, deterministic, and amenable to formal reasoning, while still being expressive enough to define a wide range of symbolic transformations.

## 2. Theoretical Background

The Symp core rests on a formal syntactic structure that defines what constitutes a valid program, and an informal semantic model describing how such programs are evaluated. Although small, the system is computationally complete when combined with the nine built-in primitive functions. This section introduces the grammar used in Symp core, its conventions, and the semantics guiding the evaluation process.

### 2.1. Formal Syntax

In computer science, the syntax of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. Symp core language itself resembles a kind of S-expression. S-expressions consist of lists of atoms or other S-expressions where lists are surrounded by parenthesis. In Symp core, the first list element to the left determines a type of a list. There are a few predefined list types depicted by the following relaxed kind of Backus-Naur form syntax rules:

```
<start> := (SYMP <elem>+)

<elem> := (FUNCTION <ATOMIC> (PARAMS <ATOMIC>+) (RESULT <ANY>))
        | (CONSTANT <ATOMIC> <ANY>)
        | (ALIAS <ATOMIC> <rec>)

<rec> := <start>
       | (FILE <ATOMIC>)
```

The above grammar defines the syntax of Symp core. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...*` for zero or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered parts of the Symp core grammar.

Atoms may be enclosed between a pair of `'` characters if we want to include special characters used in the grammar. Strings are enclosed between a pair of `"` characters. Multiline atoms and strings are enclosed between an odd number of `'` or `"` characters.
 
In addition to the exposed grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//` and span to the end of line. Multiline comments begin with `/*` and end with `*/`.

We will be using the same syntax definition nomenclature throughout the accompanying specification documents of imperative, functional, and rewriting extensions, and device components.

### 2.2 Informal Semantics

Semantics of Symp core is inspired by the functional programming paradigm and Lisp primitives. It is meant to be a very minimalist, but still practical computing environment. Program written in Symp core is a set of constants or referentially transparent functions with a fixed arity. Each constant or function returns an arbitrary S-expression evaluated using other functions or primitives. There are nine built-in primitive functions which make Symp core a Turing complete computing environment:

- `CALL`: two or more parameters. calls a function named by the first parameter, possibly with arguments from the second parameter onward.
- `IF`: three parameters. If the first parameter is `true`, it returns the second one. If not, it returns the third one. 
- `EQ`: two parameters. If both parameters are atoms and are equal, it returns `true`; otherwise `false`.
- `HEAD`: one parameter as a list. Returns the first list element.
- `TAIL`: one parameter as a list. Returns the list without the first element.
- `CONS`: two parameters. The second parameter is a list. Returns a new list, placing the first parameter at the beginning of the second parameter list
- `ISATOM`: one parameter. Returns `true` if the parameter is an atom, otherwise `false.
- `TOATOM`: one parameter. Converts to an atom a right recursive list consisting of atom's characters, ending with an empty list.
- `TOLIST`: one parameter. Converts an atom to a right recursive list consisting of atom's characters, ending with an empty list.

Symp core program may import other programs stored in separate files, using the `ALIAS` section with the `FILE` section. Each program is given a unique alias name where the first parameter determines a referent name while the second parameter points to a `SYMP` file. Functions of imported programs are called by noting the referent name followed by the `.` and the function name. Also, instead of importing a file, we can simply inline the `SYMP` section with all the functions, keeping them all in the same file.

## 3. Examples

We present seven examples each one building on the previous. Every example will be a full program with one or more functions, increasing in complexity.

#### Example 1 — A Constant Function

The absolute simplest Symp program: returns a hardcoded value.

```
(SYMP
    (CONSTANT answer 42)
)
```

Calling:

```
(CALL answer)
```

→ `42`

---

#### Example 2 — Identity Function

A function with one parameter.

```
(SYMP
    (FUNCTION id
        (PARAMS x)
        (RESULT x))
)
```

Call:

```
(CALL id hello)
```

→ `hello`

---

#### Example 3 — Boolean Branching

Use of built-in `IF`.

```
(SYMP
    (FUNCTION is-hello
        (PARAMS x)
        (RESULT
            (IF (EQ x hello)
                true
                false)))
)
```

Call:

```
(CALL is-hello hello)
```

→ `true`

---

#### Example 4 — List Manipulation (HEAD/TAIL/CONS)

A function that returns the first element of a list.

```
(SYMP
    (FUNCTION first
        (PARAMS xs)
        (RESULT
            (HEAD xs)))
)
```

Call:

```
(CALL first (a b c))
```

→ `a`

---

#### Example 5 — List Length (Recursive)

A classic example showing recursion.

```
(SYMP
    (FUNCTION length (PARAMS xs)
        (RESULT
            (IF (EQ xs ())
                zero
                (CONS one (CALL length (TAIL xs))))))
)
```

This returns a unary count like `(one (one (one zero)))` rather than a decimal number.

Call:

```
(CALL length (a b c))
```

→ `(one (one (one zero)))`

---

#### Example 6 — Factorial (Direct Recursion + Conditionals)

Using symbolic numbers.

```
(SYMP
    (FUNCTION factorial (PARAMS n)
        (RESULT
            (IF (EQ n 0)
                1
                (CALL multiply
                    n
                    (CALL factorial (CALL subtract n 1))))))
  
    (FUNCTION subtract (PARAMS a b)
        (RESULT
            (... your arithmetic ...)))

    (FUNCTION multiply (PARAMS a b)
        (RESULT
            (... your arithmetic ...)))
)
```

(You can plug in your preferred numeric encoding.)

---

#### Example 7 — Map Over a List

A higher-order structure without higher-order functions — you explicitly pass the function name.

```
(SYMP
    (FUNCTION map (PARAMS f xs)
        (RESULT
            (IF (EQ xs ())
                ()
                (CONS
                    (CALL f (HEAD xs))
                    (CALL map f (TAIL xs))))))
)
```

Call:

```
(CALL map increment (1 2 3))
```

---

We skimmed over seven examples, ranging from literals to branching, to lists, to recursion, and to higher order constructs. The examples are not intended to show the full potential of Symp core, yet to provide an introduction to core capabilities. For more advanced examples of meta-programming, interested reader is invited to read the accompanying documentation.

## 4. Conclusion

The Symp core provides a deliberately minimal foundation for symbolic computation. Its strict S-expression syntax, fixed-arity functions, and small set of primitives allow users to construct predictable and transparent computational structures. The examples provided illustrate how expressive behavior can emerge from a core symbolic substrate.
 
