
# Overview

- Ivory and Tower
- Haste
- ghcjs


---

# Ivory and Tower
--

## An experience report

- Autopilot for a military helicopter
- Deadline: 18 months
- http://dl.acm.org/citation.cfm?id=2628146&CFID=559949482&CFTOKEN=36745757

--

## Strategy

1. Design two new embedded programming languages from scratch
    - (14 person months)
--
2. Use these languages to write the actual code
    - (22 person months)

---

## Ivory

- Generates C-code
- EDSL itself is very restrictive (but Turing-complete)
- Use Haskell as macro system
- 6k lines of Haskell code

--

```haskell
[ivory|
struct fooStruct
  { bar :: Stored Uint8
  ; baz :: Array 10 (Stored Sint16)
  }
|]

setBaz :: Def ([Ref Global (Struct "fooStruct"), Sint16] :-> ())
setBaz = proc "setBaz" $ \ ref val -> body (prgm ref val)

prgm :: Ref Global (Struct "fooStruct") -> Sint16 -> Ivory eff ()
prgm ref val = arrayMap $ \ ix -> store ((ref ~> baz) ! ix) val
```

---

## Ivory

foo_module.h:
```c
//
struct fooStruct {
    uint8_t bar;
    int16_t baz[10U];
};

void setBaz(struct fooStruct* n_var0, int16_t n_var1);
```
--

foo_source.c:
```c
#include "foo_module.h"

void setBaz(struct fooStruct* n_var0, int16_t n_var1) {
    for ( int32_t n_ix0 = (int32_t) 0
        ; n_ix0 <= (int32_t) 9
        ; n_ix0++ ) {
        n_var0->baz[n_ix0] = n_var1;
    }
}
```

---

## Ivory

Guarantees:

- Type-level array lengths!  (-:
- Well-typed dynamic memory management
- Well-typed C structs
- Well-typed register / bit fields
- No Nullable pointers
- No pointer arithmetic
- No void pointer
- No unsafe casts

---

## Tower

- Ivory extension
- tasks (~ threads or processes)
- communication channels
- multiple backends (RTOSes, theorem provers, Graphviz)
- 3k lines of Haskell code

[fig. 2]

---

## Lessons Learned

- SMACCMPilot is 23kloc Ivory, Tower (48kloc C compiled)
- Still uses some ArduPilot code
--

- Types are awesome
--

- Bugs in the C backend are easy to work around (robust!)
--

- Bugs in Ivory, Tower are easy to fix
--

- Bad habits can be outlawed by changing the language


---

# Haste
--

## seamless web application programming

- web application (one-page app)
- write both server and client code in haskell
- write data types and application logic *only once*
- communicate via RPC (currently: web sockets)
- https://github.com/valderman/haste-compiler

---

## example


---

# ghcjs
--

## The ICFP programming contest

(screenshot of lambdabot simulator)
[also link]

---

##

- technology

- demos

---

- code: https://github.com/ghcjs/ghcjs
- HIW talk: http://www.youtube.com/watch?v=pXBJc4e9KIE
