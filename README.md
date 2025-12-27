# Portfolio-optimization-on-Python
This project implements a mean-variance portfolio optimization strategy using Python, aiming to maximize risk-adjusted returns, using the Sharpe ratio. The project was developed for the 2025 UK Trading Competition and estimates expected returns, the covariance matrix of asset returns, constructs the efficient frontier and solves the optimization problem.

At first, I imported historical market data for our position to find metrics such as daily returns, volatility and the covariance matrix.
Then, the PortfolioOptimization class is computed. It contains return estimation, risk estimation, portfolio performance and optimisation.
Firstly, the class estimates inputs such as returns, expected returns and the covariance matrix. Secondly, the class measures the portfolio's performance by computing expected returns and risks. The expected return of the portfolio is calculated as the weighted average of the individual asset returns, where the weight of each asset in the portfolio is denoted by $w_i$ and $E(R_i)$ is the expected return of asset i: 

$$
E(R_p) = \sum_{i=1}^{n} w_i E(R_i)
$$

The risk of the portfolio is measured as the variance of the portfolio, which is given by the individual risk of positions and the pair-wise co-movements of the holdings. As a result, it is given by these formulae:

$$
\sigma_p^2 = \sum_{i=1}^{n} \sum_{j=1}^{n} w_i w_j \sigma_{ij}
$$

Or using matrix notation:

$$
\sigma_p^2 = w^T \Sigma w
$$

where $w$ is the weight vector $\Sigma$ is the covariance matrix. This matrix cointains in the diagonal elements the individual asset risk and in the off-diagonal matrices the covariances between asset pairs.

Every possible weight vector maps to a point:

$$
(\sigma_p,E(R_p))
$$

The collection of all such points forms a cloud, with a boundary which is the efficient frontier.

There are infinitely many points on the efficient frontier. So the optimizer needs a selection rule. In this case the rule is that the weight vector needs to maximise the risk-adjusted returns: the Sharpe ratio.

As a result, at this point, the class computes the Sharpe ratio: $Sharpe = (E(R_p)-R_f)/\sigma_p$. 
In the Sharpe ratio, we include a risk-free asset (cash) with a return of 1% annually and zero volatility. This asset serves as a benchmark for risk-free investments and allows us to assess how well our portfolio performs relative to a risk-free option
The class returns the negative value because the minimize SciPy algorithm minimises the function. So, if we minimise the negative Sharpe ratio, we can maximise the Sharpe ratio. 

Then, the class runs the Optimization, with the constraints that the weights sum to 1, since there is fixed capital, and that the weights are bound between 0.01 and 0.30. The optimizer uses the Sequential Least Squares Quadratic Programming (SLSQP) algorithm. This works by starting from a current point $x_0$. At that point, it approximates the constrained problem with simpler quadratic problems, moving step by step toward the optimum point where improving the objective would violate constraints or fail to help.

After this is done, we plot the efficient frontier, by using a Monte-Carlo simulation, which generates 10,000 random portfolios, that allows us to plot the efficient frontier. The simulation helps visualize how various portfolios compare in terms of risk and return. Then the optimal portfolio is highlighted.
