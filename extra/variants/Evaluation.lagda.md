PCF with nested evaluation contexts

Philip Wadler, 2 Aug 2022
```
module variants.Evaluation where

open import Data.Nat using (β; zero; suc; _+_)
open import Data.Bool using (true; false) renaming (Bool to πΉ)
open import Data.Unit using (β€; tt)
open import Data.Empty using (β₯; β₯-elim)
open import Data.Product using (_Γ_; _,_; projβ; projβ; Ξ£; β; Ξ£-syntax; β-syntax)
open import Data.Sum using (_β_; injβ; injβ) renaming ([_,_] to case-β)
open import Relation.Binary.PropositionalEquality
     using (_β‘_; _β’_; refl; trans; sym; cong; congβ; cong-app; subst; inspect)
     renaming ([_] to [[_]])
open import Relation.Nullary using (Β¬_; Dec; yes; no)
open import Relation.Nullary.Decidable using (β_β; True; toWitness; fromWitness)
```

## Types

```
infixr 7 _β_
infix  8 `β

data Type : Set where
  `β : Type
  _β_ : Type β Type β Type

variable
  A B C : Type
```

* Type environments

```
infixl 6 _β·_

data Env : Set where
  β   : Env
  _β·_ : Env β Type β Env

variable
  Ξ Ξ : Env

infix  4 _β_
infix  9 S_

data _β_ : Env β Type β Set where

  Z :
      Ξ β· A β A

  S_ :
      Ξ β A
      ---------
    β Ξ β· B β A

variable
  x y : Ξ β A
```

## Terms

```
infix  4  _β’_
infix  5  Ζ_
infix  5  ΞΌ_
infixl 6  _Β·_
infix  7  `suc_
infix  8  `_

data _β’_ : Env β Type β Set where

  `_ :
      Ξ β A
      -----
    β Ξ β’ A

  Ζ_ :
      Ξ β· A β’ B
      ---------
    β Ξ β’ A β B

  _Β·_ :
      Ξ β’ A β B
    β Ξ β’ A
      ---------
    β Ξ β’ B

  `zero :
      ------
      Ξ β’ `β

  `suc_ :
      Ξ β’ `β
      ------
    β Ξ β’ `β

  case :
      Ξ β’ `β
    β Ξ β’ A
    β Ξ β· `β β’ A
      -----------
    β Ξ β’ A

  ΞΌ_ :
     Ξ β· A β’ A
     ---------
   β Ξ β’ A

variable
  L M N V W : Ξ β’ A
```

## Renaming maps, substitution maps, term maps

```
_βα΄Ώ_ : Env β Env β Set
Ξ βα΄Ώ Ξ = β {A} β Ξ β A β Ξ β A

_βΛ’_ : Env β Env β Set
Ξ βΛ’ Ξ = β {A} β Ξ β A β Ξ β’ A

_βα΅_ : Env β Env β Set
Ξ βα΅ Ξ = β {A} β Ξ β’ A β Ξ β’ A

variable
  Ο : Ξ βα΄Ώ Ξ
  Ο : Ξ βΛ’ Ξ
  ΞΈ : Ξ βα΅ Ξ
```


## Renaming

```
renβ· :
    (Ξ βα΄Ώ Ξ)
    ------------------
  β (Ξ β· A) βα΄Ώ (Ξ β· A)
renβ· Ο Z      =  Z
renβ· Ο (S x)  =  S (Ο x)

ren :
    (Ξ βα΄Ώ Ξ)
    --------
  β (Ξ βα΅ Ξ)
ren Ο (` x)          =  ` (Ο x)
ren Ο (Ζ N)          =  Ζ (ren (renβ· Ο) N)
ren Ο (L Β· M)        =  (ren Ο L) Β· (ren Ο M)
ren Ο `zero          =  `zero
ren Ο (`suc M)       =  `suc (ren Ο M)
ren Ο (case L M N)   =  case (ren Ο L) (ren Ο M) (ren (renβ· Ο) N)
ren Ο (ΞΌ M)          =  ΞΌ (ren (renβ· Ο) M)

lift : Ξ βα΅ (Ξ β· A)
lift = ren S_
```

## Substitution

```
subβ· :
    (Ξ βΛ’ Ξ)
    ------------------
  β (Ξ β· A) βΛ’ (Ξ β· A)
subβ· Ο Z      =  ` Z
subβ· Ο (S x)  =  lift (Ο x)

sub :
    (Ξ βΛ’ Ξ)
    --------
  β (Ξ βα΅ Ξ)
