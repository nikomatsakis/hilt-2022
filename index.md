class: center
name: title
count: false

# a MIR Formality

.p60[![Ferris](./images/ferris.svg)]

.me[.grey[*by* **Nicholas Matsakis**]]
.left[.citation[View slides at `https://nikomatsakis.github.io/hilt-2022/`]]

---

What is...

# a MIR Formality

?

---

# From the repository...

> This repository is an **early-stage experimental project** that aims to be a **complete, authoritative formal model** of the Rust MIR. **Presuming** these experiments bear fruit, the intention is to bring this model into Rust as an RFC and develop it as an **official part of the language definition**.

(emphasis mine)

---

What is...

# a MIR Formality

* A **definition** for the Rust type system (and extensions thereof)
* A **prototype** for how to implement it

---

What is...

# a MIR Formality

More concretely:

* implemented in PLT Redex<sup>1</sup>
* we have a compiler front-end to parse and generate tests
* available at https://github.com/nikomatsakis/a-mir-formality

.footnote[<sup>1</sup> More on this later.]

---

# Example in Rust

```rust
trait Debug {...}

impl<T: Debug> Debug for Data<T> {...}

struct Data<T> { data: T }
```

---

# Example in PLT Redex

```scheme
(crate A {
    ; trait Debug { }
    (trait Debug[] where [] { })

    ; impl<T: Debug> Debug for Data<T>
    (impl[(type T)] Debug[] for (Data < T >) where [(T : Debug)] {})

    ; struct Data<T> { }
    (struct Data[(type T)] where [] { (data : T)})
})
```

---

# Automated generation

```bash
> cargo run -- --print input.rs
(crate A {
    ; trait Debug { }
    (trait Debug[] where [] { })

    ; impl<T: Debug> Debug for Data<T>
    (impl[(type T)] Debug[] for (Data < T >) where [(T : Debug)] {})

    ; struct Data<T> { }
    (struct Data[(type T)] where [] { (data : T)})
})
```

.footnote[
    Actually the output is much messier. But you get the idea.
]

---

# Layers

Programs begin in the...

# Rust layer

...and get transformed to the...

# Decl layer

---

# Decl layer: structs

```scheme
(struct Foo[(type T)] where [] { (data : T) })
```

```scheme
(struct Foo[(type T)] where [] {
    (Foo [(data : T)]) ; 👈
})
```

* Close to surface syntax in most ways
* More uniform

---

# Decl layer

```scheme
(struct Foo[(type T)] where [] {
    (Foo [(data : T)])
})
```

```scheme
(enum Option[(type T)] where [] {
    (Some [(data : T)])
    (None [])                   
})
```

---

# Decl layer: impls (1/3)

```scheme
(impl[(type T)] Debug[] for (Foo < T >) where [(T : Debug)] {})
;               -----------------------
```

```scheme
(impl[(type T)] (Debug [(Foo < T >)] where [(is-implemented (Debug [T])] {})
;               --------------------
```

--

Another example of more uniform notation:

```scheme
(TraitRef ::= (TraitId Parameters))

(Parameter ::= Ty Lt)
```

---

# Decl layer: impls (2/3)

```scheme
(impl[(type T)] Debug[] for (Foo < T >) where [(T : Debug)] {})
;                                              -----------
```

```scheme
(impl[(type T)] (Debug [(Foo < T >)] where [(is-implemented (Debug [T])] {})
;                                           ---------------------------
```

**Rust where-clauses** become **logical formulae**<sup>1</sup>

.footnote[
    <sup>1</sup> Please forgive me the pretentious, and probably wrong, latin plural here. I've been waiting my whole life to end a word with an `-e`.
]

---

# Logical formulas

Base predicates:

```scheme
(Predicate ::= (is-implemented TraitRef)
               ...
               )
```

---

# Decl layer

```scheme
(impl[(type T)] Debug[] for (Foo < T >) where [(T : Debug)] {})
;                           -----------
```

```scheme
(impl[(type T)] (Debug [(rigid-ty Foo [T])] where [(is-implemented (Debug [T])] {})
;                       ------------------
```

**Rust types** become **internal types**

---

# Rust types in the decl layer

```scheme
(Ty ::= RigidTy AliasTy PredicateTy VarId)
```

* **Rigid types:** things like `u32`, `Vec`, or `String`

--
* **Alias types:** associated types like `T::Item`

--
* **Predicate types:** sci-fi types<sup>1</sup> like `∀X. T`, `∃X. T`, or `P => T` .footnote[ These types don't directly exist in Rust. We desugar more complex types to use them. ]

--
* **Variables:** Generic types or inference variables

---

# Decl layer

* Uniform notation
* Rust where clauses expressed as logical formulas
* Rust types broken to their "essence"

---

Programs begin in the **Rust layer**...

...get transformed to the **Decl layer**...

--

...and then checked by the 

# Check layer

---

# Check layer

* Given a Rust program...
    * Check each item in the crate is well-formed
    * Check coherence: impls don't apply to the same type
    * Check each function body type checks
    * Check each function body passes the borrow check

---

# Type checking

---

# Representing function bodies

```rust
struct Data<T> { value: T }

fn process(mut datum: Data<u32>) {
    datum.value += 1;
    print(datum);
}

fn print<T: Debug>(t: T) {...}
```

---

name: mir

# Rust's MIR

```rust
fn process(_1: Data<u32>) -> () {
    let mut _0: ();
    let mut _2: u32;
    let _3: ();
    let mut _4: Data<u32>;

    bb0: {
        _2 = _1.0 + const 1_u32;                  // _2 = datum.value + 1
        (_1.0: u32) = move _2;                    // datum.value = _2
        _4 = move _1;                             // _4 = datum;
        _3 = print::<Data<u32>>(move _4) -> bb1;  // _3 = print(_4);
    }

    bb1: {
        return;
    }
}
```

---

template: mir

.line1[![Arrow](images/Arrow.png)]

---

template: mir

.arg1[![Arrow](images/Arrow.png)]

---

template: mir

.bb0[![Arrow](images/Arrow.png)]

---

template: mir

.addition[![Arrow](images/Arrow.png)]

---

template: mir

.move41[![Arrow](images/Arrow.png)]

---

template: mir

.movekw[![Arrow](images/Arrow.png)]

---

template: mir

.turbofish[![Arrow](images/Arrow.png)]

---

template: mir

.branchbb1[![Arrow](images/Arrow.png)]

---

name: mir

# Type checker

```
```