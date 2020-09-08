## Cellular Automata Rule 30 for random number generation

non-cryptographic pseudo-random number generation can be done using Rule 30 from Cellular Automata.

#### Rule 30

rule 30 is an elementary rule determining the state of a cell based on its current left and right cell value. there are 8 states:

current cell states left, cell, right => next cell state
1. 1 1 1 => 0
2. 1 1 0 => 0
3. 1 0 1 => 0
4. 1 0 0 => 1
5. 0 1 1 => 1
6. 0 1 0 => 1
7. 0 0 1 => 1
8. 0 0 0 => 0

we can build a 2D grid, where the rows represent different time generation, and column represent different cells. each cell has an initial cell state. the behavior of the grid becomes random over time. we can generate random number of taking value from one of the column for n number of bits after k generations, k being the initial seed number.

the advantage of rule 30 is we can generate random numbers in parallel, with each column of the grid can generate random numbers at the same time.