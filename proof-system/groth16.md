---
title: Groth16
tags: [proof system, groth16, zero knowledge]

---

# Groth16
:::info
$Problem: 3-Factorization$
Problem statement: Given $x_4$, we want to prove (in a zero knowledge fashion) that we know $x_1, x_2, x_3$ that $x_1 · x_2 · x_3 = x_4$, with all of those numbers are from the finite field $F_{13}$
:::
For examples:
✅ $<x_1, x_2, x_3, x_4> = <2,12,4,5>$ is a valid solution
❌ $<x_1, x_2, x_3, x_4> = <2,12,5,5>$ is not a valid solution
❌ $<x_1, x_2, x_3, x_4> = <2,12,4,7>$ is not a valid solution

## Constrain
In the context of zero-knowledge proof systems, our notion of constructive proofs is refined in such a way that it is possible to hide parts of the proof instance and still be able to prove the statement. In this context, it is therefore necessary to split a proof into an **unhidden, public par**t called the ==instance== and a **hidden, private part** called a ==witness==

We can express our problem this way
$$
W1 · W2 · W3 = I1 
$$

## Rank-1 Constraint Systems (R1CS)
:::info
Rank-1 (quadratic) Constraint Systems is a system of constrains, whose left side is a product of 2 factors (quadratic) as follow

- k: number of contrains
- n: number of instances
- m: number of witnesses
- i: superscript s.t 1 ≤ i ≤ k
- j: subscript s.t 0 ≤ j ≤ n+m

\begin{align*}
\left( a_{0}^1 + \sum_{j=1}^{n} a_{j}^1 I_j + \sum_{j=1}^{m} a_{n+j}^1 W_j \right) \left( b_{0}^1 + \sum_{j=1}^{n} b_{j}^1 I_j + \sum_{j=1}^{m} b_{n+j}^1 W_j \right) &= c_{0}^1 + \sum_{j=1}^{n} c_{j}^1 I_j + \sum_{j=1}^{m} c_{n+j}^1 W_j \\
&\vdots \\
\left( a_{0}^k + \sum_{j=1}^{n} a_{j}^k I_j + \sum_{j=1}^{m} a_{n+j}^k W_j \right) \left( b_{0}^k + \sum_{j=1}^{n} b_{j}^k I_j + \sum_{j=1}^{m} b_{n+j}^k W_j \right) &= c_{0}^k + \sum_{j=1}^{n} c_{j}^k I_j + \sum_{j=1}^{m} c_{n+j}^k W_j
\end{align*}
:::

In 3-Factorization, our constrain is not a product of 2 factors, we need to flatten it

\begin{align}
W1 · W2 · W3 = I1 \\
\Leftrightarrow 
\begin{cases}
W1 · W2 = W4 \\
W4 · W3 = I1
\end{cases}
\end{align}

Now we have 2 constrains (k = 2), 1 instance (n = 1) and 4 witness (m = 4).
With 
\begin{align*}
a^1_0=0 \quad a^1_1=0 \quad a^1_2=1 \quad a^1_3=0 \quad a^1_4=0 \quad a^1_5=0 \\ 
a^2_0=0 \quad a^2_1=0 \quad a^2_2=0 \quad a^2_3=0 \quad a^2_4=0 \quad a^2_5=1 \\
b^1_0=0 \quad b^1_1=0 \quad b^1_2=0 \quad b^1_3=1 \quad b^1_4=0 \quad b^1_5=0 \\
b^2_0=0 \quad b^2_1=0 \quad b^2_2=0 \quad b^2_3=0 \quad b^2_4=1 \quad b^2_5=0 \\
c^1_0=0 \quad c^1_1=0 \quad c^1_2=0 \quad c^1_3=0 \quad c^1_4=0 \quad c^1_5=1 \\
c^2_0=0 \quad c^2_1=1 \quad c^2_2=0 \quad c^2_3=0 \quad c^2_4=0 \quad c^2_5=0 
\end{align*}
our 2 constrains now can be represented by R1CS below


\begin{cases}
W1 · W2 = W4 \\
W4 · W3 = I1
\end{cases}

$\Leftrightarrow$

\begin{cases}
\left( a_{0}^1 + a_{1}^1 I_1 + a_{2}^1 W_2 + a_{3}^1 W_3 + a_{4}^1 W_4 \right) \left( b_{0}^1 + b_{1}^1 I_1 + b_{2}^1 W_2 + b_{3}^1 W_3 + b_{4}^1 W_4 \right) = \left( c_{0}^1 + c_{1}^1 I_1 + c_{2}^1 W_2 + c_{3}^1 W_3 + c_{4}^1 W_4 \right) \\
\left( a_{0}^2 + a_{1}^2 I_1 + a_{2}^2 W_2 + a_{3}^2 W_3 + a_{4}^2 W_4 \right) \left( b_{0}^2 + b_{1}^2 I_1 + b_{2}^2 W_2 + b_{3}^2 W_3 + b_{4}^2 W_4 \right) = \left( c_{0}^2 + c_{1}^2 I_1 + c_{2}^2 W_2 + c_{3}^2 W_3 + c_{4}^2 W_4 \right)
\end{cases}

