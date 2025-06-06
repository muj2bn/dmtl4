```lean
import Mathlib.Algebra.Group.Defs
```

<!-- toc -->

# Groups

Advanced mathematics is to a considerable extent about
generalizing from numbers and operations on them to a
diversity of abstract objects and analogous operations
on them.

As an example, we could imagine a set of objects, let's
called them the *rotations of a clock hand.* A rotation is
an action involving a clock hand moving from one position
to another on the clock dial (e.g., from 3PM to 5PM, which
we could say is a rotation by two hours).

There's a zero rotation, which is no rotation at all. Any
two rotations can be added. Addition is associative (just
think about it). Every rotation has an inverse, which is
rotation by the same amount in the opposite direction. And
under these definitions, any rotation added to its inverse
puts the hand of the clock where it started, so the sum
of any rotation and its inverse is the zero rotation.

Any set of objects with these properties satisfies the
rules required to form what is called a *group*. A group
is any *mathematical structure* having several elements:

- a set of objects, sometimes called the *carrier set*
- an associative *binary operation*, such as addition
- an element designated as the *zero* or *identity* element
- an inverse operation on elements of the set

These objects are tied together by additional rules:

- the binary operation must be associative
- adding zero on the left or right has no effect
- the sum of any element and its inverse must be zero

What we have, then, in the notion of a group is a very
general structure that we can impose on many different
kinds of objects. Rotations of a clock hand can serve as
group elements. The rationals form a group. The set of
invertible matrices forms a group. The notion of a group
gives us as way to study any such structure in common
terms.

Moreover, in everyday algebra we have not only groups.
which have but one operation (typically an addition or
multiplication operator), but a whole zoo of abstract
mathematical structures, each having special values
(such as the zero group element), operations (such as
addition), and axioms, which ensure that everything
works as required (addition is associative, etc).

Now having seen how we can embed proposition logic,
predicate logic, and set theory in Lean, we turn to
the definition of more abstract mathematics, namely
basic *abstract algebra*, in Lean.

## A Robot Vacuum

While abstract algebra sounds, well, abstract, it has
immediate practical applications. Imaging we have the
job to design a robot vacuum cleaner. One of the things
it has to do is rotate in place before heading off in a
new direction.

In this chapter we will develop an algebraic structure
called a torsor, comprising rotational actions that turn
a robot from one orientation to another. We will cast the
the rotations of a robot as a group of *actions* that in
turn change the robot's *orientations* (the directions it
points in).

To keep things simple, let's that there are only three
directions, indicated by the vertices of an equilateral
triangle, initially pointing "north." In this setting,
we can also imaging a rotation that, when *applied* to
an orientation, changes it to another orientation. We
can imagine a *unit* orientation that rotates the robot
by just one orientation going counter-clockwise.

Just as before, we can now think of rotations as a group.
What are the orientations? Well, they aren't a group, as
it makes no sense to add or multiply orientations in the
way we can do with rotations. On the other hand, we can
take the difference of two orientations defined to be the
rotation that makes the robot turn from the first to the
second orientation.

When we have a set of actions operating on a set of points
(here orientations), following certain rules, one has what
in geometry is called a torsor. In this chapter, we will
thus introduce the formalization of abstract algebra in
Lean through the simple example of a torsor with rotations
as *actions,* and orientations as *points*, with actions
acting on points by rotating them around in a circle with
three designated positions.

What we'll have by the end of this chapter is the algebraic
architecture of a robot that can only rotate (and only to a
few fixed orientations), but not one that can move in straight
lines. We'll fix that in the next chapter by defining a new
torsor, where the actions are *linear* translations, and the
points are all of the points of an n-dimensional linear space.
such as a line, plane, or 3-D Euclidean space.

```lean
namespace DMT1.Lecture.classes.groups

#check Zero
#check AddZeroClass
#check AddSemigroup
#check AddMonoid
```

## Overloaded Definitions

