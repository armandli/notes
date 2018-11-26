### Freivald Algorithm - probabilistic algorithm for verifying matrix multiplication
### Algorithm
input: A, B, C matrix of size n * n
goal: verify A * B = C

create vector r of size n * 1 where each element have value {0, 1} chosen at uniform random
compute A * (B * r) - (C * r) = o
if any element in o is none zero, A * B != C with probability of correctness 1
otherwise A * B = c with probability of correctness > 1/2

repeat with different random vector r for higher probabilty of correctness 

### Complexity
O(n^2) where n is the row size of the matrix