## Quadratic Arithmetic Programs (QAP)
### QAP representation
:::info
A QAP associated to the R1CS $\mathcal{R}$ is the following ==set of polynomials== over $\mathbb{F}$:
$$
QAP(\mathcal{R}) = \left\{ T(x), \{ A_j, B_j, C_j \in \mathbb{F}[x] \}_{j=0}^{n+m} \right\} \quad (6.16)
$$
Here $T(x) := \prod_{l=1}^{k} (x - m_l)$ is a polynomial of degree $k$, called the **target polynomial** of the QAP and $A_j$, $B_j$ as well as $C_j$ are the unique degree $k-1$ polynomials defined by the following equation:
$$
A_j(m_i) = a_j^i, \quad B_j(m_i) = b_j^i, \quad C_j(m_i) = c_j^i 
$$

---
Computing a QAP from any given R1CS can be achieved in the following steps.
- Choose k different, invertible elements ($m_1,\dots,m_k$) from the field F`(If F is prime field, any non-zero element is invertible)`. Every choice defines a different QAP for the same R1CS. 
- Calculate T(x) = $(x−m_1)\dots(x−m_2)$
- Use Lagrange Interpolation to calculate A(x), B(x) and C(x)
:::

Since our 3-Factorization problem has two constraints, we need to choose two arbitrary but invertible and distinct elements m1 and m2 from F13. Let pick $m_1$ = 5 and $m_2$ = 7. The target polynomial
$$
\begin{align}
T(x) &= (x−m_1)(x−m_2) \\
&= (x − 5)(x − 7) \\
&= (x+8)(x+6) \\
&= x^2 +x+9 \\
\end{align}
$$
Then, it's time for $A_j, B_j, C_j$. Since the R1CS has two constraining equations, those polynomials are of degree 1 and they are defined by their evaluation at the point m1 = 5 and the point m2 = 7.
At point $m_1$, each polynomial $A_j$ is defined to be $a^1_j$, and at point $m_2$, each polynomial $A_j$ is defined to be $a^2_j$. The same holds true for the polynomials Bj as well as $C_j$. Writing all these
equations down, we get:
\begin{align}
A_0(5)=0 \quad A_1(5)=0 \quad A_2(5)=1 \quad A_3(5)=0 \quad A_4(5)=0 \quad A_5(5)=0 \\
A_0(7)=0 \quad A_1(7)=0 \quad A_2(7)=0 \quad A_3(7)=0 \quad A_4(7)=0 \quad A_5(7)=1 \\
B_0(5)=0 \quad B_1(5)=0 \quad B_2(5)=0 \quad B_3(5)=1 \quad B_4(5)=0 \quad B_5(5)=0 \\
B_0(7)=0 \quad B_1(7)=0 \quad B_2(7)=0 \quad B_3(7)=0 \quad B_4(7)=1 \quad B_5(7)=0 \\
C_0(5)=0 \quad C_1(5)=0 \quad C_2(5)=0 \quad C_3(5)=0 \quad C_4(5)=0 \quad C_5(5)=1 \\
C_0(7)=0 \quad C_1(7)=1 \quad C_2(7)=0 \quad C_3(7)=0 \quad C_4(7)=0 \quad C_5(7)=0
\end{align}
Use Lagrange interpolation, we get 
![image](https://hackmd.io/_uploads/BkiaA330ex.png)

Our final QAP is
\begin{multline}
QAP(R_{3.fac_zk}) = \{x^2 +x+9, \{0,0,6x+10,0,0,7x+4\},\\
\{0,0,0,6x+10,7x+4,0\},\{0,7x+4,0,0,0,6x+10\}\}
\end{multline}


### QAP Satisfiability
Prover calculate polynomial P(W,I) as follow
\begin{multline}
P_{(I;W)} = \\
(A_0+\sum{n_j I_j·A_j}+\sum{m_j Wj·A_{n+j}})·(B_0+\sum{n_j I_j·B_j}+\sum{m_j Wj·B_{n+j}} \\ −(C_0+\sum{n_j I_j·C_j}+\sum{m_j W_j·C_{n+j}}
\end{multline}

and send to verifier. Verifier accept the proof if ==$T(x)$ | $P_{(I;W)}$==

\begin{align}
P_{(I;W)} &=(2(6x+10)+6(7x+4))·(3(6x+10)+4(7x+4))
−(11(7x+4)+6(6x+10)) 
\\&=((12x+7)+(3x+11))·((5x+4)+(2x+3))−((12x+5)+(10x+8)) 
\\&=(2x+5)·(7x+7)−(9x)
\\&=(x2 +2·7x+5·7x+5·7)−(9x) 
\\&=(x2 +x+9x+9)−(9x)
\\&=x^2 +x+9
\end{align}

In our particular example, $P_{(I;W)}$ is accidentally equal to the target polynomial T, and hence, it is divisible by T with P/T = 1

:::success
**WHY $T(x)$ | $P_{(I;W)}$ ?**

---

At $x = 5$
$$P(x) = P(5) = A(5) \cdot B(5) - C(5) = A_1 \cdot B_1 - C_1 = 0$$
At $x = 7$
$$P(x) = P(7) = A(7) \cdot B(7) - C(7) = A_2 \cdot B_2 - C_2 = 0$$
So $P(x)$ as two roots 5 and 7 and P(x) can be written as
\begin{align}
P(x) &= (x-5)(x-7)R(x)\\
     &= T(x)R(x)
\end{align}
:::
# Groth16
:::info
$Problem: 3-Factorization$
Problem statement: Given $x_4$, we want to prove (in a zero knowledge fashion) that we know $x_1, x_2, x_3$ that $x_1 · x_2 · x_3 = x_4$, with all of those numbers are from the finite field $F_{13}$
:::
For examples:
✅ $<x_1, x_2, x_3, x_4> = <2,12,4,5>$ is a valid solution
❌ $<x_1, x_2, x_3, x_4> = <2,12,5,5>$ is not a valid solution
❌ $<x_1, x_2, x_3, x_4> = <2,12,4,7>$ is not a valid solution

## Constrain
In the context of zero-knowledge proof systems, our notion of constructive proofs is refined in such a way that it is possible to hide parts of the proof instance and still be able to prove the statement. In this context, it is therefore necessary to split a proof into an **unhidden, public par**t called the ==instance== and a **hidden, private part** called a ==witness==

We can express our problem this way
$$
W1 · W2 · W3 = I1 
$$

## R1CS representation
:::info
Rank-1 (quadratic) Constraint Systems is a system of constrains, whose left side is a product of 2 factors (quadratic) as follow

- k: number of contrains
- n: number of instances
- m: number of witnesses
- i: superscript s.t 1 ≤ i ≤ k
- j: subscript s.t 0 ≤ j ≤ n+m

\begin{align*}
\left( a_{0}^1 + \sum_{j=1}^{n} a_{j}^1 I_j + \sum_{j=1}^{m} a_{n+j}^1 W_j \right) \left( b_{0}^1 + \sum_{j=1}^{n} b_{j}^1 I_j + \sum_{j=1}^{m} b_{n+j}^1 W_j \right) &= c_{0}^1 + \sum_{j=1}^{n} c_{j}^1 I_j + \sum_{j=1}^{m} c_{n+j}^1 W_j \\
&\vdots \\
\left( a_{0}^k + \sum_{j=1}^{n} a_{j}^k I_j + \sum_{j=1}^{m} a_{n+j}^k W_j \right) \left( b_{0}^k + \sum_{j=1}^{n} b_{j}^k I_j + \sum_{j=1}^{m} b_{n+j}^k W_j \right) &= c_{0}^k + \sum_{j=1}^{n} c_{j}^k I_j + \sum_{j=1}^{m} c_{n+j}^k W_j
\end{align*}
:::

In 3-Factorization, our constrain is not a product of 2 factors, we need to flatten it

\begin{align*}
W1 · W2 · W3 = I1 \\
\Leftrightarrow 
\begin{cases}
W1 · W2 = W4 \\
W4 · W3 = I1
\end{cases}
\end{align*}

Now we have 2 constrains (k = 2), 1 instance (n = 1) and 4 witness (m = 4).
With 
\begin{align*}
a^1_0=0 \quad a^1_1=0 \quad a^1_2=1 \quad a^1_3=0 \quad a^1_4=0 \quad a^1_5=0 \\ 
a^2_0=0 \quad a^2_1=0 \quad a^2_2=0 \quad a^2_3=0 \quad a^2_4=0 \quad a^2_5=1 \\
b^1_0=0 \quad b^1_1=0 \quad b^1_2=0 \quad b^1_3=1 \quad b^1_4=0 \quad b^1_5=0\\
b^2_0=0 \quad b^2_1=0 \quad b^2_2=0 \quad b^2_3=0 \quad b^2_4=1 \quad b^2_5=0\\
c^1_0=0 \quad c^1_1=0 \quad c^1_2=0 \quad c^1_3=0 \quad c^1_4=0 \quad c^1_5=1\\
c^2_0=0 \quad c^2_1=1 \quad c^2_2=0 \quad c^2_3=0 \quad c^2_4=0 \quad c^2_5=0 
\end{align*}