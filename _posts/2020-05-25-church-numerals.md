---
title: Church numerals
author: Lucas Paradisi
date: 2020-05-25 14:10:00 -0300
categories: [Programming, SICP]
tags: [church,lisp, racket]
---


I have been working on exercises 2.4 & 2.6 from [SICP book](https://mitpress.mit.edu/sites/default/files/sicp/index.html) (I also recommend the [video lectures](https://www.youtube.com/playlist?list=PLB63C06FAF154F047)) for two days so I wanted to share the solutions I came up on these.

## Exercise 2.4
> *Here is an alternative procedural representation of pairs. For this representation, verify that (car (cons x y)) yields x for any objects x and y.*

```racket
(define (cons x y)
  (lambda (m) (m x y)))

(define (car z)
  (z (lambda (p q) p)))
```

> *What is the corresponding definition of cdr? (Hint: To verify that this works, make use of the substitution model of Section 1.1.5)*

So this looks quite clear as ```cdr``` implementation should be similar to ```car``` (```cdr``` returns the second value on ```cons``` instead of the first one):

```racket
(define (cdr z)
  (z (λ(p q) q)))
```

```
> (define a (cons 1 2))
> (car a)
1
> (cdr a)
2
```

These are the substitution steps for ```(car z)``` with z as ```(cons 1 2)```:
```
(car z)
(z (λ(p q) p))
((cons 1 2) (λ(p q) p))
((λ(m) (m 1 2)) (λ(p q) p))
((λ(p q) p) 1 2)
1
```

## Exercise 2.6

> *In case representing pairs as procedures wasn't mind-boggling enough, consider that, in a language that can manipulate procedures, we can get by without numbers (at least insofar as nonnegative integers are concerned) by implementing 0 and the operation of adding 1 as*

```racket
(define zero
  (lambda(f) (lambda(x) x)))
  
(define (add-1 n)
  (lambda(f) (lambda(x) (f ((n f) x)))))
```

> *This representation is known as Church numerals, after its inventor, Alonzo Church, the logician who invented the λ calculus.*

> *Define one and two directly (not in terms of zero and add-1). (Hint: Use substitution to evaluate (add-1 zero)).*

This is how ```(add-1 zero)``` is evaluated:
```
(add-1 zero)
(λ(f) (λ(x) (f ((n f) x))))
(λ(f) (λ(x) (f (((λ(f) (λ(x) x)) f) x)))) <--- n replaced by zero ---
(λ(f) (λ(x) (f x)))
```

So the result of ```(add-1 zero)``` is the way of defining the number one:
```racket
(define one-directly
  (λ(f) (λ(x) (f x))))
    
(define two-directly
  (λ(f) (λ(x) (f (f x)))))
```

> *Give a direct definition of the addition procedure + (not in terms of repeated application of add-1).*

```racket
(define (add a b) 
  (λ(f) (λ(x) ((a f) ((b f) x)))))
```


I used a helper function to turn church numerals to integers, which is just a function acting as a accumulator, called as many times as f functions we have in the **value** parameter:

```racket
(define (count value)
  ((value (λ(a) (+ 1 a))) 0))
```



Let's use substitution to see how ```(count zero)``` is evaluated. Note how the definition of zero ignores the first argument received (which in this example is the accumulator function ```(λ(a) (+ 1 a))```) and uses directly the second argument 0:

```
(count zero)
((zero (λ(a) (+ 1 a))) 0)
(((λ(f) (λ(x) x)) (λ(a) (+ 1 a))) 0)
((λ(x) x) 0)
0
```
More obvious examples:
```
> (count (add one-directly two-directly))
3
> (count (add zero one-directly))
1
> (count (add zero zero))
0
> (count (add (add-1 (add-1 (add-1 (add-1 two-directly)))) (add-1 (add-1 (add-1 (add-1 two-directly))))))
12
```

Of course there are more operations like substract (taking care of negative values!), multiplication and exponentiation which could be added in a different post... 