sub Ο (` x)          =  Ο x
sub Ο (Ζ N)          =  Ζ (sub (subβ· Ο) N)
sub Ο (L Β· M)        =  (sub Ο L) Β· (sub Ο M)
sub Ο `zero          =  `zero
sub Ο (`suc M)       =  `suc (sub Ο M)
sub Ο (case L M N)   =  case (sub Ο L) (sub Ο M) (sub (subβ· Ο) N)
sub Ο (ΞΌ M)          =  ΞΌ (sub (subβ· Ο) M)
```

Special case of substitution, used in beta rule
```
Οβ :
    Ξ β’ A
    ------------
  β (Ξ β· A) βΛ’ Ξ
Οβ M Z      =  M
Οβ M (S x)  =  ` x

_[_] :
    Ξ β· A β’ B
  β Ξ β’ A
    ---------
  β Ξ β’ B
_[_] N M =  sub (Οβ M) N
```

## Values

```
data Value : (Ξ β’ A) β Set where

  Ζ_ :
      (N : Ξ β· A β’ B)
      ---------------
    β Value (Ζ N)

  `zero :
      Value {Ξ} `zero

  `suc_ :
      Value V
      --------------
    β Value (`suc V)

  ΞΌ_ :
      (N : Ξ β· A β’ A)
      ---------------
    β Value (ΞΌ N)

variable
  v : Value V
  w : Value W
```


Extract term from evidence that it is a value.
```
value : β {Ξ A} {V : Ξ β’ A}
  β (v : Value V)
    -------------
  β Ξ β’ A
value {V = V} v  =  V  
```


Renaming preserves values
(not needed, but I wanted to check that automatic generalisation works)
```
ren-val :
    (Ο : Ξ βα΄Ώ Ξ)
  β Value V
    ------------------
  β Value (ren Ο V)
-- ren-val Ο (Ζ N)    =  

