# [blog](http://blog.vmchale.com/)

-   [ALL](http://blog.vmchale.com/) 
-   [PROGRAMMING](http://blog.vmchale.com/category/Programming) 
-   [HASKELL](http://blog.vmchale.com/category/Haskell) 
-   [IDRIS](http://blog.vmchale.com/category/Idris) 
-   [ATS](http://blog.vmchale.com/category/ATS)
-   [About](http://blog.vmchale.com/about)
-   [Tips](http://blog.vmchale.com/tips) 

## Projective Programming

by [Vanessa McHale](http://blog.vmchale.com/author/Vanessa%20McHale) | 2018-09-10 00:36

I read a recent [Functional Pearl](http://www.cs.ox.ac.uk/ralf.hinze/publications/ICFP09.pdf) by Hinze and this inspired me to write up an example of projective programming and its motivation in logic/model theory.

The key to understanding the value of abstraction comes from logic and model theory: when you have more axioms, you have fewer models satisfying the axioms and the axioms prove more theorems; conversely, when you have fewer axioms you have more models and fewer theorems. This means that something more abstract can be in some sense simpler.

Let us consider an example in Haskell, viz.

```haskell
concat :: [[a]] -> [a]
```

This can we written several ways:

```haskell
concat :: [[a]] -> [a]
concat []     = []
concat (x:xs) = x ++ concat xs
```

```haskell
concat :: [[a]] -> [a]
concat = foldr (++) []
```

```haskell
concat :: [[a]] -> [a]
concat = concatMap id
```

```none
{-# LANGUAGE DeriveFunctor #-}

newtype Fix f = Fix { unFix :: f (Fix f) }

data ListF a b = Cons a b
               | Nil
               deriving (Functor)

cata :: Functor f => (f a -> a) -> (Fix f -> a)
cata f = g where g = f . fmap g . unFix

fixList :: [a] -> Fix (ListF a)
fixList []     = Fix Nil
fixList (x:xs) = Fix (Cons x (fixList xs))

concat :: [[a]] -> [a]
concat = cata a . fixList
    where a Nil         = []
          a (Cons x xs) = x ++ xs
```

By contrast,

```haskell
join :: Monad m => m (m a) -> m a
```

is expressible only as

```none
join :: Monad m => m (m a) -> m a
join = (>>= id)
```

Note that `join` is a generalization of `concat`.

Despite the fact that `join` is more abstract, it is easier to come up with its implementation: we write the only sensible thing that typechecks.

This is not the case with our first implementation of `concat`; we could have easily written

```haskell
concat :: [[a]] -> [a]
concat []     = []
concat (x:xs) = concat xs ++ x
```

# Coda

I believe programming in this style is a place where Haskell and its descendants have an advantage over other mainstream programming languages, notably due to typeclasses. I do not think this is the final word on abstraction by any means, but it does provide a poignant demonstration of the limits of non-functional programming languages.

[return](http://blog.vmchale.com/)

[](http://blog.vmchale.com/)

[

](http://blog.vmchale.com/)

[](http://blog.vmchale.com/)  Save [![](http://blog.vmchale.com/img/atom.png)](http://blog.vmchale.com/atom)