There is a common structure to every group, but the details
vary wildly from one to another. A group of rotations is not
the same as a group of invertible matrices representing, say,
translations in 3-D Euclidean space. Addition of rotations will
be different that addition of translations.

We thus cannot use *parametric* polymorphism to factor out the
common group structure, because that notion rests on the idea
of one definition--of an operation, for example---for arguments
of *any* type, but that clearly won't work here. What we need is
a new, but weaker, notion of polymorphism: one that allows us to
define the *same* operation (say, addition, +) for objects with
entirely different structures.

The idea is known as *overloading*, or *ad hoc* polymorphism.
We actually use it all the time in everyday programming. Java,
for example, defines + for both *int* and *float* types, but
there is not a single definition of +, Rather, the compiler
looks at an expression, such sa *x + y*, decides what types
*x* and *y* have, and then selected a particular definition
(implementation) of *+* depending: using the floating point
addition operation if they're floats, and int addition if they
are ints.

The key idea with *ad hoc polymorphism* is that we associate
different *implementations* of the same general *structure*
(here just a + operation) with different *types* of values to
be operated upon. Our plan is to generalize algebraic structures
to isolate their common elements, and then *overload* each of
the elements for different *types* of group elements.

In other words, for any group, we will have to overload (give
group-specific implementations of) (1) the type of the group
elements (e.g., rotations, tranlations, or rational numbers);
(2) the definition of addition for the particular element type;
(3) the definition of the *zero* object of that type; along with
proofs that, under these definitions, all of the rules hold. As
an example, we'll need proofs that the addition operator, as it
is defined, is associative.

That's the big idea. We will formalize algebraic concepts as
overloadable, or ad hoc polymorphic, structure types in Lean.
You will now progress from foundations in basic logic, then set
theory, to abstract algebra in Lean, resting on this new idea,
of overloadable structure types. In Lean, as in Haskell where
they were first introduced, they are called typeclasses. You must
now read standard material on typeclasses in Lean. See assignment.

The types of objects for which we will overload definitions in
this chapter are *rotations* and *orientations*. We will define
a group structure on rotations, and what is called a *torsor*
structure on *orientations*, where groups elements (rotations)
*act* on orientation points by rotating them around a circle.


## Rotations

We start by defining a rotation type and ordinary
operations on this type, not yet having anything to
do with a generic group structure. We start with three
rotations: by 0, 120, and 240 degrees, respectively.

```lean
inductive Rot : Type where | r0 | r120 | r240
open Rot
```

We define an addition operation on rotations, as an
ordinary function in Lean, so they add up in a way that
makes physical sense. The Lean function definition just
formalizes the contents of an addition table.

```lean
def addRot : Rot → Rot → Rot
| r0, r => r
| r, r0 => r
| r120, r120 => r240
| r240, r120 =>  r0
| r120, r240 => r0
| r240, r240 => r120
```

As of Right now we have no definition of a *+* notation
for objects of this type. If you uncomment the next line
of code, you will see that Lean indicates that there is no
such operator.
```lean
--#eval r0 + r120
```

What we'd like to do now is to *overload* the *+* operator
so that it will work with our *Rot* objects. To do this we
will create an *instance* a *typeclass* in Lean. A typeclass
is just an ordinary data (structure) type with one or more
fields to be overloaded.

The particular typeclass we need to instantiate is *Add α*
in Lean, where α is the type of objects for which *+* is to
be overloaded. It has just a single field, *add* to which one
assigns whatever function will perform addition for objects
of our *Rot* type: and that would be our *addRot* function.

Here's the definition of the *Add* typeclass in Lean, followed
by our definition of an instance of this class for our *Rot*
type.

```lean
class Add (α : Type u) where
  add : α → α → α
```

A *class* is just like a *structure* type in Lean, but with
added typeclass machinery what we can now explain. For that,
here's our instance definition.

```lean
instance : Add Rot := { add := addRot }
```

