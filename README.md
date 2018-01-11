Hi, Emacs! -*- mode: gfm -*-

# Maclisp-like `GENSYM` for LFE


## Table of Contents
   * [Maclisp-like GENSYM for LFE](#maclisp-like-gensym-for-lfe)
      * [Dictionary](#dictionary)
         * [GENSYM [function]](#gensym-function)
         * [GENSYM_COUNTER [function]](#gensym_counter-function)
         * [GENSYM_LIMIT [function]](#gensym_limit-function)
         * [ONCE-ONLY [macro]](#once-only-macro)
         * [SYSTEM_INFO [function]](#system_info-function)
         * [WITH-GENSYMS [macro]](#with-gensyms-macro)

## Dictionary

### `GENSYM` [function]
#### Syntax:
`gensym` ⇒ *new-symbol*

#### Arguments and Values:
* *new-symbol* — a fresh symbol.

#### Description:
Creates and returns a new symbol.  The symbol's prefix is of the form
*sym_<number>*; e.g. `sym_1`.  The *number* is incremented each time.

The value of *number* is that of `gensym_counter`.

#### Examples:
``` lisp
(gensym) ⇒ sym_24
```

#### Side Effects:
The resulting symbol is really an Erlang atom, and is interned in Erlang's atom
table.

Will increment `gensym_counter`.

Should the gensym counter reach the limit (as returned by `gensym_limit`), the
counter will wrap around to 1.

#### Affected By:
* `gensym_counter`

#### See Also:
* [`gensym_counter`](#gensym_counter-function)
* [`gensym_limit`](#gensym_limit-function)

**Notes**:
1. Unlike Common Lisp or Maclisp, it is not possible to modify the name of the 
   generated symbol.  This is to try to avoid allowing one to shoot oneself in
   the foot by creating too many Erlang atoms.

### `GENSYM_COUNTER` [function]
#### Syntax:
`gensym_counter` ⇒ *number*

#### Arguments and Values:
* *number* — a non-negative integer.

#### Description:
Returns a numeric value that will be used in constructing the name of the next
symbol generated by `gensym`.

#### Examples:
``` lisp
(gensym_counter) ⇒ 18
(gensym) ⇒ sym_18
(gensym_counter) ⇒ 19
```

#### Affected By:
* `gensym`

#### See Also:
* [`gensym`](#gensym-function)

#### Notes:
1. Unlike Common Lisp, this is a function and not a variable.

### `GENSYM_LIMIT` [function]
#### Syntax:
`gensym_limit` ⇒ *number*

#### Arguments and Values:
* *number* — a non-negative integer.

#### Description:
Returns the maximum amount of symbols that may be created by `gensym`.

#### Example:
``` lisp
(gensym_limit) ⇒ 104857
```

#### See Also:
* [`system_info`](#system_info-function)

#### Notes:
1. The gensym limit shall be considered to be a value that is approximate to 10%
   of the Erlang atom table size.  This can be changed with the `-t` CLI
   argument.

### `ONCE-ONLY` [macro]
#### Syntax:
`once-only` `(`*symbol_1 symbol_n ...* `)` *form ...* `)` ⇒ *result*

#### Arguments and Values:
* *symbol* — a symbol.
* *form* — a form.
* *result* — a form.

#### Description:
Execute *form* in a context where the values bound to the symbols in
*symbol_1* through *n* are bound to fresh gensyms, and the symbols in
*symbol_1* through *n* are bound to the corresponding gensym.

This macro ensures that symbols passed as variables are only executed once
within the context of the body form.

Taken from Peter Norvig's *Paradigms of Artificial Intelligence Programming*.

#### Example:
``` lisp
  (defun random (limit)
    (erlang:rem
     (erlang:trunc (* (rand:uniform) limit))
     limit))
⇒ random
  (defmacro square (x)
    (once-only (x)
      `(* ,x ,x)))
⇒ square
  (macroexpand-all '(square (random 10)) $ENV)
⇒ (let ((sym_1 (random 10)))
    (call 'erlang '* sym_1 sym_1))
  (square (random 10))
⇒ 64
```

#### See Also:
* [`with-gensyms`](#with-gensyms-macro)

### `SYSTEM_INFO` [function]
#### Syntax:
`system_info` ⇒ `(tuple` *limit count* `)`

#### Arguments and Values:**
* *limit* — a non-negative integer.
* *count* — a non-negative integer.

#### Description:
Returns information about the gensym system.

*Limit* is the maximum number of symbols that can be generated with `gensym`.
*Count* is the total number of symbols that have been created with `gensym`.

#### Example:
``` lisp
(lfe_gensym:system_info) ⇒ #(104857 0)
(lfe_gensym:gensym) ⇒ sym_1
(lfe_gensym:system_info) ⇒ #(104857 1)
```

#### Affected By:
* `gensym`

#### See Also:
* [`gensym_limit`](#gensym_limit-function)

### `WITH-GENSYMS` [macro]
#### Syntax:
`with-gensym` `(`*symbol_1 symbol_n ...* `)` *form ...* `)` ⇒ *result*

#### Arguments and Values:
* *variable* — a symbol.
* *form* — a form.
* *result* — a form.

#### Description:
Execute *form* in a context where the values bound to the symbols in
*symbol_1* through *n* are bound to fresh gensyms

Taken from Paul Graham's *On Lisp*.

#### Example:
``` lisp
  (defmacro nif (val pos zero neg)
    (with-gensyms (thing)
      `(let ((,thing ,val))
        (cond ((> ,thing 0)  ,pos)
              ((== ,thing 0) ,zero)
              (else          ,neg)))))
⇒ nif
  (macroexpand-all '(nif 42 'positive 'zero 'negative) $ENV)
⇒ (let ((sym_14 42))
    (if (call 'erlang '> sym_14 0)
      (progn 'positive)
      (if (call 'erlang '== sym_14 0)
          (progn 'zero)
          (progn 'negative))))
  (nif 42
    (io:format "Positive~n" '())
    (io:format "Zero~n" '())
    (io:format "Negative~n" '()))
⇒ Positive
```

#### See Also:
* [`once-only`](#once-only-macro)
