
If  $G/Z(G)$ is cyclic, prove G is abelian.

G is finite group. For any prime divisor p of $|G|$ , $\exists g \in G$ s.t. $|g|=p$. 


Here is a detailed English solution for **Problem 3.36**. You can copy this directly into your LaTeX document.

### Problem 3.36

If $R$ is a commutative ring, define $R[[x]]$ as the **Ring of Formal Power Series** over $R$, which is the set of all sequences of elements in $R$.

(i) Show that the definitions of addition and multiplication on $R[x]$ are valid for $R[[x]]$, and prove that $R[[x]]$ forms a commutative ring under these operations.

(ii) Prove that $R[x]$ is a subring of $R[[x]]$.

(iii) Let $\sigma = (s_0, s_1, s_2, \dots)$ be denoted by $\sigma = s_0 + s_1x + s_2x^2 + \dots$. Prove that if $\sigma = 1 + x + x^2 + \dots$, then $\sigma = 1/(1-x)$ in $R[[x]]$.

---

### Solution

**(i) Ring Structure of $R[[x]]$**

Let $R[[x]]$ be the set of formal power series $\sum_{i=0}^{\infty} a_i x^i$ with coefficients $a_i \in R$. Let $f = \sum_{i=0}^{\infty} a_i x^i$ and $g = \sum_{j=0}^{\infty} b_j x^j$ be elements of $R[[x]]$.

**1. Well-definedness of Operations**

- **Addition:** Defined component-wise as $f + g = \sum_{i=0}^{\infty} (a_i + b_i)x^i$. Since $R$ is a ring, $a_i + b_i$ is a well-defined element of $R$ for every $i$. Thus, the resulting series is a valid element of $R[[x]]$.
    
- **Multiplication (Cauchy Product):** Defined as $fg = \sum_{k=0}^{\infty} c_k x^k$, where $c_k = \sum_{i=0}^{k} a_i b_{k-i}$.
    
    For any fixed index $k$, the coefficient $c_k$ is a sum of finitely many products of elements in $R$. Since $R$ is a ring, this finite sum is well-defined and yields a unique element in $R$. Thus, the product series is a valid element of $R[[x]]$.
    

**2. Commutative Ring Axioms**

- **Additive Group:** $(R[[x]], +)$ is an abelian group because addition is defined component-wise, and $(R, +)$ is an abelian group. The additive identity is the zero series $0 = \sum 0 x^i$. The additive inverse of $f$ is $-f = \sum (-a_i)x^i$.
    
- **Multiplicativity Identity:** The series $1 = 1 + 0x + 0x^2 + \dots$ serves as the multiplicative identity.
    
    Proof: The $k$-th coefficient of $f \cdot 1$ is $\sum_{i=0}^k a_i \cdot (\text{coeff of } x^{k-i} \text{ in } 1)$. The term in the sum is non-zero only when $k-i = 0$ (i.e., $i=k$), in which case it is $a_k \cdot 1 = a_k$. Thus $f \cdot 1 = f$.
    
- **Commutativity:** We show $fg = gf$.
    
    The coefficient of $x^k$ in $fg$ is $c_k = \sum_{i=0}^{k} a_i b_{k-i}$.
    
    Let $j = k-i$. As $i$ ranges from $0$ to $k$, $j$ ranges from $k$ to $0$. We can rewrite the sum as:
    
    $$ c_k = \sum_{j=0}^{k} a_{k-j} b_j. $$
    
    Since $R$ is a commutative ring, $a_{k-j} b_j = b_j a_{k-j}$. Thus:
    
    $$ c_k = \sum_{j=0}^{k} b_j a_{k-j}. $$
    
    This is precisely the definition of the coefficient of $x^k$ in the product $gf$. Hence $fg = gf$.
    
- **Associativity and Distributivity:** These follow from the associativity and distributivity of $R$ and the manipulation of finite sums (similar to the proof for polynomials).
    

**Conclusion:** $R[[x]]$ is a commutative ring. $\square$

**(ii) $R[x]$ is a Subring**

Recall that the polynomial ring $R[x]$ consists of series with only finitely many non-zero terms:

$$ R[x] = \left\{ \sum_{i=0}^{\infty} a_i x^i \in R[[x]] \;\middle|\; \exists N \in \mathbb{N} \text{ such that } a_i = 0 \text{ for all } i > N \right\}. $$

To prove $R[x]$ is a subring of $R[[x]]$, we verify the subring criteria:

1. **Identity:** The multiplicative identity $1 = 1 + 0x + \dots$ has only one non-zero term, so $1 \in R[x]$.
    
2. **Closed under Subtraction:** Let $f, g \in R[x]$. Let $\deg(f) = n$ and $\deg(g) = m$. Then for all $k > \max(n, m)$, the coefficients of both $f$ and $g$ are zero. Consequently, the coefficients of $f-g$ are zero for all $k > \max(n, m)$. Thus $f-g \in R[x]$.
    
3. **Closed under Multiplication:** Let $f, g \in R[x]$ with degrees $n$ and $m$ respectively. The coefficient $c_k$ of their product involves terms $a_i b_{k-i}$. If $k > n+m$, then for any pair $(i, k-i)$, either $i > n$ (so $a_i=0$) or $k-i > m$ (so $b_{k-i}=0$). In either case, every term in the sum for $c_k$ is zero. Thus $fg$ is a polynomial of degree at most $n+m$, so $fg \in R[x]$.
    

Therefore, $R[x]$ is a subring of $R[[x]]$. $\square$

**(iii) Inverse of $(1-x)$**

Let $\sigma = 1 + x + x^2 + x^3 + \dots = \sum_{i=0}^{\infty} 1 \cdot x^i$.

We want to show that $\sigma = (1-x)^{-1}$. This is equivalent to showing that:

$$ (1-x) \cdot \sigma = 1. $$

We compute the product directly in $R[[x]]$:

$$ (1-x)\sigma = 1 \cdot \sigma - x \cdot \sigma $$

$$ = (1 + x + x^2 + x^3 + \dots) - (x + x^2 + x^3 + x^4 + \dots) $$

Let us look at the coefficient of $x^k$ in the result:

- **For $k=0$ (Constant term):** The first series contributes $1$, the second contributes $0$. Total: $1$.
    
- **For $k \ge 1$:** The first series contributes $1$ (coefficient of $x^k$). The second series contributes $-1$ (coefficient of $x \cdot x^{k-1} = x^k$). Total: $1 - 1 = 0$.
    

Thus:

$$ (1-x)\sigma = 1 + 0x + 0x^2 + \dots = 1. $$

Since the product is the multiplicative identity, $\sigma$ is the inverse of $1-x$.

$\square$



> [!question]
> If $G$ is a finite abelian group, then for ant prime divisor $p$ of $|G|$, $G$ has an element $g\in G$ such that g has order $p$.

对群的阶数 $∣G∣=n$ 进行归纳。
1. $n=2$ 时 $G=\{ a,e \},a^{2}=e,|a|=2$，结论成立
2. 设对所有阶小于 $n$ 的交换群都成立
3. 考察阶为 $n$ 的交换群 $G$, $p$ 为 $n$ 的任一素因子. $\forall a\in G,a \neq e$, 设 $|a|=r>1$
	1. 若 $p|r$, 则 

$$
Prove that S4/V  S3.
$$

$G_{\sigma x}=\sigma G_{x}\sigma^{-1}$

Show that the prime field of $\mathbb{R}$ is $\mathbb{Q}$.

证明 $<x^{2}+1>$ 为 $\mathbb{Z}_{3}[x]$ 的极大理想。

$\bar{ar+b}=\bar{ar}$