Our instance definition defines an instance of the *Add* class
viewed as an ordinary structure type. But because it's a class,
not just a structure, Lean *registers* this definition in its
internal registry of typeclass instances. When a definition of
addition for *Rot* objects is needed, Lean searches its database
for an instance for the *Rot* type (or for instances from which
such an instance can be *synthesized*). If no instance with a
suitable definition of *+* can be found, Lean issues an error;
otherwise it will use the definition of *+* stored as the value
of the *add* field in that instance.

An important rule of thumb when using Lean is that one should
ever have only one instance of a particular typeclass. Suppose
you had two instances of *Add Rot*. You would then have two
different definitions of *add* for *Rot*. Even if they are
assigned the same value, e.g., *addRot*, Lean will not know
they are the same without a proof of that. When you are using
typeclasses and proofs are failing mysteriously, look to the
possibility that you've got multiple instances.

Now, the *Add* typeclass provides an overloaded definition
of *add* for *Rot* objects. Under the hood, Lean actually
instantiates a second typeclass, *HAdd,*  for *heterogeneous*
addition (addition of possibly different types of objects).
It's this class that actually provides the definition of the
*+* notation* for the *Rot* type.

With all of this, we can not only add rotations using the
orginal function, *addRot r0 r120*, but using the *add*
field of the *instance* of the *Add Rot* typeclass that
Lean fetches when we write *Add.add r0 r120*. Moreover,
we can now also write *r0 + r120*. Lean will determine
that the *+* is *HAdd.hadd*, defined in turn to be just
*Add.add*, defined finally to be *addRot*.
```lean
#eval addRot r0 r120  -- no typeclass
#eval Add.add r0 r120 -- Finds Add instance for Rot uses Add.add
#eval r0 + r120       -- Same thing w/ notation from HAdd class
```



## Group Structure

To provide *Rot* with the structure of a group under an addition
operator, we overload the *AddGroup* typeclass. Whereas *Add* had
just one field, the *AddGroup* class has several. When we look to
its definition, we run into a second Lean concept that's new to us:
that one structure (including typeclasses) can *extend* another.


```lean
class AddGroup (A : Type u) extends SubNegMonoid A where
  protected neg_add_cancel : ∀ a : A, -a + a = 0
```

This definition explains that an *AddGroup* instance will contain
all of the fields of the *SubNegMonoid* structure, plus one new
field, called *neg_add_cancel*, holding a proof that, however all
the operations are defined, it will be the case that if you add a
group element and its (additive) inverse, the result will always
be the zero element of the group. To instantiate an *AddGroup* class
we have to provide values for all of these fields.

One way to see what fields need to be provided is to start an instance
definition and then use Lean's error reporting to see what's missing.
We want to define our rotations as a group, so let's see what we need.

Uncomment the following line then hover over the red underlined curly
brace.

```lean
-- instance : AddGroup Rot :=
-- {
-- }
```

Lean will tell you that you'll need to provide values for all of
the following fields.
- 'add_assoc'
- 'zero'
- 'zero_add'
- 'add_zero'
- 'nsmul'
- 'neg',
- 'zsmul',
- 'neg_add_cancel'

But what about the addition operator? Shouldn't the group typeclass
have a field where you define the addition operator? Yes, a group class
*does* need to, and does, have a field called *add*. So why isn't Lean
asking it, for *add*? That's because you have already registered an
instance of Add.add for *Rot*, and typeclass inference found it and
used the definition it provides to populate the add field. If you had
defined several other typeclasses, all the fields but that last one
would already be defined.

There are two development paths you can take here. One is to follow
the `*extends* hierarchy and define and register instances, bottom-up.
By the time you get back here to *AddGroup*, instances will be found
that provide definitions for all of the needed fields but for the one
new field: *neg_add_cancel*.


TODO: A diagram of the inheritance structure is meant to go here.

-- ```mermaid
-- classDiagram
--     class Add {
--       add : α → α → α
--     }
--     class Zero {
--       zero : α
--     }
--     class Neg {
--       neg : α → α
--     }
--     class Sub {
--       sub : α → α → α
--     }

