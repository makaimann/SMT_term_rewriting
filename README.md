# SMT_term_rewriting

Generalized algebraic theories (GATs) consist of:
- declaring types of entities (called sorts, which can depend on values, e.g. "List of length `i`"),
- operator declarations (which can be thought of as pure functions among terms inhabiting the above sorts)
- equations (notions of when two sorts or two terms are equal).

This is a very expressive language, but to be useful we must be able to identify when arbitrary terms are equal or not with respect to the equations of the theory. Although this is an undecidable problem in general, we can convert the question of whether two terms are equal into a logic problem which can be solved by a SMT solver through finite model checking. This strategy has at least two caveats: we are restricted to checking whether a path of rewrites exists up to a finite length, and we can apply rewrites only up to a finite depth from the root of any term.


## TO-DO
 - Connect to [SMT-switch](https://github.com/makaimann/smt-switch) and [cosa2](https://github.com/upscale-project/cosa2) to do model checking for an unspecified number of rewrites.

## Usage

To run this, first install [CVC4](https://github.com/CVC4/CVC4), then compile the program `src/ast.cpp` (this works on my Mac by running `src/bin/build.sh`). The executable prompts the user for the following inputs:
1. A user-specified theory (by name or path)
2. A starting term in that theory
3. A goal term in that theory
4. How many rewrite steps are needed
5. Max depth in the abstract syntax tree we want to be able to apply rewrites.

GATs can be declared in two ways. Firstly, they can be constructed with a C++ API, with examples in the `src/theories` folder. However, it's also possible to point to a file specifying a GAT. Each theory currently in `src/theories` has an equivalent model data file in the `data` folder to show how this is done. This is an snippet of a theory of arrays, with which we can read and write objects:
```
Sort Ob "Ob" "Some datatype that can be stored in an array" []
Sort N "Nat" "A natural number: 0, 1, 2, ..." []
Sort B "Bool" "Either true or false" []
Sort Arr "Arr" "An array that can be indexed by natural numbers." []

Op S "S({})" "Successor, e.g. S(0)=1" N [i:N]
Op E "({}≡{})" "Equality of numbers" B [i:N, j:N]
Op ite "ite({},{},{})" "If-then-else" Ob [b:B, o:Ob, p:Ob]
Op read "read({},{})" "Read array A at position i" Ob [A:Arr, i:N]
Op write "write({},{},{})" "Write object o to position i" Arr [A:Arr, i:N, o:Ob]

Rule row "Read-over-write: if indices not equal, look at previous write"
    read(write(A:Arr, i:N, o:Ob), j:N)
    ite(E(i:N, j:N), o:Ob, read(A:Arr,j:N))
Rule eq1 "Recursively peel of successor"
    E(i:N,j:N)
    E(S(i:N),S(j:N))
```

The second argument for Sort/Operation declarations is important for both printing and parsing expressions of the GAT. Parsing is important due to arguments 2 and 3, see for example `data/inputs1` or `data/inputs2`.

## Example

Consider the example in `data/inputs1` which searches for a rewrite path of length 2:
```
data/cat.dat
(x:(A:Ob⇒Q:Ob) ⋅ id(Q:Ob))
(id(A:Ob) ⋅ x:(A:Ob⇒Q:Ob))
2
3
```

The executable can be called with a file, e.g. `build/ast < data/inputs1`, which will print out that a solution was found and produce an output file `build/model.dat` which contains:
```
p0 : Path = Empty;
r0 : Rule = R2r;
p1 : Path = Empty;
r1 : Rule = R1f;
```
This shows that the first rewrite step applies the second rule of *cat* (`f:(A:Ob⇒B:Ob) <-> (f:(A:Ob⇒B:Ob) ⋅ id(B:Ob)))`) in the reverse direction at the top level, followed by the first rule (`f:(A:Ob⇒B:Ob) <-> (id(A:Ob) ⋅ f:(A:Ob⇒B:Ob))`), in the forward direction and also at the top level.
