---
title: Reasoning about Caml Functions
---

The [computer science department](http://www.cs.umn.edu/) at the
University of Minnesota has recently introduced a new course titled
"Advanced Programming Principles". Generally, the course is focused on
teaching students how to use modern programming languages (i.e.,
typed, type-safe, functional) in a productive and principled way.

A staple of functional programming is recursion. Recursive functions
are ones that are defined in terms of themselves. Typically, recursive
functions map arguments to a set of base cases that relate arguments
to initial values (values not relying on self-reference) and a set of
recursive cases that relate arguments to self-referencing
expressions. The canonical example of a recursive function is the one
used to computed the
[Fibonacci sequence](http://en.wikipedia.org/wiki/Fibonacci_number).

Reasoning about recursive functions can often be done using
induction. Inductive reasoning requires that one show a property holds
in all base cases and all recursive cases. The latter arguments may
rely on a hypothesis that the property holds for all function
applications on "smaller" inputs. The notion of what "smaller" is can
be complicated, however in recursive functions acting on finite data
structures, "smaller" usually refers to some measure of input data
structure size. Referring back to Fibonacci example, recursive
function applications in a recursive definition of the Fibonacci
sequence always act on arguments less than the defined input
arguments. Therefore, the notion of "smaller" could be $<$ on
natural numbers for such definitions of the Fibonacci sequence.

In what follows, I will inductively reason about a property of a
recursive function written in [Caml](http://caml.inria.fr/). Many
similar exercises can be found elsewhere on the internet, in academic
papers, and textbooks. What may be interesting to some is that I
follow informal reasoning with formal reasoning done in
[Coq](https://coq.inria.fr/) using a shallow embedding. Shallow
embeddings rely heavily on the meaning of the target language
semantics (i.e., the language we are embedded an object language in,
in this case it is Coq) to capture the meaning of the object langangue
(i.e., the language under study, in this case Caml) where deep
embeddings rely heavily on a definition of the object language
semantics within the target language.

# Order Preserving List Insertion

Two functions will be considered for this exercise: a boolean function
that returns true when an input list of integers is in ascending order
and a function that places an integer in the list and yields a new
list.

```ocaml
  let rec sorted l = match l with
  | [ ] -> true
  | x::[] -> true
  | x1::x2::xs -> x1 <= x2 && sorted (x2::xs)
```

```ocaml
  let rec place e l = match l with
  | [ ] -> [e]
  | x::xs -> if e < x then e::x::xs else x :: (place e xs)
```


I would like to show that for all sorted integer lists $xs$ and
integers $x$ that the evaluation result of applying place to $x$ and
$xs$ is a sorted list. Showing this property requires some
understanding of what it means to evaluate Caml expressions. I will
not go into this here, but one may look
[here](http://www.cs.princeton.edu/~dpw/courses/cos326-12/notes/evaluation.php)
for some intuition about Caml evalution semantics.

# Informal Proof

I would like to show that for all possible values of `l : int
list`{:.language-ocaml} and `e : int`{:.language-ocaml}, if `sorted
l`{:.language-ocaml} evaluates to `true`{:.language-ocaml} then
`sorted (place el)`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}. Notice here I am confusing the written words
and fragments of Ocaml. My intention is to make reading this informal
proof more intuitive.

The obvious way to try to prove this problem is by induction on the
size of `l`{:.language-ocaml} . When considering a non-empty
`l`{:.language-ocaml} , the case when `e>=m`{:.language-ocaml} must be
considered and it must be shown that `sorted (m::(place e n))`{:.language-ocaml} where
`l = m::n`{:.language-ocaml}. There are two issues here:

1. the head of the list resulting from evaluating `place e
   n`{:.language-ocaml} must be extracted due to the third pattern in
   `match`{:.language-ocaml} , this value may be greater, equal, or
   less than `e`{:.language-ocaml} and will require further case analysis

2. `sorted (place e n)`{:.language-ocaml} will be the same size as
   `l`{:.language-ocaml} so the induction hypothesis cannot be
   applied. One might unfold the definition again, consider both case
   of issue 1, and then the inductive hypothesis can be
   applied. However, a variant of issue 1 will remain.

The proof that will be presented avoids these issues through a lemma.

## *Lemma*: for all `n : int list`{:.language-ocaml}, `m : int`{:.language-ocaml}, and `e : int`{:.language-ocaml} if `sorted (m::n)`{:.language-ocaml} evaluates to `true`{:.language-ocaml} and `e>=m`{:.language-ocaml} then `sorted (m::(place e n)`{:.language-ocaml} evaluates to `true`{:.language-ocaml}.

Let `n`{:.language-ocaml} be an arbitrary integer list such that
`sorted (m::n)`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}. Let `e`{:.language-ocaml} and
`m`{:.language-ocaml} be arbitrary integers such that
`e>=m`{:.language-ocaml}. It then must be shown that `sorted
(m::(place e n))`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}. We will show this by induction on the size
of `n`{:.language-ocaml}.

Case analysis of data structures of type `int list`{:.language-ocaml}
yields two case; one where `n`{:.language-ocaml} is empty and one
where `n`{:.language-ocaml} is not. In the case where
`n`{:.language-ocaml} is empty it must be shown `sorted (m::(place e
[]))`{:.language-ocaml} evaluates to `true`{:.language-ocaml}. The
expression `place e []`{:.language-ocaml} may be replaced with
`[e]`{:.language-ocaml} by evaluation. Therefore, it must be shown
that `sorted (m::e::[])`{:.language-ocaml} evaluates to to
`true`{:.language-ocaml}. This expression may be replace with `m<=e &&
sorted [e]`{:.language-ocaml}, then `m<=e && true`{:.language-ocaml}
by evaluation. Therefore, it must be shown that `m<=e &&
true`{:.language-ocaml} evaluates to `true`{:.language-ocaml} which
evaluation and assumptions yield.

In the case where `n`{:.language-ocaml} is not empty we must show
`sorted (m::(place e n))`{:.language-ocaml} evaluates to
`true`{:.language-ocaml} where `n`{:.language-ocaml} is equal to
`o::p`{:.language-ocaml}, `o`{:.language-ocaml} is an integer and
`p`{:.language-ocaml} is an integer list. Evaluation and case analysis
of `place`{:.language-ocaml} yields that `place e n`{:.language-ocaml}
may be replace with either `e::o::p`{:.language-ocaml} assuming
`e<o`{:.language-ocaml} or `o::(place e p)`{:.language-ocaml} assuming
`e>=o`{:.language-ocaml}.

When replacing with `e::o::p`{:.language-ocaml} and assuming
`e<o`{:.language-ocaml} it must be shown that `sorted
(m::e::o::p)`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}. The expression may be replaced with `m<=e &&
e<=o && sorted (o::p)`{:.language-ocaml} by evaluation. Evaluation,
`e>=m`{:.language-ocaml}, and `e<o`{:.language-ocaml} yields it
suffices to show `true && sorted (o::p)`{:.language-ocaml} evaluates
to `true`{:.language-ocaml}.  Evaluation of the assumption `sorted
(m::o::p)`{:.language-ocaml} yields `m<=o && sorted
(o::p)`{:.language-ocaml} evaluates to `true`{:.language-ocaml} which
yields `true && sorted (o::p)`{:.language-ocaml} evaluates to
`true`{:.language-ocaml} immediately.

When replacing with `o::(place e p)`{:.language-ocaml} and assuming
`e>=o`{:.language-ocaml} it must be shown that `sorted (m::o::(place e
p))`{:.language-ocaml} evaluates to `true`{:.language-ocaml}. It
suffices to show `m<=o && sorted (o::(place e p))`{:.language-ocaml}
evaluates to `true`{:.language-ocaml}. The inductive hypothesis, for
all `l : int list`{:.language-ocaml}, `m : int`{:.language-ocaml}, and
`e : int`{:.language-ocaml} if `sorted (m::l)`{:.language-ocaml}
evaluates to `true`{:.language-ocaml} and `e>=m`{:.language-ocaml} and
`l`{:.language-ocaml} is smaller than `n`{:.language-ocaml} then
`sorted (m::(place e l)`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}, can be used to show that `sorted (o::(place
e p))`{:.language-ocaml} evaluates to `true`{:.language-ocaml};
`p`{:.language-ocaml} is shorter than `n`{:.language-ocaml}, `sorted
(o::p)`{:.language-ocaml} evaluates to `true`{:.language-ocaml} by
evaluation of the assumption `sorted (m::o::p)`{:.language-ocaml}, and
`e>=o`{:.language-ocaml} by assumption.