--     class AddSemigroup {
--       add_assoc : ∀ (a b c : G), (a + b) + c = a + (b + c)
--     }
--     class AddCommMagma {
--       add_comm : ∀ (a b : G), a + b = b + a
--     }
--     class AddCommSemigroup {
--     }

--     class AddZeroClass {
--       zero_add : ∀ (a : M), 0 + a = a
--       add_zero : ∀ (a : M), a + 0 = a
--     }
--     class AddMonoid {
--       nsmul : ℕ → M → M
--       nsmul_zero : ∀ (x : M), nsmul 0 x = 0
--       nsmul_succ : ∀ (n : ℕ) (x : M), nsmul (n+1) x = nsmul n x + x
--     }
--     class AddCommMonoid

--     class SubNegMonoid {
--       sub_eq_add_neg : ∀ (a b : G), a - b = a + -b
--       zsmul : ℤ → G → G
--       zsmul_zero' : ∀ (a : G), zsmul 0 a = 0
--       zsmul_succ' : ∀ (n : ℕ) (a : G), zsmul (n.succ) a = zsmul n a + a
--       zsmul_neg' : ∀ (n : ℕ) (a : G), zsmul (-n.succ) a = -(zsmul (n.succ) a)
--     }
--     class AddGroup {
--       neg_add_cancel : ∀ (a : A), -a + a = 0
--     }
--     class AddCommGroup

--     Add <|-- AddSemigroup
--     Add <|-- AddCommMagma
--     AddSemigroup <|-- AddCommSemigroup
--     AddCommMagma <|-- AddCommSemigroup

--     Zero <|-- AddZeroClass
--     Add <|-- AddZeroClass

--     AddSemigroup <|-- AddMonoid
--     AddZeroClass <|-- AddMonoid

--     AddMonoid <|-- AddCommMonoid
--     AddCommSemigroup <|-- AddCommMonoid

--     AddMonoid <|-- SubNegMonoid
--     Neg <|-- SubNegMonoid
--     Sub <|-- SubNegMonoid

--     SubNegMonoid <|-- AddGroup

--     AddGroup <|-- AddCommGroup
--     AddCommMonoid <|-- AddCommGroup
--     AddCommMonoid <|-- AddCommGroup
-- ```

With that defined we have a group structure on *Rot*, and we can use
all of the concepts, intuitions, notations, and results common to all
groups, to work with objects of particular interest to us: here just
our rotations. There's a zero rotation, associative rotation addition
(+, from the integers), additive inverse (negation), and consequently,
via induction, both natural number and integer scalar multiplication.

Alternatively, you can just give values to all of the remaining fields
right here without defining or instantiating any of the dependencies.
The downside is that Lean will *not* infer definitions of those classes
a definition or instance of, here, the *AddGroup* class. If you should
ever need such an instance elsewhere in your system, it might be be a
good idea to define it, and its dependencies, explicitly and register
instances for all of them.

Definition and instantiation of the full tree of typeclasses maximally
enables the typeclass inference system to synthesize a broad variety
instances as might be needed downstream. In this section we will thus
take that route. We'll start by diagramming the tree and annotating the
nodes with the definitions provided at each one.



## AddMonoid
```
class AddMonoid (M : Type u) extends AddSemigroup M, AddZeroClass M where
  protected nsmul : ℕ → M → M
  protected nsmul_zero : ∀ x, nsmul 0 x = 0 := by intros; rfl
  protected nsmul_succ : ∀ (n : ℕ) (x), nsmul (n + 1) x = nsmul n x + x := by intros; rfl
```

```lean
#check AddGroup
#check AddSemigroup
#check SubNegMonoid
```

## Additive Semigroups
```
class AddSemigroup (G : Type u) extends Add G where
  protected add_assoc : ∀ a b c : G, a + b + c = a + (b + c)
