# Portfolio-optimization-on-Python overview #
This project implements a mean-variance portfolio optimization strategy using Python, aiming to maximize risk-adjusted returns, using the Sharpe ratio. The project was developed in my first-year of university for the 2025 UK Trading Competition and estimates expected returns, the covariance matrix of asset returns, constructs the efficient frontier and solves the optimization problem.



# Code and theory explained: 
At first, I imported historical market data for our position to find metrics such as daily returns, volatility and the covariance matrix.
Then, the PortfolioOptimization class is computed. It contains return estimation, risk estimation, portfolio performance and optimisation.
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

If we then compute this using matrices we can obtain:

$$
var(R_p)=w^T \Sigma w
$$

where $w$ is the weight vector $\Sigma$ is the covariance matrix. This matrix cointains in the diagonal elements the individual asset risk and in the off-diagonal matrices the covariances between asset pairs.

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

We include a risk-free asset, cash in this case with a return $r_f$ of 1% annually and zero volatility. 
In the code, the class defines the negative Sharpe ratio, $f(w) = -S(w)$,  because the minimize SciPy algorithm minimises the function. So, if we minimise the negative Sharpe ratio, we can maximise the Sharpe ratio:

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

The objective function is non-linear, smooth and not quadratic, so to solve this, the class has to use numerical optimization.

As a result, to solve this optimization problem I have decided to use the Sequential Least Squares Quadratic Programming (SLSQP) algorithm, since it is a non-linear constrained optimization.

What the algorithm does is it solves a simpler quadratic problem, by approximating the objective as a quadratic function and the constrained as linear functions. 
The objective function is approximated using a second order Taylor expansion and the constraints are approximated with a first order Taylor expansion: 

$$
min_{\Delta w}    \nabla f(w_k)^T \Delta w + frac{1} {2} \Delta w^T B_k \Delta w
$$

$$
s.t. c(w_k) + \nabla c(w_k) \Delta w = 0
$$

$$
0.01 \le h(w_k) + \nabla h(w_k) \Delta w \le 0.30
$$

$B_k$ is the approximation of the Hessian (the matrix containing the second order partial derivatives which indicates curvature) of the Lagrangian (since it is a constrained optimization problem), calculed via BFGS. 

The algorithm starts from a current point $x_0$. At that point, it approximates the constrained problem with simpler quadratic problems, moving step by step toward the minimum point of the negative Sharpe ratio, where improving the objective would violate constraints or fail to help.

After this is done, we plot the efficient frontier, by using a Monte-Carlo simulation, which generates 10,000 random portfolios, that allows us to plot the efficient frontier. The simulation helps visualize how various portfolios compare in terms of risk and return. Then the optimal portfolio is highlighted.

# Assumptions
The optimizer treats estimated means, like estimated returns, as truth, even though they are mostly random noise.
The use of variance to model risk is limited, since it treats price appreciation the same as drawdowns and it does not appropriately capture tail-risk.
The covariance matrix $\Sigma$ is estimated from historical data and it is still, even though covariances change constantly also according to market regimes.
