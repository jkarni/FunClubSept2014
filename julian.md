
# Overview

- Contracts
    * Soft Contract Verification, P. Nguyên et al.
- Deep and shallow DSL embeddings
    * Folding Domain-Specific Languages, J. Gibbons and N. Wu
- Reflection without remorse
    * Reflection Without Remorse, A. van der Ploeg and O. Kiselyov

   
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
template: proof

Can assume:
- `(f : even? -> even?)`
- `((+ n 1) : even?)`

---
template: proof

Can assume:
- `(f (+ n 1) : even?)`

---
template: proof

Can assume:
- `(f (+ n 1) : even?)`
- `(- : {even?} 1) -> odd?`

---
template: proof

Can assume:
- `(- (f (+ n 1)) 1) : odd?)`

---
template: proof

Can assume:
- `(- (f (+ n 1)) 1) : odd?)`

DONE

---
name: deep-and-shallow
# Deep and Shallow Embeddings


---
template: deep-and-shallow

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
template: deep-and-shallow

Deep and shallow embeddings as duals:

--

- Lit and Add do none of the work, evalD all of it.

--

- lit and add do all of the work, evalS is just id.

--

- Easy to add other 'evaluators' to deep (such as pretty-printing).

--

- Hard to do the same for shallow.

--

- Hard to add another construct to deep.

--

- Easy to do the same for shallow.

--

The expression problem. 

---
name: multiple-interps
## Multiple interpretations


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
template: multiple-interps

```haskell
type Exp = Int

lit n = \(f, _) -> f n
add x y = \(_, s) -> s x y

evalSInt :: Exp -> Int
evalSInt e = e (id, (+)) 

evalSShow :: Exp -> Show
evalSShow e = e (show, (++))
```

--

Very easy to add more interpretations!

--

But note that it's gotten somewhat harder to add new constructs.

---
# Folds

--

There are a lot more cools things in the paper

--

Unified by the theme of treating shallow embeddings as folds.

--

- Banana Split Theorem

--

- Mutumorphisms


---
name: refl
## Reflection without Remorse

--


Lists as our running analogy:

- List append efficiency depends on associativity of operation.
    * Solution: difference lists
        - [1,2,3] ==> \xs -> [1,2,3] ++ xs
        - [1,2,3] ++ [4,5,6] ==> (\xs -> [1,2,3] ++ xs) . (\ys -> [4,5,6] ++ ys)
        -                    ==> (\zs -> [1,2,3] ++ ([4,5,6] ++ zs))
    * But now 'observing' the list is expensive.
    * Solution: Okasaki's bootstrapped data types.

---
template: refl

Observation - Monads often have this property (associativity changes
performance). 

--

- Example: Trees, Free Monads

--

Can we apply the same tricks to them?

---

template: refl

```haskell
data Tree a = Node (Tree a) (Tree a)
            | Leaf a

(<~) :: Tree a -> (a -> Tree b) -> Tree b
Leaf x <~ f = f x
Node l r <~ f = Node (l <~ f) (r <~ f)

instance Monad Tree where
    return = Leaf
    (>>=) = (<~)
```

--
```haskell
(m >>= f) >>= g == m >>= (\x -> f x >>= g)
```
--
Or more clearly:
```haskell
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
(f >=> g) >=> h == f >=> (g >=> h)
```

---
template: refl

```haskell
newtype Tree = Tree (CQueue TreeView)
data TreeView = Node Tree Tree
              | Leaf

toView :: Tree -> TreeView

fromView :: TreeView -> Tree
fromView x = Tree $ singleton x

(<~) :: Tree -> Tree -> Tree
Tree l <~ Tree r = Tree (l `concat` r)
```