&#x25a0;

## *Theorem*: for  `l : int list`{:.language-ocaml} and `e : int`{:.language-ocaml}, if `sorted l`{:.language-ocaml} evaluates to `true`{:.language-ocaml} then `sorted (place e l)`{:.language-ocaml} evaluates to `true`{:.language-ocaml}.

Let `l`{:.language-ocaml} be an aribitrary integer list such that
`sorted l`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}. Let `e`{:.language-ocaml} be an arbitrary
integer. It then must be shown that `sorted (place e
l)`{:.language-ocaml} evaluates to `true`{:.language-ocaml}. This will
be shown by case analysis on the forms of integer lists.

In the case where `l`{:.language-ocaml} is empty it must be shown
`sorted (place e [])`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}. The expression `place e
[]`{:.language-ocaml} may be replaced with `[e]`{:.language-ocaml} by
evaluation. Therefore, it must be shown that `sorted
[e]`{:.language-ocaml} evaluates to to `true`{:.language-ocaml}. This
expression immediately evaluates to `true`{:.language-ocaml}.

In the case where `l`{:.language-ocaml} is not empty it must be shown
that `sorted (place e (m::n))`{:.language-ocaml} evaluates to
`true`{:.language-ocaml} where `l = m::n`{:.language-ocaml}.  Analysis
of `place`{:.language-ocaml} yields that we must consider two
sub-cases; when `e<m`{:.language-ocaml} or
`e>=m`{:.language-ocaml}. When `e<m`{:.language-ocaml}, `place e
(m::n)`{:.language-ocaml} evaluates to `e::m::n`{:.language-ocaml} and
it must be shown that `sorted e::m::n`{:.language-ocaml} evaluates to
`true`{:.language-ocaml}.  This expression may be replace with `e<m &&
sorted m::n`{:.language-ocaml} by evaluation and this evaluates to
`true`{:.language-ocaml} by our assumptions. When
`e>=m`{:.language-ocaml}, `place e (m::n)`{:.language-ocaml} evaluates
to `m::(place e n)`{:.language-ocaml} and it must be shown that
`sorted (m::(place e n)`{:.language-ocaml} evaluates to
`true`{:.language-ocaml} which may be proved using the lemma above.

