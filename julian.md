
Overview
========

- Contracts
- Deep and shallow DSL embeddings
- Reflection without remorse

Contracts
=========

First-order:

`racket
(pos? -> pos?)
(define (add1 x)
    (+ x 1))
`

---

But what about higher-orders? 

`racket
((pos? -> pos?) -> (neg? -> neg?)
(define (neg f)
 (lambda (x)
  (- (f (- x)))))
`

-- 

Just keep the boundary checking inside the higher-order function.

---

Static checking
---------------
`
(lambda ([pos? -> pos?]
    (- ([pos? -> pos?] (- x))))
`

'x' must be negative, or the context is to blame.

`
(lambda ([pos? -> pos?]
    (- ([pos? -> pos?] (- [neg?]))))
`
Fact: (- x) => [neg? -> pos?]
`
(lambda ([pos? -> pos?]
    (- ([pos? -> pos?] ([neg?]))))
`

Deep and Shallow Embeddings
===========================

Two options

Deep
-----
`haskell
data Exp :: * where
    Lit :: Int -> Exp
    Add :: Exp -> Exp -> Exp

evalD (Lit x)    = x
evalD (Plus x y) = evalD x + evalD y
`

--

Shallow
-------
`haskell
type Exp = Int

lit n = n
add x y = x + y

evalS :: Exp -> Int
evalS = id
`
        
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

Folds and Multiple interpretations
----------------------------------

How do we get multiple interpretations for shallow embeddings?

-- 
`haskell
type Exp = (Int, String)

lit n = (n, show n)
add x y = (x + y, show x ++ " + " ++ show y)

evalSInt :: Exp -> Int
evalSInt = fst

evalSShow :: Exp -> Show
evalSShow = snd
`

--
But what if we don't want to commit ahead of time to the ways in which we will
interpret the data?

---

Reflection without Remorse
==========================

Lists as our running analogy:

- List append efficiency depends on associativity of operation.
    * Solution: difference lists
        - [1,2,3] ==> \xs -> [1,2,3] ++ xs
        - [1,2,3] ++ [4,5,6] ==> (\xs -> [1,2,3] ++ xs) . (\ys -> [4,5,6] ++ ys)
        -                    ==> (\zs -> [1,2,3] ++ ([4,5,6] ++ zs))
    * But now 'observing' the list is expensive.
    * Solution: Okasaki's bootstrapped data types.

---

Observation: Monads often have this property (associativity changes
performance). 
- Example: Free Monads

`haskell
data Free f r = Free (f (Free f r)) | Pure r
instance (Functor f) => Monad (Free f) where
    return = Pure
    (Free x) >>= f = Free (fmap (>>= f) x)
    (Pure r) >>= f = f r
`
--
So
`
x >>= f >>= g = g (f x)
Free a >>= f >>= g = Free (fmap (>>= f) x) >>= g
Free a >>= f >>= g = Free (fmap (>>= g) (fmap (>>= f) x)))
Free a >>= (f . g) = Free (fmap (>>= f . g) x)
`
 
Can we apply the same tricks to them?

