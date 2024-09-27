---
title: Singular Value Decomposition and Investment Analysis
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2024-09-25 12:00:00 +0800
categories: [ML in Finance]
tags: [machine_learning, factor_model]
render_with_liquid: false
use_math: true
math: true
---

This post is about SVD for investment analysis.

## What is Singular Value Decomposition
For a set of orthogonal vectors, after a linear transformation, their magnitudes may change, but they can still remain orthogonal. What is this orthogonal set, and what is the result after the linear transformation? 

SVD is decompose matrix $A$ which can be decomposed into $ U \Sigma V^{T}$. In here each matrixs have below size and characteristics.

A: $(m \times n)$ rectangular matrix \\
U: $(m \times m)$ orthogonal matrix \\
$\Sigma$: $(m \times n)$ diagonal matrix \\
V: $(n \times n)$ orthogonal matrix \\

In here, orthogonal matrix(U) is a matrix $UU^T = I $ and thus $U^T = U^{-1}$. The orthogonal vectors \( $\vec{x}$, $\vec{y}$ \), shown in the example before the linear transformation, can be thought of as a collection of column vectors, which corresponds to the \( V \) matrix in \( $A = U \Sigma V^{T} $\).

$V =
\begin{pmatrix}
\vec{x} & \vec{y}
\end{pmatrix}
\tag{8}
$

Additionally, the orthogonal vectors after the linear transformation, $\( A \vec{x}, A \vec{y} \)$, can be normalized to have unit length as $(\vec{u}_1, \vec{u}_2)$. The collection of these column vectors corresponds to the \( U \) matrix in $\( A = U \Sigma V^T \)$.

$
U =
\begin{pmatrix}
\vec{u}_1 & \vec{u}_2
\end{pmatrix}
\tag{9}
$

Finally, the singular values (i.e., scaling factors) can be gathered into the \( \Sigma \) matrix as shown below.

$
\Sigma =
\begin{pmatrix}
\sigma_1 & 0 \\
0 & \sigma_2
\end{pmatrix}
\tag{10}
$

So, in terms of linear transformation, 
$
    AV = U \Sigma \\
    A = U \Sigma V^{-1} \\
    A = U \Sigma V^{T} \\
$

In here, matrix V is called singular matrix which consists of n singular vectors (Eigenvector). The left singular vectors U come from the rows of A, and the right singular vectors V come from the columns of A. This means that The left singular vectors U correspond to the domain (input) of the transformation, while the right singular vectors V correspond to the codomain (output).

Eigenvector exists if the matrix A is square matrix, while singular vector can be defined for both square and rectangular matrix. In fact, for symmetric matrices, the eigenvectors and the singular vectors are the same, and the eigenvalues correspond to the singular values. However, for non-symmetric or rectangular matrices, this relationship does not hold.


## How this can be used in investment portfolio
For our application in investment portfolio, which consists of time series data, formulation of SVD will follow below. We are going to use three years ETF return data ($ 250 \times 3 = 750$). And there are 10 5 ETFs in our portfolio. So, matrix A will have $(750 \times 5)$. Graphically, it would be like this.

![SVD](/assets/img/post_image/ML/SVD/picture1.png)


The power of Singular Value Decomposition (SVD) shines in the process of reconstructing a decomposed matrix.

Using only \( p \) singular values, we can approximate the matrix \( A' \) from the original matrix \( A \), which was decomposed into \( U, \Sigma, V^T \). As mentioned earlier, since the amount of information in \( A \) is determined by the magnitude of the singular values, even with only a few large singular values, we can retain a significant amount of useful information. $A \approx U \Sigma_p V^T$

![SVD](/assets/img/post_image/ML/SVD/picture2.png)