ren-val Ο (Ζ N)     = Ζ (ren (renβ· Ο) N)
ren-val Ο `zero     = `zero
ren-val Ο (`suc M)  = `suc (ren-val Ο M)
ren-val Ο (ΞΌ M)     = ΞΌ (ren (renβ· Ο) M)
```

## Evaluation contexts

```
infix  6 [_]Β·_
infix  6 _Β·[_]
infix  7 `suc[_]
infix  7 case[_]
infix  9 _β¦_β§

data _β’_=>_ : Env β Type β Type β Set where

  β‘ : Ξ β’ C => C

  [_]Β·_ :
      Ξ β’ C => (A β B)
    β Ξ β’ A
      ---------------
    β Ξ β’ C => B

  _Β·[_] :
      {V : Ξ β’ A β B}
    β Value V
    β Ξ β’ C => A
      ----------------
    β Ξ β’ C => B

  `suc[_] :
      Ξ β’ C => `β
      -----------
    β Ξ β’ C => `β

  case[_] :
      Ξ β’ C => `β
    β Ξ β’ A
    β Ξ β· `β β’ A
      -----------
    β Ξ β’ C => A
```

The plug function inserts an expression into the hole of a frame.
```
_β¦_β§ :
    Ξ β’ A => B
  β Ξ β’ A
    ----------
  β Ξ β’ B
β‘ β¦ M β§                 =  M
([ E ]Β· M) β¦ L β§        =  E β¦ L β§ Β· M
(v Β·[ E ]) β¦ M β§        =  value v Β· E β¦ M β§
(`suc[ E ]) β¦ M β§       =  `suc (E β¦ M β§)
(case[ E ] M N) β¦ L β§   =  case (E β¦ L β§) M N
```

Composition of two frames
```
_β_ :
    Ξ β’ B => C
  β Ξ β’ A => B
    ----------
  β Ξ β’ A => C
β‘ β F                 =  F
([ E ]Β· M) β F        =  [ E β F ]Β· M
(v Β·[ E ]) β F        =  v Β·[ E β F ]
(`suc[ E ]) β F       =  `suc[ E β F ]
(case[ E ] M N) β F   =  case[ E β F ] M N
```

Composition and plugging
```
β-lemma :
    (E : Ξ β’ B => C)
  β (F : Ξ β’ A => B)
  β (P : Ξ β’ A)
    -----------------------------
  β E β¦ F β¦ P β§ β§ β‘ (E β F) β¦ P β§
β-lemma β‘ F P                                         =  refl
β-lemma ([ E ]Β· M) F P         rewrite β-lemma E F P  =  refl
β-lemma (v Β·[ E ]) F P         rewrite β-lemma E F P  =  refl
β-lemma (`suc[ E ]) F P        rewrite β-lemma E F P  =  refl
β-lemma (case[ E ] M N) F P    rewrite β-lemma E F P  =  refl
```

## Reduction

```
infix 2 _β¦_ _ββ_

data _β¦_ : (Ξ β’ A) β (Ξ β’ A) β Set where

  Ξ²-Ζ :
      Value V
      --------------------
    β (Ζ N) Β· V β¦ N [ V ]

  Ξ²-zero :
      ------------------
      case `zero M N β¦ M

  Ξ²-suc :
      Value V
      ---------------------------
    β case (`suc V) M N β¦ N [ V ]

  ΞΌ-Β· :
     Value V
     -------------------------
   β (ΞΌ N) Β· V β¦ (N [ ΞΌ N ]) Β· V

  ΞΌ-case :
     -------------------------------------
     case (ΞΌ L) M N β¦ case (L [ ΞΌ L ]) M N

data _ββ_ : (Ξ β’ A) β (Ξ β’ A) β Set where

  ΞΎ-refl :
      {Mβ² Nβ² : Ξ β’ B}
    β (E : Ξ β’ A => B)
    β Mβ² β‘ E β¦ M β§
    β Nβ² β‘ E β¦ N β§
    β M β¦ N
      --------
    β Mβ² ββ Nβ²
```

Notation
```
pattern ΞΎ E MββN = ΞΎ-refl E refl refl MββN
```

## Reflexive and transitive closure of reduction

```
infix  1 begin_
infix  2 _ββ _
infixr 2 _βββ¨_β©_
infix  3 _β

data _ββ _ : Ξ β’ A β Ξ β’ A β Set where

  _β :
      (M : Ξ β’ A)
      -----------
    β M ββ  M

  _βββ¨_β©_ :
      (L : Ξ β’ A)
    β {M N : Ξ β’ A}
    β L ββ M
    β M ββ  N
      ---------
    β L ββ  N

begin_ : (M ββ  N) β (M ββ  N)
begin Mββ N = Mββ N
```

## Irreducible terms

Values are irreducible.  The auxiliary definition rearranges the
order of the arguments because it works better for Agda.  
```
value-irreducible : Value V β Β¬ (V ββ M)
value-irreducible v VββM  =  nope VββM v
  where
  nope : V ββ M β Value V β β₯
  nope (ΞΎ β‘ (Ξ²-Ζ v)) ()
  nope (ΞΎ `suc[ E ] VββM) (`suc w)  =  nope (ΞΎ E VββM) w
```

Variables are irreducible.
```
variable-irreducible :
    ------------
    Β¬ (` x ββ N)
variable-irreducible (ΞΎ β‘ ())
```

## Progress

Every term that is well typed and closed is either
blame or a value or takes a reduction step.

```
data Progress : (β β’ A) β Set where

  step :
      L ββ M
      ----------
    β Progress L

  done :
      Value V
      ----------
    β Progress V

progress :
    (M : β β’ A)
    -----------
  β Progress M

progress (Ζ N)                           =  done (Ζ N)
progress (L Β· M) with progress L
... | step (ΞΎ E Lβ¦Lβ²)                    =  step (ΞΎ ([ E ]Β· M) Lβ¦Lβ²)
... | done v with progress M
...     | step (ΞΎ E Mβ¦Mβ²)                =  step (ΞΎ (v Β·[ E ]) Mβ¦Mβ²)
...     | done w with v
...         | (Ζ N)                      =  step (ΞΎ β‘ (Ξ²-Ζ w))
...         | (ΞΌ N)                      =  step (ΞΎ β‘ (ΞΌ-Β· w))
progress `zero                           =  done `zero
progress (`suc M) with progress M
... | step (ΞΎ E Mβ¦Mβ²)                    =  step (ΞΎ (`suc[ E ]) Mβ¦Mβ²)
... | done v                             =  done (`suc v)
progress (case L M N) with progress L
... | step (ΞΎ E Lβ¦Lβ²)                    =  step (ΞΎ (case[ E ] M N) Lβ¦Lβ²)
... | done v with v
...     | `zero                          =  step (ΞΎ β‘ Ξ²-zero)
...     | (`suc v)                       =  step (ΞΎ β‘ (Ξ²-suc v))
...     | (ΞΌ N)                          =  step (ΞΎ β‘ ΞΌ-case)
progress (ΞΌ N)                           =  done (ΞΌ N)
```


## Evaluation

Gas is specified by a natural number:
```
record Gas : Set where
  constructor gas
  field
    amount : β
```
When our evaluator returns a term `N`, it will either give evidence that
`N` is a value, or indicate that blame occurred or it ran out of gas.
```
data Finished : (β β’ A) β Set where

   done :
       Value N
       ----------
     β Finished N

   out-of-gas :
       ----------
       Finished N
```
Given a term `L` of type `A`, the evaluator will, for some `N`, return
a reduction sequence from `L` to `N` and an indication of whether
reduction finished:
```
data Steps : β β’ A β Set where

  steps :
      L ββ  M
    β Finished M
      ----------
    β Steps L
```
The evaluator takes gas and a term and returns the corresponding steps:
```
eval :
    Gas
  β (L : β β’ A)
    -----------
  β Steps L
eval (gas zero) L          =  steps (L β) out-of-gas
eval (gas (suc m)) L
    with progress L
... | done v               =  steps (L β) (done v)
... | step {M = M} LββM
    with eval (gas m) M
... | steps Mββ N fin       =  steps (L βββ¨ LββM β© Mββ N) fin
```

# Example

Computing two plus two on naturals:
```agda
pattern two = `suc `suc `zero

pattern xβ² = ` S Z
pattern yβ² = ` Z
pattern pβ² = ` S S S Z
pattern xβ³ = ` Z
pattern yβ³ = ` S Z
pattern plus = ΞΌ Ζ Ζ (case xβ² yβ² (`suc (pβ² Β· xβ³ Β· yβ³)))
```

Next, a sample reduction demonstrating that two plus two is four:
```agda
_ : plus Β· two Β· two ββ  `suc `suc `suc `suc (`zero {β})
_ = begin
      plus Β· two Β· two
    βββ¨ ΞΎ ([ β‘ ]Β· two) (ΞΌ-Β· two) β©
      (Ζ (Ζ case yβ³ xβ³ (`suc (plus Β· xβ³ Β· yβ³)))) Β· two Β· two
    βββ¨ ΞΎ ([ β‘ ]Β· two) (Ξ²-Ζ two) β©
      (Ζ case two xβ³ (`suc (plus Β· xβ³ Β· yβ³))) Β· two
    βββ¨ ΞΎ β‘ (Ξ²-Ζ two) β©
      case two two (`suc (plus Β· xβ³ Β· two))
    βββ¨ ΞΎ β‘ (Ξ²-suc (`suc `zero)) β©
      `suc (plus Β· `suc `zero Β· two)
    βββ¨ ΞΎ `suc[ [ β‘ ]Β· two ] (ΞΌ-Β· (`suc `zero)) β©
      `suc ((Ζ (Ζ case yβ³ xβ³ (`suc (plus Β· xβ³ Β· yβ³)))) Β· `suc `zero Β· two)
    βββ¨ ΞΎ `suc[ [ β‘ ]Β· two ] (Ξ²-Ζ (`suc `zero)) β©
      `suc ((Ζ case (`suc `zero) xβ³ (`suc (plus Β· xβ³ Β· yβ³))) Β· two)
    βββ¨ ΞΎ `suc[ β‘ ] (Ξ²-Ζ two) β©
      `suc case (`suc `zero) two (`suc (plus Β· xβ³ Β· two))
    βββ¨ ΞΎ `suc[ β‘ ] (Ξ²-suc `zero) β©
      `suc (`suc (plus Β· `zero Β· two))
    βββ¨ ΞΎ `suc[ `suc[ [ β‘ ]Β· two ] ] (ΞΌ-Β· `zero) β©
      `suc (`suc ((Ζ (Ζ case yβ³ xβ³ (`suc (plus Β· xβ³ Β· yβ³)))) Β· `zero Β· two))
    βββ¨ ΞΎ `suc[ `suc[ [ β‘ ]Β· two ] ] (Ξ²-Ζ `zero) β©
      `suc (`suc ((Ζ case `zero xβ³ (`suc (plus Β· xβ³ Β· yβ³))) Β· two))
    βββ¨ ΞΎ `suc[ `suc[ β‘ ] ] (Ξ²-Ζ two) β©
      `suc (`suc case `zero two (`suc (plus Β· xβ³ Β· two)))
    βββ¨ ΞΎ `suc[ `suc[ β‘ ] ] Ξ²-zero β©
      `suc (`suc two)
    β
```
