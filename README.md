# Portfolio-optimization-on-Python overview #
This project aims to maximize risk-adjusted returns, using the Sharpe ratio, by implementing a mean-variance portfolio optimization strategy using Python, The project was developed in my first year of university for the 2025 UK Trading Competition and estimates expected returns, the covariance matrix of asset returns, constructs the efficient frontier and solves the optimization problem.



# Code and theory explained: 
I imported and cleaned historical market data for each asset in the team's portfolio to find metrics such as daily returns, volatility and the covariance matrix.

Then, the PortfolioOptimization class is computed. It contains return estimation, risk estimation, portfolio performance and optimization.
Firstly, the class estimates inputs such as returns, expected returns and the covariance matrix. Secondly, the class measures the portfolio's performance by computing expected returns and risks. The expected return of the portfolio is calculated as the weighted average of the individual asset returns. The weight of each asset in the portfolio is denoted by $w_i$ and $E(R_i)$ is the expected return of asset i: 

$$
E(R_p) = \sum_{i=1}^{n} w_i E(R_i)
$$

using matrix notation:

$$
E(R_p) = w^T \mu
$$

where $w^T$ is the weight vector transposed and $\mu$ is the expected return vector, showing the expected return for each asset in the portfolio.

We then model the portfolio risk as the variance of the portfolio returns. This is given both by the individual variance of positions and by the pair-wise co-movement of the holdings.
Using the definition of variance we have that:

$$
var(R_p)=E[(R_p - E(R_p)]
$$

Simplyfiing the above, using matrices' notation, we obtain:

$$
var(R_p)=w^T \Sigma w
$$

where $w$ is the weight vector and $\Sigma$ is the covariance matrix. This matrix cointains in the diagonal elements the individual asset risk and in the off-diagonal matrices the covariances between asset pairs.

If we write the formula component-wise we have:

$$
\sigma_p^2 = \sum_{i=1}^{n} \sum_{j=1}^{n} w_i w_j \sigma_{ij}
$$

Every possible weight vector maps to a point:

$$
(\sigma_p,E(R_p))
$$

The collection of all such points forms a cloud, with a boundary which is the efficient frontier.

There are infinitely many points on the efficient frontier. So the optimizer needs a selection rule. In this case the rule is that the weight vector needs to maximise the risk-adjusted returns: the Sharpe ratio.

As a result, at this point, the class computes the Sharpe ratio: $Sharpe = \frac{E(R_p)-R_f}{\sigma_p}$. We can then write the Sharpe ratio as a function of the weight vector:

$$
S(w)=\frac{w^T \mu - r_f}{\sqrt{w^T \Sigma w}}
$$

We include a risk-free asset - cash in this case - with a return $r_f$ of 1% annually and zero volatility. 
In the code, the class defines the negative Sharpe ratio, $f(w) = -S(w)$,  because the minimize SciPy algorithm (SLSQP) minimises the function. So, if we minimise the negative Sharpe ratio, we can maximise the Sharpe ratio:

$$
min_{w} -S(w) = min_{w} f(w) = max_{w} S(w)
$$

The class then defines the constraints:

$$
w^T 1 = 1
$$ 

Where $1$ is a $n \times 1$ vector of ones. Since the sum of the weights are equal to one.

and

$$
0.01\le w_i\le 0.30
$$

where $w_i$ are the elements of $w$.

This means that the weights sum to 1, since there is fixed capital, and that the weights are bound between 0.01 and 0.30.

At this point, the class has defined the portfolio and the optimization problem:

$$
min_{w}   f(w)
$$

$$
s.t. w^T 1 = 1
$$

$$
w_i \in [0.01, 0.30]
$$

The objective function is non-linear, smooth and not quadratic. So - to solve this - the class has to use numerical optimization.

As a result, to solve this optimization problem, the best choice is to use the Sequential Least Squares Quadratic Programming (SLSQP) algorithm. This is because it can handle non-linear constrained optimization extremely well even with high dimensions $\mathbb{R}^n$.

The algorithm starts from a point $x_0$. At that point the algorithm uses Taylor's expansion to approximate the objective function as a quadratic and the constraints as a linear model. To do so, it approximates $\nabla f(w)$ using the method of finite differences and it approximates the curvature (the matrix containing all second order derivatives), by approximating the Hessian of the Lagrangian using BFGS. At the first point the curvature is chosen to be the identity matrix. 
So, the quadratic subproblem is:

$$
min_{\Delta w}    \nabla f(w_k)^T \Delta w + \frac{1} {2} \Delta w^T B_k \Delta w
$$

$$
s.t. c(w_k) + \nabla c(w_k) \Delta w = 0
$$

$$
0.01 \le h(w_k) + \nabla h(w_k) \Delta w \le 0.30
$$

After this is solved, a new gradient, Hessian and new weights' vector are obtained and another quadratic optimization problem is solved until the algorithm converges to the minimum of the function that respects the constraints.

After the algorithms finds the optimal weight vector, the code plots the efficient frontier using a Monte-Calro simulation: it generates 10,000 random portfolios, each with different weight vector,  showing for each potofolio its expected returns, variance and Sharpe ratio. The simulation helps visualize how various portfolios compare in terms of risk and return. Finally, the optimal portfolio is highlighted.

# Assumptions
The real world performance of this kind of optimization is limited. First of all, the estimated weights are highly sensitive to expected means,such as expected returns. However, expected returns change constantly, are regime-dependent and are mostly random noise. Moreover, a small mistake in the estimation of the mean level of returns can cause large changes in portfolio weights.
This could be improved by treating asset returns as a stochastic process with time-varying parameters rather than relying on fixed point estimates, using Bayesian time-series or regime-switching frameworks, allowing for parameter uncertainty and structural changes across market regimes.
In addition, the optimization problem is highly dependent on modelling risk as variance. This is limited, since it treats price appreciation the same as drawdowns and it does not appropriately capture tail-risk.
Furthermore, the optimization is over-reliant on the covariance matrix $\Sigma$, which is assumed to be still, even though covariances change constantly also according to market regimes.
A possible advancement is to treat covariance matrix and variance as varying and regime-dependent rather than fixed estimates
