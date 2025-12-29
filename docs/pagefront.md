---
layout: docs
---

# pagefront specs

> **[about document]**  
> Specification of *Pagefront*, a symbolic rendering engine
>
> **[intended audience]**  
> Intermediate programmers
> 
> **[abstract]**  
> This document describes the Pagefront markup format, an S-expression-based alternative to HTML. It defines the syntax, semantics, and structure of the language, along with a short illustrative example.

## table of contents

- [1. Introduction](#1-introduction)  
- [2. Syntax](#2-syntax)
- [3. Semantics](#3-semantics)  
- [4. Example](#4-example)  
- [5. Conclusion](#5-conclusion)  

## 1. Introduction

Pagefront is a symbolic markup format designed to describe simple textual user interfaces in a structured, text-based form. It uses a uniform S-expression notation to define hierarchical layouts, interactive elements, and textual content. Its purpose is to provide a lightweight, extensible, and easily parsable format that can serve as an alternative to HTML in environments where symbolic representation and compact syntax are preferred.

## 2. Syntax

These are the syntax rules of Pagefront:

```
<start> := <pageFront>

<pageFront> := (PAGEFRONT (ID <ATOMIC>)? (PARAMS <ANY>+)? <var>+ <content>)

<var> := (VAR (ID <ATOMIC>) <ATOMIC>)

<content> := (FRAMENS
                 (ID <ATOMIC>)?
                 (NORTH (ID <ATOMIC>)? (HEIGHT <ATOMIC>) (CONTENT <content>))
                 (CENTER (ID <ATOMIC>)? (CONTENT <content>))
                 (SOUTH (ID <ATOMIC>)? (HEIGHT <ATOMIC>) (CONTENT <content>)))
           
           | (FRAMEWE
                 (ID <ATOMIC>)?
                 (WEST (ID <ATOMIC>)? (WIDTH <ATOMIC>) (CONTENT <content>))
                 (CENTER (ID <ATOMIC>)? (CONTENT <content>))
                 (EAST (ID <ATOMIC>)? (WIDTH <ATOMIC>) (CONTENT <content>)))
           
           | <formElement>+

<formElement> := (HDR1 (ID <ATOMIC>)? <ATOMIC>)
               | (HDR2 (ID <ATOMIC>)? <ATOMIC>)
               | (HDR3 (ID <ATOMIC>)? <ATOMIC>)
               | (HDR4 (ID <ATOMIC>)? <ATOMIC>)
               | (HDR5 (ID <ATOMIC>)? <ATOMIC>)
               | (HDR6 (ID <ATOMIC>)? <ATOMIC>)
               | (PARAG (ID <ATOMIC>)? <ATOMIC>)
               | (HRULER (ID <ATOMIC>)?)
               | (ULIST (ID <ATOMIC>)? <formElement>+)
               | (OLIST (ID <ATOMIC>)? <formElement>+)
               | (CODE (ID <ATOMIC>)? <ATOMIC>)
               | (CHECK (ID <ATOMIC>)? <formElement>+)
               | (RADIO (ID <ATOMIC>)? <formElement>+)
               | (EDIT (ID <ATOMIC>)? (ROWS <ATOMIC>)? (COLS <ATOMIC>)? <ATOMIC>)
               | (HBLIST (ID <ATOMIC>)? (BUTTON (ID <ATOMIC>)? (CAPTION <ATOMIC>)? (SEND <ATOMIC> <ANY>*)))
               | (VBLIST (ID <ATOMIC>)? (BUTTON (ID <ATOMIC>)? (CAPTION <ATOMIC>)? (SEND <ATOMIC> <ANY>*)))
```

We apply the same syntax definition nomenclature as noted in Symp specification document. In addition, we reserve some freedom to make an order of some sections arbitrary where appropriate.

## 3. Semantics

Here, we shortly enumerate the components which may form the GUI of your application:

- `PAGEFRONT` - the document root. It contains the document contents, and may take a number of parameters used in document construction. It also may define a number of internal global variables used for storing session states.
- `FRAMENS`: spans three containers on the page in the north/center/south direction. `FRAMENS` and `FRAMEWE` may be combined, or entirely skipped.
- `FRAMEWE`: spans three containers on the page in the west/center/east direction. `FRAMENS` and `FRAMEWE` may be combined, or entirely skipped.
- `HDR1` - `HDR6`: document headers expressed with quoted string.
- `PARAG`: paragraph sections.
- `HRULER`: a horizontal ruler.
- `ULIST`: bullet-pointed unordered list.
- `OLIST`: automatically numbered ordered list.
- `CODE`:  a multiline string describing a code block rendered inverted.
- `CHECK`: an interactive checkbox which may be referred to in submitting a form.
- `RADIO`: interactive radio buttons group which may be referred to in submitting a form.
- `EDIT`: a text box used as an editor which may be referred to in submitting a form.
- `HBLIST`: a horizontal button list. A `SET` action replaces the `TARGET` node’s content subtree with the S-expression provided.
- `VBLIST`: a vertical button list. A `SET` action replaces the `TARGET` node’s content subtree with the S-expression provided.

`PARAG`, `ULIST`, `OLIST`, `CHECK`, and `RADIO` content strings are interpreted in a following way: in the parameter string, words enclosed within a pair of backticks shown in inverted color (used for inline code blocks). There is also a possibility to create hyperlinks denoted by the `[...text...](...link path...)` pattern within strings.

Buttons in `HBLIST` and `HVLIST`, when clicked, send to the core a message in a form of function/constant call. The core may respond in sending back a target id bundled with a new content of the referred element. 

## 4. Example

The following example demonstrates a simple Pagefront document describing a small user interface with headers, text, an image, and a simple form.

```
(PAGEFRONT
    (ID "top")
    (FRAMENS
        (NORTH (HEIGHT "20%")
            (CONTENT
                (HDR1 "Pagefront Example Page")
                (PARAG "A simple layout showcasing *Pagefront* elements.")))
        
        (CENTER
            (HDR2 "Interactive Form")
            (PARAG "Below is an example of user input elements:")
            (CHECK (ID "agree")
                (PARAG "I **agree** to the terms and conditions."))
                
            (PARAG "Choose direction:")
            (RADIO (ID "direction")
                (PARAG "*Foreward*")
                (PARAG "*Backward*")
                (PARAG "*Left*")
                (PARAG "*Right*"))
                
            (EDIT (ID "feedback") "4")
            (HBLIST (BUTTON (CAPTION "Submit Form") (SEND "submit" (agree direction feedback))))
            HRULER
            (CODE
                """
                echo "Pagefront form example"
                form_action();
                """))
                
        (SOUTH (HEIGHT "2em")
            (PARAG "Pagefront Project"))))
```

This example defines a page where the main area is divided into a vertical frame with a header, central frame, and footer. A simple interactive form is included, demonstrating checkboxes, radio buttons, text input, and a navigation button. The button, once clicked, sends a message to the core, and the core is responsible of what happens next.

## 5. Conclusion

Pagefront provides a structured, S-expression-based representation of GUI documents suitable for deterministic parsing and compact storage. It offers basic layout and interactivity features inspired by HTML, yet remains minimal and adaptable for embedded or symbolic systems.