# Formal Proof

The property we are trying to prove is simple. However, tracking
assumptions, cases, and the constraints they place on quantified terms
is error prone when done informally. Furthermore, much of the informal
proof relies on evaluating Caml terms; something that is best done by
computers. Therefore, we will use the Coq to verify that our informal
proofs is indeed a correct proof. However, we will use the structure
of the informal proof when constructing this formal proof in
Coq. Specifically, we will see that the informal lemma proved above
will be useful in our formal proof.

Coq is a
[proof assistant](http://en.wikipedia.org/wiki/Proof_assistant) built
on the
[Calculus of Construction](http://en.wikipedia.org/wiki/Calculus_of_constructions).
Practically speaking, it provides a dependently typed functional
programming language. Due to the
[Curry-Howard correspondence](http://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
typed languages can be interpreted as proof systems where types in the
language are formulas and terms in the language are proofs. What
follows assumes a familiarity with Coq that can be gained through the
many Coq tutorials on the internet. The Coq code is intermixed with
discussion. However, to use the following code, it must be entered
into Coq in the order it appears.

Using proof assistants can be tedious because fundamental theorems
must be proved before they can be used. For instance, all basic
mathematical properties used in the informal proof (e.g., inequalities
on natural numbers) must be proved before they can be
utilized. Fortunately, Coq provides many theories that can be used as
libraries.

```coq

Require Import Coq.Bool.Bool.
Require Import Coq.Arith.EqNat.
Require Import Coq.Lists.List.
```

Definitions in terms of booleans are not given for the less than and
less than or equal to relations.

```coq
Fixpoint blt_nat (m n : nat) : bool :=
  match m with
    | O => match n with
             | O => false
             | _ => true
           end
    | S m => match n with
               | S n => blt_nat m n
               | _ => false
             end
  end.

Definition ble_nat (m n: nat) := orb (blt_nat m n) (beq_nat m n).
```

The first definition gives the meaning for less than on natural
numbers.  This syntax should look fairly similar to Caml syntax. The
definition is inductive:

1. Zero is less than every natural number with exception to itself.
2. A natural number is less than another if it's predecessor (the
   result of subtracting one) is less than the predecessor of the
   other.


With these definitions,
[trichotomy](http://en.wikipedia.org/wiki/Trichotomy_%28mathematics%29)
can now be established for inequalities on the natural numbers. This
result will be crucial when reasoning about how placement of an
element in list will impact it's sortedness. Trichotomy is proved
through use of the `sym_blt_false_beq_true`{:.language-coq} lemma.

```coq

Lemma sym_blt_false_beq_true:
    forall (m n:nat),
      blt_nat m n = false ->
      blt_nat n m = false ->
      beq_nat m n = true.
intros m.
induction m.
    intros.
    destruct n.
      auto.
      simpl in H.
      inversion H.
   intros.
   simpl in *.
   destruct n.
     simpl in H0.
     inversion H0.
     simpl in H0.
     apply (IHm n H H0).
Qed.

Theorem trichotomy_nat:
    forall (m n:nat),
      blt_nat m n = true \/
      beq_nat m n = true \/
      blt_nat n m = true.
intros.
destruct (blt_nat m n) eqn:fst.
   left.
   auto.
   right.
   destruct (blt_nat n m) eqn:thd.
     right.
     auto.
     left.
     eapply (sym_blt_false_beq_true m n fst thd).
Qed.
```

From trichotomy it may be proved that the less than relation implies
the less than or equal to relation.

```coq

Lemma blt_imp_ble :
    forall (m n:nat),
      blt_nat m n = false -> ble_nat n m = true.
intros.
unfold ble_nat.
remember (trichotomy_nat n m).
case o.
  intros.
  rewrite H0.
  auto.
  intros.
  case H0.
    intros.
    rewrite H1.
    rewrite orb_comm.
    auto.
    intros.
    rewrite H1 in H.
    inversion H.
Qed.
```

At this point there is enough of a theory to make an attempt on the
final result. Therefore,a shallow embedding of the `place`{:.language-coq} and
`sorted`{:.language-coq} definitions are given in Coq. These definitions rely on the
built in polymorphic list type in Coq. Ignoring syntactic differences,
their definitions are very similar to the ones given in
Caml. Furthermore, due to the semantics of Coq the results in this
write up will assume that the Caml and Coq implementations of these
functions are semantically equivalent (something that should actually
be proved).

```
Fixpoint place (e : nat) (l : list nat) :=
  match l with 
    | nil => e :: nil
    | x :: xs => if blt_nat e x 
                   then e :: l  
                   else  x :: (place e xs)
  end.

Fixpoint sorted l :=
match l with
| nil => true
| x1 :: l'  => 
  match l' with
  | nil => true
  | x2 :: xs =>
    andb (ble_nat x1 x2) (sorted l')
  end
end.
```

Now it can be proved that the tail of a sorted list is sorted.  In the
informal proof, it was easy to see that by evaluation the tail of a
sorted list is sorted. In Coq, we must do a bit more work therefore,
it is captured in a lemma.

```coq

Lemma sorted_is_recursive:
  forall (a:nat) (l:list nat),
    sorted (cons a l) = true -> sorted l = true.
intros.
simpl in H.
destruct l eqn:Hl.
  auto.
  apply andb_true_iff in H.
  decompose [and] H.
  auto.
Qed.
```

Now we may prove the lemma we used in the informal argument.

```coq
Lemma lemma : forall (l:list nat) (a e:nat), 
sorted (a :: l) = true -> 
ble_nat a e = true -> 
sorted (a :: place e l) = true.
induction l.
  intros.
  simpl.
  rewrite H0.
  auto.
  intros.
  simpl.
  destruct (blt_nat e a) eqn:test.
    rewrite H0.
    simpl.
    unfold ble_nat.
    rewrite test.
    simpl.
    simpl in H.
    apply andb_true_iff in H.
    decompose [and] H.
    auto.
    assert (H':=H).
    apply sorted_is_recursive in H.
    apply blt_imp_ble in test.
    assert (IR:=IHl a e H test).
    rewrite IR.
    simpl in H'.
    apply andb_true_iff in H'.
    decompose [and] H'.
    rewrite  H1.
    auto.
Qed.
```

As it was in the informal proof, this lemma yields the theorem.

```coq
Theorem theorem :
    forall (l:list nat) (e : nat), 
      sorted l = true -> sorted (place e l) = true.
intros.
destruct l.
  auto.
  simpl.
  destruct (blt_nat e n) eqn:elta.
    simpl.
    unfold ble_nat.
    rewrite elta.
    simpl.
    simpl in H.
    auto.
    apply blt_imp_ble in elta.
    remember (lemma l n e H elta).
    auto.
Qed.
```

# Comments

<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
    var disqus_shortname = 'chaosape'; // Required - Replace example with your forum shortname

    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
