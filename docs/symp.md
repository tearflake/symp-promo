---
layout: docs
---

# symp intro

> **[about document]**  
> Introduction to *Symp*, a symbolic processing framework
>
> **[intended audience]**  
> Advanced programmers
> 
> **[abstract]**  
> Symp is a symbolic programming framework based on a strict separation between a stateless, referentially transparent core and a set of stateful devices. Computation, configuration, and communication are uniformly represented as symbolic expressions, enabling interactive programs to be constructed in a minimal and formally tractable setting. This document introduces the core concepts of Symp and illustrates their use with a representative example.

## table of contents

- [1. introduction](#1-introduction)  
- [2. theoretical background](#2-theoretical-background)
- [3. informative example](#3-informative-example)  
- [4. conclusion](#4-conclusion)  

## 1. introduction

Symbolic programming systems provide a natural medium for expressing computation in a form close to its mathematical description. Symp is a small, principled system in this tradition, designed to favor simplicity, explicit structure, and semantic clarity over syntactic convenience or performance.

The defining feature of Symp is a strict separation between a stateless, referentially transparent core and a set of stateful devices that interface with the external world. This document introduces the basic ideas behind this architecture and presents a representative example illustrating how interactive programs are constructed in Symp.

## 2. theoretical background

Symp is a symbolic programming system prioritizing simplicity and minimalism. It is intended to serve as a formally supported computing environment for different kinds of tasks, ranging from bringing simple decisions to solving complex programming tasks.

Symp program represents a core paired with a number of devices (Pagefront is the only device as of the time of writing this document). Core is a network of stateless and referentially transparent constants and functions. Devices represent an interface to the outer world, and can hold states during their lifetime.

Devices exchange messages using the core. Initial device configuration can be read from the core constants and functions (constant `main` as of the time of writing this document). Subsequent updates to running devices are performed by sending messages through the core. Messages sent through the core are of the form `(SEND core <message>)` or `(BATCH (SEND core <message>)+)`. Response messages from the core are of the form `(SEND <device> <message>)` or `(BATCH (SEND <device> <message>)+)`.

The system of stateful devices linked by stateless core may form an interactive program whole, resembling a virtual machine dedicated to accomplishing assigned tasks. Its appearance coincides with physical machine setup consisting of a central processor unit and peripheral devices.

## 3. informative example

Here, we bring a composite example of a simplistic calculator with its own user interface. The purpose of this example is not to give a comprehensive explanation about how Symp works, but to exhibit a representative form of a Symp application. A more interested reader may return to this example after completing reading the entire documentation. The example is called "Unary Calculator" because it is performing computations on unary numbers.

We start our exposure with the `main.symp` file defining user interface and boilerplate function calls:

```
(SYMP
    (ALIAS u (FILE "unary.symp"))
    
    (CONSTANT
        main
        (PAGEFRONT
            (ID "top")
            (HDR1 "Unary Calculator")
            (PARAG (ID "num") "Z")
            (HBLIST
                (BUTTON
                    (CAPTION "Increment")
                    (SEND core (add1 num)))
                
                (BUTTON
                    (CAPTION "Double")
                    (SEND core (mul2 num)))
                
                (BUTTON
                    (CAPTION "Square")
                    (SEND core (pow2 num)))
                
                (BUTTON
                    (CAPTION "Reset")
                    (SEND core (setParag "Z"))))))

    (FUNCTION
        setParag
        (PARAMS x)
        (RESULT
            (SEND
                device
                (SET
                    (TARGET "num")
                    (PARAG (ID "num") x)))))

    (FUNCTION
        add1
        (PARAMS x)
        (RESULT (CALL setParag (TOATOM (CALL u.add ("S" ("Z" ())) (TOLIST x))))))
    
    (FUNCTION
        mul2
        (PARAMS x)
        (RESULT (CALL setParag (TOATOM (CALL u.mul ("S" ("S" ("Z" ()))) (TOLIST x))))))
    
    (FUNCTION
        pow2
        (PARAMS x)
        (RESULT (CALL setParag (TOATOM (CALL u.pow (TOLIST x) ("S" ("S" ("Z" ()))))))))
)
```

The user interface consists of a title, a number, and three horizontally aligned buttons for incrementing, doubling, and squaring a number initially set to zero. Pressing the buttons will make a call to functions `add1`, `mul2`, and `pow2`, respectively.

Utility functions `add`, `mul`,and `pow` are placed within the `unary.symp` file. They are included in `main.symp` using the `ALIAS` section. The called utility functions are defined as follows:

```
(SYMP
    (FUNCTION
        add
        (PARAMS x y)
        (RESULT
            (IF
                (EQ (HEAD x) "Z")
                y
                ("S" (CALL add (TAIL x) y)))))
    
    (FUNCTION
        mul
        (PARAMS x y)
        (RESULT
            (IF
                (EQ (HEAD x) "Z")
                ("Z" ())
                (CALL add y (CALL mul (TAIL x) y)))))
    
    (FUNCTION
        pow
        (PARAMS x y)
        (RESULT
            (IF
                (EQ (HEAD y) "Z")
                ("S" ("Z" ()))
                (CALL mul x (CALL pow x (TAIL y))))))
)
```

Unary numbers produced/consumed by these functions are of the following notation:

```
0 = (Z ())
1 = (S (Z ()))
2 = (S (S (Z ())))
3 = (S (S (S (Z ()))))
...
```

This kind of unary numbers is suitable for performing computations in a very simple manner, as shown in the file above.

## 4. conclusion

This document has outlined the fundamental design of Symp and illustrated its use through a simple interactive example. The separation of pure symbolic computation from stateful devices allows interactive programs to be built while preserving formal clarity and referential transparency.

Further documentation will elaborate on the formal semantics of the system and the device model in greater detail. The example presented here is intended as a reference point that becomes more informative as the reader’s understanding of Symp deepens.