```
### Rotation Addition is Associative

We will prove that rotation addition is associative
as a separate theorem now, and then will simply plug
that in to our new typeclass instance as the proof.

```lean
theorem rotAddAssoc : ∀ (a b c : Rot), a + b + c = a + (b + c) :=
by
    intro a b c
    cases a
    -- a = ro
    {
      cases b
      -- b = r0
      {
        sorry
      }
      -- b = r120
      {
        sorry
      }
      -- b = 240
      {
        sorry
      }
    }
    -- a = r120
    {
      sorry
    }
    -- a = r240
    {
      sorry
    }
```

## AddSemigroup Rot
Now we can augment the Rot type with the
structure of a mathematical semigroup. All
that means, again, is that (1) there is an
addition operation, and (2) it's associative.


```lean
instance : AddSemigroup Rot :=
{
  add_assoc := rotAddAssoc
}
```

Next, on our path to augmenting the Rot type
with the structure of an additive monoid, we
also need to have AddZeroClass for Rot.

This
class will add the structure that overloads
a *zero* value and requires it to behave as
both a left and right identity (zero) for +.

```lean
#check AddZeroClass
```


## AddZeroClass Rot

Here's the AddZeroClass typeclass. It in turn
requires Zero and Add for Rot.
```
class AddZeroClass (M : Type u) extends Zero M, Add M where
  protected zero_add : ∀ a : M, 0 + a = a
  protected add_zero : ∀ a : M, a + 0 = a
```

It inherits from (extends) Zero and Add. Here's the Zero
class. It just defines a value to be known as *zero*, and
denoted as *0*.

## Zero Rot

```
class Zero (α : Type u) where
  zero : α
```

```lean
#check Zero

instance : Zero Rot := { zero := r0 }

