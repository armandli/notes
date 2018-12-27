## Algebraic Data Types
allowing the creation of new types by taking the sum or product of existing types. a sum of types means a super type that can be an instance of any of the element of the sum, a product of types means the type represent the cartesian combination of 2 types, with both required to exist.

algebraic data type allows pattern matching to match on the sumed type.

algebraic data type should be immutable.

### subset type
a subset type is a type combined with a predicate limiting on the value of the type.

### Quotient Type
the duel of subset type.

def. an equivalence relation (~) is a binary relation, it is reflexive x ~ x for all x, symmetric, x ~ y then y ~ x, and transitive, x ~ y and y ~ z then x ~ z.

we can check if a ~ b iff there exist integer k s.t. a - b = k * n for any integer n. a - a = 0 * n, a - b = k * n and b - a = - k * n, a - b = k1 * n, b - c = k2 * n, then a - b = (k1 + k2) * n

any binary relation can be a equivalence relation. this is called the reflexive, symetric, transitive closure of the relation.

if T is a type, and ~ is a equivalence relation, we use T / ~ as the notation for quotient type. we call T the underlying type of the quoteint type.

any value of the underlying type is a value of the quotient type. e.g. Z is subtype of Z / nZ where Z is natural number.

let function f : T / ~ -> X is for some type X, a function f : T / ~ -> X is a function g : T -> X which satisfies g(a) = g(b) for all a and b such that a ~ b. we call f well-defined if g satisfy the condition. in other words, any function that consume a quotient type is not allowed to produce an output that distinquish between equivalent inputs.

quotient types allow us to change what notion of equality is for a type.

any function h : T -> B gives rise to an equivalence relation on T via a ~ b iff h(a) = h(b). in this case, any function g : B -> X gives rise to function f : T / ~ -> X via f = g * h. when B = T, we are guaranteed to have suitable g for any function f : T / ~ -> X

common example is rational numbers. we can reduce rational numbers to lowest term either when it's produced or the numerator or denominator gets accessed, so that we don't acciently write functions that distinquish between 1 / 2 and 2 / 4. for modular arithmetic, mod by n function is a suitable h

### Quotient Type in Set Theory View
in set theory, such an h function can always be made by mapping the elements of T to equivalence classes containing them. i.e. a gets mapped to { b | a ~ b } which is called equivalence class of a. in fact, in set theory T / ~ is usually defined to be set of equivalence class of ~.

equivalence class are also called partitions and are said to partition the underlying set. elements of these equivalence class are called representatives of equivalence class. often notation [a] is used for the equivalence class of a.

#### Examples
1) integers can be represented as pair of natural numbers (n, m) with the idea being that the pair represent n - m. equivalence relation is (n1, m1) ~ (n2, m2) iff n1 + m1 = n2 + m2
2) rationals can represented as pair of integer number n and non-zero natural number m, (n, m) representing n / m. with equivalence relation (n1, m1) ~ (n2, m2) iff n1 * m1 = n2 * m2
3) topological circles. we can extend integers mod n to the continuous case. consider the real numbers with equivalence relation r ~ s iff s - k = k for some integer k. you can call this reals mod 1. topologically this is a circle. if you walk on it long enough you end up at a point equivalent to where you started. occasionally this is written as R / Z with R being real and Z being integer
4) Torii. doing topological circles in 2D gives a torus. specifically we have a pair of real numbers with equivalence relation (x1, y1) ~ (x2, y2) iff x1 - x2 = k and y1 - y2 = l for some integers k and l.
5) unordered pairs. consider equivalence relation on arbiturary pairs (a1, b1) ~ (a2, b2) iff a1 = a2 and b1 = b2 or a1 = b2, and b1 = a2. a pair of numbers is equivalent to itself and swapped place version of itself. 
6) Gluing/Pushout. in topology, we want to take 2 topological spaces and glue them together in a specific way. this produce a space that contains both input spaces, but don't interact in any way. we now want to say that certain pairs of points, one from each space, are really the same point. this is, we want a quotient by an equivalence relation that would identify those points. we need some mechanism to specify which point we want to specify. we can define relation R on the disjoint sum via R(a, b) iff there is c such that a = inl(f(c)) and b = inr(g(c)), this is not an equivalence relation, but can be extended into one. the quotient we get is then gluing A and B specified by C.
7) polynomial ring ideals. we use R[X] for type of polynomials with one indeterminate X, R[X,Y] for polynomial with 2 interdeterminate X and Y. e.g. X^2 + 1, X^2 + Y^2. We consider quotienting these types by equivalence relations generated from identifications like X^2 + 1 ~ 0, or X^2 - Y ~ 0. we want more than just reflexive, symmetric, transitive closure, on top of that we want the equivalence relation to respect operations we have on polynomials, in particular addition and multiplication. more precisely, we want a ~ b and c ~ d then ac ~ bd, and similarly for addition. equivalence relation that respect all these operations is called **congruence**. example: R[X,Y] / (X^2 + 1, X^2 - Y). an **ideal** is an equivalence class of 0 with respect to some **congruence**.

