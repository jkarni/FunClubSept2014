
# Overview

- Contracts
- Deep and shallow DSL embeddings
- Reflection without remorse

   
---
name: contract-demo
layout: false
 
# Contracts


```racket
(succ : pos? -> pos?) ;; contract
(define (succ x)
    (+ x 1))
```
--
                 |    |
      succ       |    |      x
                 |    | 

---
template: contract-demo

                 |    |
      succ       |    |  <-  x
                 |    | 

---
template: contract-demo
 
                 |    |
      succ       | x  | 
                 |    | 

---
template: contract-demo
  
                 |    |
      succ       | x  | 
                 |    | 
                   ^---- (pos? x)

---
template: contract-demo
  
                 |    |
      succ       | x  | 
                 |    | 
                   ^---- (false)

---
template: contract-demo
  
                 |    |
      succ       | x  | 
                 |    | 
                   ^---- (false) BLAME CONTEXT

---
template: contract-demo
  
                 |    |
      succ       | x  | 
                 |    | 
                   ^---- (pos? x)
                   
---
template: contract-demo
  
                 |    |
      succ       | x  | 
                 |    | 
                   ^---- (true)
---
template: contract-demo
  
                 |    |
      succ     x |    | 
                 |    | 
 
---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 

---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 
        ^--- (pos? (succ x))

---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 
        ^--- (false)

---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 
        ^--- (false)   BLAME succ
---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 
        ^--- (pos? (succ x))

---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 
        ^--- (true)

---
template: contract-demo
  
                 |    |
      succ x     |    | 
                 |    | 
        ^--- (true)   CARRY ON

---
name:static-checking

## Static checking


---
template: static-checking

```racket
(f : pos? -> neg?) ;; contract
(define (f x)
    (* x -1))
```
---
template: static-checking

```racket
(f : pos? -> neg?) ;; contract
(* x -1))
```

---
template: static-checking

```racket
(f : pos? -> neg?) ;; contract
(* x -1))
```
'x' can be assumed to be positive

---
template: static-checking

```racket
(f : pos? -> neg?) ;; contract
(* {pos} -1))
```

---
template: static-checking

```racket
(f : pos? -> neg?) ;; contract
(* {pos} -1))
```
Fact: `* -1` {pos} -> {neg}

---
template: static-checking

```racket
(f : pos? -> neg?) ;; contract
({neg})
```

---
name: higher-order

But what about higher-orders? 

---
template: higher-order

```racket
(e2o : (even? -> even?) -> (odd? -> odd?)
(define (e2o f)
 (lambda (n) (- (f (+ n 1)) 1)))
```
 
-- 

> The key insight is that contracts delay higher-order checks and failures
> always occur with a first order witness.
 
  
--

Just keep the boundary checking inside the higher-order function.

---
name: proof
```racket
(e2o : (even? -> even?) -> (odd? -> odd?)
(define (e2o f)
 (lambda (n) (- (f (+ n 1)) 1)))
```
---
template: proof

Can assume:
- `(f : even? -> even?)`

---
template: proof

Can assume:
- `(f : even? -> even?)`


Must prove:
- `f` is called with `x: even?`

---
template: proof

Can assume:
- `(f : even? -> even?)`



Must prove:
- `f` is called with `x: even?`

---
template: proof

Can assume:
- `(f : even? -> even?)`
- `(n : odd?)`


Must prove:
- `f` is called with `x: even?`

---
template: proof

Can assume:
- `(f : even? -> even?)`
- `(n : odd?)`
- `(+ : {odd?} 1) -> {even?}` 

Must prove:
- `f` is called with `x: even?`

---
template: proof

Can assume:
- `(f : even? -> even?)`
- `((+ n 1) : even?)`

Must prove:
- `f` is called with `x: even?`

---
template: proof

Can assume:
- `(f : even? -> even?)`
- `((+ n 1) : even?)`

Must prove:
- `f` is called with `x: even?` - DONE

---

# Deep and Shallow Embeddings


Two options

## Deep

```haskell
data Exp :: * where
    Lit :: Int -> Exp
    Add :: Exp -> Exp -> Exp

evalD (Lit x)    = x
evalD (Plus x y) = evalD x + evalD y
```

--

## Shallow

```haskell
type Exp = Int

lit n = n
add x y = x + y

evalS :: Exp -> Int
evalS = id
```
        
---

Deep and shallow embeddings as duals:

- Lit and Add do none of the work, evalD all of it.
- lit and add do all of the work, evalS is just id.
- Easy to add other 'evaluators' to deep (such as pretty-printing).
- Hard to do the same for shallow.
- Hard to add another construct to deep.
- Easy to do the same for shallow.

The expression problem. 

---

## Folds and Multiple interpretations


How do we get multiple interpretations for shallow embeddings?

-- 

```haskell
type Exp = (Int, String)

lit n = (n, show n)
add x y = (x + y, show x ++ " + " ++ show y)

evalSInt :: Exp -> Int
evalSInt = fst

evalSShow :: Exp -> Show
evalSShow = snd
```

--
But what if we don't want to commit ahead of time to the ways in which we will
interpret the data?

---

## Reflection without Remorse


Lists as our running analogy:

- List append efficiency depends on associativity of operation.
    * Solution: difference lists
        - [1,2,3] ==> \xs -> [1,2,3] ++ xs
        - [1,2,3] ++ [4,5,6] ==> (\xs -> [1,2,3] ++ xs) . (\ys -> [4,5,6] ++ ys)
        -                    ==> (\zs -> [1,2,3] ++ ([4,5,6] ++ zs))
    * But now 'observing' the list is expensive.
    * Solution: Okasaki's bootstrapped data types.

---
layout: false

Observation: Monads often have this property (associativity changes
performance). 
- Example: Free Monads

```haskell
data Free f r = Free (f (Free f r)) | Pure r
instance (Functor f) => Monad (Free f) where
    return = Pure
    (Free x) >>= f = Free (fmap (>>= f) x)
    (Pure r) >>= f = f r
```
--
So
```
x >>= f >>= g = g (f x)
Free a >>= f >>= g = Free (fmap (>>= f) x) >>= g
Free a >>= f >>= g = Free (fmap (>>= g) (fmap (>>= f) x)))
Free a >>= (f . g) = Free (fmap (>>= f . g) x)
```
 
Can we apply the same tricks to them?