#reduce (0 : Rot) -- 0 now notation for r0
```

## Rest of AddZeroClass

```lean
instance : AddZeroClass Rot :=
{
  zero_add :=
    by
      intro a
      cases a
      repeat rfl
  add_zero :=
    by
      intro a
      cases a
      repeat rfl
}
```

## Scalar Multiplication of Rot by Nat
We're almost prepared to add the structure of
a monoid on Rot. For that, we'll need to implement
a *natural number scalar multiplication operator*
for Rot.
```lean
def nsmulRot : Nat → Rot → Rot
| 0, _ => 0   -- Note use of 0 as notation for r0
| (n' + 1), r => nsmulRot n' r + r
```

## Voila, Monoid Rot

And voila, we add the structure of an additive
monoid to objects of type Rot: an AddMonoid α.
Fortunately, Lean provides default implementation
that we \use, implemeting natural number scalar
multiplication as n-iterated addition.
```lean
instance : AddMonoid Rot :=
{
  -- Mathlib: Multiplication by a natural number.
  -- Set this to nsmulRec unless Module diamonds
  -- are possible.
  nsmul := nsmulRot
}
```

Note that the *nsmul_zero* and *nsmul_succ* fields
have default values, proving (if possible) that *nsmul*
behaves properly. Namely, scalar multiplication by the
natural number, *0*, returns the *0* rotation, and that
scalar multiplication is just iterated addition.

With this complete definition AddMonoid for *Rot*, we
have gained a *scalar multiplication by natural number*
operation, with concrete notation, • (enter as *\smul*).

The result is an algebraic structure with *Rot* as the
carrier group, and with addition (*+*), zero (*0*), and
scalar multiplication (•) operations.

```lean
#eval (0 : Rot)   -- zero
#eval r120 + 0    -- addition
#eval! 2 • r240   -- scalar mult by ℕ
```



## Additive Groups

```lean
#check AddGroup
```

## AddGroup Rot
```
class AddGroup (A : Type u) extends SubNegMonoid A where
  protected neg_add_cancel : ∀ a : A, -a + a = 0
```

To impose a group structure on *Rot*, we will need first
to impose the structure of a *SubNegMonoid* and then add
a proof that *-a + a* is *0* for any *a*. Instantiating
*SubNegMonoid* in turn will require *AddMonoid* (which we
already have), *Neg*, and *Sub* instances to be defined,
as seen next.

```lean
#check SubNegMonoid
```
Note that most fields of this structure have default values.

```
class SubNegMonoid (G : Type u) extends AddMonoid G, Neg G, Sub G where
  protected sub := SubNegMonoid.sub'
  protected sub_eq_add_neg : ∀ a b : G, a - b = a + -b := by intros; rfl
  /-- Multiplication by an integer.
  Set this to `zsmulRec` unless `Module` diamonds are possible. -/
  protected zsmul : ℤ → G → G
  protected zsmul_zero' : ∀ a : G, zsmul 0 a = 0 := by intros; rfl
  protected zsmul_succ' (n : ℕ) (a : G) :
      zsmul n.succ a = zsmul n a + a := by
    intros; rfl
  protected zsmul_neg' (n : ℕ) (a : G) : zsmul (Int.negSucc n) a = -zsmul n.succ a := by
    intros; rfl
  ```

```lean
-- EXERCISE: Instantiate SubNegMonoid, thus also Neg and Sub
```

```lean
class Neg (α : Type u) where
  neg : α → α
```

```lean
def negRot : Rot → Rot
| 0 => 0
| r120 => r240
| r240 => r120

instance : Neg Rot :=
{
  neg := negRot
}
```

```lean
class Sub (α : Type u) where
  sub : α → α → α
```

```lean
instance : Sub Rot :=
{
  sub := λ a b => a + -b
}


instance : SubNegMonoid Rot :=
{
  zsmul := zsmulRec
}

  -- EXERCISE: Instantiate AddGroup

theorem rotNegAddCancel :  ∀ r : Rot, -r + r = 0 :=
by
  intro r
  cases r
  rfl
  rfl
  rfl

instance : AddGroup Rot :=
{
  neg_add_cancel := rotNegAddCancel
}
```

## Example: A Rotation Group

We have succeeded in establishing that the rotational
symmetries of an equilateral triangle for an additive
group. Th


```lean
def aRot := r120                  -- rotations
def zeroRot := r0                 -- zero element
def aRotInv := -r120              -- inverse
def aRotPlus := r120 + r240       -- addition
def aRotMinus := r120 - r240
def aRotInvTimesNat := 2 • r120   -- scalar mul by ℕ
def aRotInvTimeInt := -2 • r120   -- scalar mul by ℤ
```

Question: Can we define scalar multiplication by reals
or rationals?


## Constraints on Type Arguments

```lean
-- uncomment to see error
-- def myAdd {α : Type u} : α → α → α
-- | a1, a2 => a1 + a2

def myAdd {α : Type u} [Add α] : α → α → α
| a, b => a + b
```

By requiring that there be a typeclass instance
for α in *myAdd* we've constrained the function to
take and accept only those type for which this is
the case. We could read this definition as saying,
"Let myAdd by a function polymorphic *in any type for
which + is defined*, such that myAdd a b = a + b."

What we have thus defined is a polymorphic function,
but one that applies only to values of type for which
some additional information is defined, here, how to
add elements of a given type.

```lean
#eval myAdd 1 2                   -- Nat
#eval myAdd r120 r240             -- Rot
```

We cannot use *myAdd* at the moment to add
strings, because there's no definition of the
Add typeclass for the String type.
```lean
-- uncomment to see error
-- #eval myAdd "Hello, " "Lean"   -- String (nope)
```

That problem can be fixed by creating an
instance of Add for the String type, where
we use String.append to implement add (+).
With that we can now apply *myAdd* to String
arguments as well. We have to define a special
case *Add* typeclass instance for each type we
want to be addable using *Add.add*, namely *+*.
```lean
instance : Add String :=
{
  add := String.append
}
#eval myAdd "Hello, " "Lean"


 end DMT1.Lecture.classes.groups
```
