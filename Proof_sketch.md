# Sketch of Proposed Non-Heuristic Proof of the Finiteness of Prime-Complete Products of Consecutive Integers

## 1. Background
This note presents a sketch of a non-heuristic proof for finiteness of the 
OEIS sequence [A141399]( https://oeis.org/A141399), the prime-complete products of the form

$$ N_2(n)=n(n+1), $$

with termination at $n = 633{,}555.$ The sequence comprises:

$$n = 1, 2, 3, 5, 8, 9, 14, 15, 20, 24, 35, 80, 125, 224, 384, 440, 539, 714, 1715, 
2079, 2400, 3024, 4374, 9800, 12375, 123200, 194480, 633555 $$

A stong heuristic argument for termination has been presented in the file Pidx_Argument.md in
[this](https://github.com/kenatiod/Pidx_search/tree/main) repository. In order to advance from
an heuristic argument to an analytic proof, we must examine how a pair of consecutive integers
form a product that has a greatest prime divisor index, Pidx, equal to its number of factors, $\omega$.

## 2. Solution Distribution
A key observation was put in A141399 by Michael De Vlieger on Jul 13 2024, which I will call the "De Vlieger Distribution":

|$\omega$  | p_ω＃     |{n values of solutions}               |
| -------: | ------------: | :-------------------------------------- |
| 1 | 2 | {1} |
|2  | 6 | {2, 3, 8} |
|3  |        30  |      {5, 9, 15, 24, 80} |
|4  |       210  |      {14, 20, 35, 125, 224, 2400, 4374} |
|5  |      2310  |      {384, 440, 539, 3024, 9800} |
|6  |     30030  |      {1715, 2079, 123200} |
|7  |    510510  |      {714, 12375, 194480} |
|8  |   9699690  |      {633555} |
|9  | 223092870  |      {} |

If the first column is $\omega(N_2)$, and if we count the prime-complete values we get:

|$\omega$ | solutions |  
| --: | :---------- |
|1| - 1|
|2| --- 3|
|3| ----- 5|
|4| ------- 7|
|5| ----- 5|
|6| --- 3|
|7| --- 3|
|8| - 1|
|9| 0|

What causes this distribution? It seems that the rising slope of the curve is the result of increasing numbers 
of possible solutions with increasing variables as prime divisor exponents, while the falling slope is a result 
of the difficulty of integer quantized exponents to hit a target of consecutive integers. If one sees this 
distribution and applies [Størmer's Theorem](https://en.wikipedia.org/wiki/St%C3%B8rmer%27s_theorem) one 
can make a path to a proof.

## 3. Equation Solving
This proof requires looking at the equations that have prime-complete solutions for each case of $\omega$ in 
products of consecutive integers. For example, if we look at the $\omega = 2$ case we have the equation:

$$2^a \times 3^b  + 1 = 2^c \times 3^d$$ 

where $a$ and $c$ cannot both be greater than zero (consecutive integers are coprime) but one must be, and the 
same goes for $b$ and $d$. The solutions to this Diophantine equation are $(a, b, c, d) = (1,0,0,1), (0,1,2,0) 
\text{ and } (3,0,0,2)$, which match the De Vlieger Distribution we saw above. For $\omega = 3$ we would need solutions to:

$$2^a \times 3^b \times 5^c  + 1 = 2^d \times 3^e \times 5^f$$

with corresponding restrictions on exponent pairs $(a, d), (b, e) \text{ and } (c, f)$, and so on for all values of $\omega.$ 
It seems an infinite task of equation solving, except Størmer sets finite limits on solutions and those 
can be solved by algorithms (e.g. Lehmer's method) involving [Pell equations](https://en.wikipedia.org/wiki/Pell%27s_equation).

Solving the Pell equations under the Størmer-Lehmer limitation is significant because it gives the total set of 
solutions for each value of $\omega$; i.e. proof for those values. Fortunately, the 
[PARI symbolic math system](https://en.wikipedia.org/wiki/PARI/GP) has a Pell solver function, 
and it is possible to write a Python interface to PARI. I have written a program, Nr_Solver.py, 
[at this repo](https://github.com/kenatiod/Nr_Solver), to do that. Nr_Solver takes a parameter "--r" 
that specifies the run length of the product, which is 2 for $N_2$. It also takes a "--start_omega" 
and "--end_omega" to specify the omega range through which to solve the equations. The data at the 
repo has the results of the $N_2$ run for $\omega = 2$ through $\omega = 17$, which match the distribution, 
and prove no solutions for $\omega = 9$ through $17$.

The time it takes to run Nr_Solver.py goes up exponentially with omega, so going past $\omega = 17$ is 
not so easy. However, it establishes a base, and shifts the general proof to showing that if we could 
keep Nr_Solver.py running, it would never hit another solution. For that, we need to reach 
for the Chinese Remainder Theorem (CRT).


## 4. Cutting Off the Solution Tail
The CRT tells you when a system of simultaneous congruence requirements can be satisfied, and if so, 
what the smallest solution looks like. If one needs an index $n$ satisfying $n ≡ 3 (mod 5)$ 
and $n ≡ 1 (mod 7)$ and $n ≡ 0 (mod 4)$ simultaneously, the CRT guarantees a unique solution 
modulo $lcm(5, 7, 4) = 140$, with subsequent solutions spaced $140$ apart. The Entry $LCM$ $\lambda$ 
is exactly this: each prime $p$ that must divide $n(n+1)$, but whose Pell entry period does 
not fit within the Lehmer bound, $L$, contributes a congruence condition on the index $n$. 
The CRT combines all these conditions into a minimum valid index, $\lambda$. If $\lambda > L$, 
the entire Pell family is eliminated — no candidate n exists within the range that 
Størmer's theorem allows, and that family is dead. Crucially, one need never compute 
the actual value of $n$ at index $\lambda$, which would be astronomically large. Just 
compute the entry periods, take their $LCM$, compare to $L$, and one line of arithmetic 
settles the question.

What makes this devastating as $\omega$ grows is that each additional missing prime 
contributes an entry period of at least $2$, and typically much larger, to the $LCM$. 
Even when entry periods share common factors — which can reduce $\lambda$ below the 
raw product — the CRT lower bound on $\lambda$ grows far faster than $L$, which increases 
only linearly with $p_{\omega}$. By $\omega = 9$, every mask family (every possible assignment 
of which primes are "missing" from the product) has a $\lambda$ that overshoots $L$; the Nr_Solver 
outputs for $\omega = 9$ through 17 confirm this exhaustively, certifying 
zero prime-complete pairs in each case.

For $\omega > 17$, the CRT argument becomes self-contained and requires no Pell computation at all. 
The number of missing primes is large enough that $\lambda$ exceeds $L$ for every possible mask 
family by pure arithmetic, independent of any search. The computational phase establishes the base; 
the CRT closes off all remaining cases analytically. Together they imply that the list of 
prime-complete products of consecutive integers — the sequence A141399 is provably finite, 
with $m = 633{,}555$ constituting its largest member.

