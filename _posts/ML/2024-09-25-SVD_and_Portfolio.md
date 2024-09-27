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

SVD is decompose matrix A which can be decomposed into $ \text{U} \times \Sigma \times V^{T}$. In here each matrixs have below size and characteristics.

A: $(m \times n)$ rectangular matrix     
U: $(m \times m)$ orthogonal matrix (User matrix)    
$\Sigma$: $(m \times n)$ diagonal matrix  (Singular Values $\lambda$ in Matrix)  
V: $(n \times n)$ orthogonal matrix (Singular Vector)   

So, in terms of linear transformation, 

$ AV = U \Sigma $   
$ AVV^{-1} = U \Sigma V^{-1}$      
$ AVV^{T} = U \Sigma V^{T} $      
$ A = U \Sigma V^{T}$     

Eigenvector exists if the matrix A is square matrix, while singular vector can be defined for both square and rectangular matrix. In fact, for symmetric matrices, the eigenvectors and the singular vectors are the same, and the eigenvalues correspond to the singular values. However, for non-symmetric or rectangular matrices, this relationship does not hold.   

In here, orthogonal matrix(U) is a matrix $ UU^T = I $ and thus $U^T = U^{-1}$. The orthogonal vectors \( $\vec{x}$, $\vec{y}$ \), shown in the example before the linear transformation, can be thought of as a collection of column vectors, which corresponds to the \( V \) matrix in \( $A = U \Sigma V^{T} $\).
$ V =
\begin{pmatrix}
\vec{x}  \vec{y}
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

Finally, the singular values (i.e., scaling factors) can be gathered into the \( \Sigma \) matrix as shown below. In dimensionality reduction stage level of variance explained can be calculated by how many singluar values you choose in this matrix. 
$
\Sigma =
\begin{pmatrix}
\sigma_1 & 0 \\
0 & \sigma_2
\end{pmatrix}
\tag{10}
$

If you choose only $ \sigma_1 $ in dimensionality reduction, variance explained in dimensionality reduction can be expressed as follows:   
$ Var Explained = \frac{\sigma_1}{\sigma_1 + \sigma_2}$   

Combining this explained variance in singluar values, transpose of V $\( V^{T} \)$, can be used to explain portfolio. In here we are going to suppose that we have portfolio with N assets, and each asset have equal weight $\frac{1}/{N}$. If we use only $\sigma_1$ for dimensionality reduction, portfolio with weights $\vec{u}_1 \times original weight = (u_1, u_2, ... ,u_10) \times \( 1/10, 1/10, ... , 1/10 \)$ explains $\frac{\sigma_1}{\sigma_1 + \sigma_2}$ variance of original portfoli.

In here, matrix V is called singular matrix which consists of n singular vectors (Eigenvector). The left singular vectors U come from the rows of A, and the right singular vectors V come from the columns of A. This means that The left singular vectors U correspond to the domain (input) of the transformation, while the right singular vectors V correspond to the codomain (output).


## How this can be used in investment portfolio
For our application in investment portfolio, which consists of time series data, formulation of SVD will follow below. We are going to use three years ETF return data ($ 250 \times 3 = 750$). And there are 10 5 ETFs in our portfolio. So, matrix A will have $(750 \times 5)$. Graphically, it would be like this.

![SVD](/assets/img/post_image/ML/SVD/picture1.png)


The power of Singular Value Decomposition (SVD) shines in the process of reconstructing a decomposed matrix.

Using only \( p \) singular values, we can approximate the matrix \( A' \) from the original matrix \( A \), which was decomposed into \( U, $\Sigma$, $V^{T}$ \). As mentioned earlier, since the amount of information in \( A \) is determined by the magnitude of the singular values, even with only a few large singular values, we can retain a significant amount of useful information. $A \approx {U_p} \Sigma_p {V_p}^T$

![SVD](/assets/img/post_image/ML/SVD/picture2.png)


## Application in Portfolio
Let's use this idea in investment portfolio. We are going to use 10 US sector ETFs.
```yaml
The Technology Select Sector SPDR Fund (XLK)
The Communication Services Select Sector SPDR ETF Fund (XLC)
The Health Care Select Sector SPDR Fund (XLV)
The Energy Select Sector SPDR Fund (XLE)
The Consumer Discretionary Select Sector SPDR Fund (XLY)
The Industrial Select Sector SPDR Fund (XLI)
The Materials Select Sector SPDR Fund (XLB)
The Financial Select Sector SPDR Fund (XLF)
The Utilities Select Sector SPDR Fund (XLU)
The Consumer Staples Select Sector SPDR Fund (XLP)
```

code will be added after the deadline

