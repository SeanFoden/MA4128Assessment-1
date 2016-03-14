Apache Spark
=========
Financial Applications
----------
Over recent years there has been significant progress in the area of risk assessment using the creation financial management portfolios that take risk into account.  This area of study is relatively new and has only gathered momentum since the 2000's.  Events around the world have highlighted the need for investors to be able accurately assess risks they may undertake while trading.

Spark facilitates the calculation of computationally-intensive statistics such as VaR via the Monte Carlo method.
It allows an individual to better understand the risk characteristics of large portfolios, enablig them to compute it before executing trades to help make informed and immediate decisions.
It aims to utilize VaR (Value at risk) in order to better understand the potnetial risks involved when dealing with these large scale portfolios.

VaR is a simple measure of an investment’s risk that tries to provide a reasonable estimate of maximum probable loss over a particular time period. A VaR calculation considers a given confidence level: a VaR of US$1 million with a 95% confidence level means that we believe our investment stands only a 5% chance of losing more than US$1 million over the time period.

A number of different methods are used to calculate VaR.

1. Variance-covariance
2. Historical simulation
3. Monte Carlo simulation

It is worth noting that the Monte Carlo method **_isn't perfect_**.  The models for generating trial conditions and for inferring instrument performance from them must make simplifying assumptions, and the distribution that comes out won’t be more accurate than these models going in.

In its most general form the Monte Carlo method:
* Defines a relationship between market conditions and each instrument’s returns
* Poses “trials” consisting of random market conditions
* Calculates the portfolio loss for each trial, and uses the aggregated trial data to build up a profile of the portfolio’s risk characteristics.

# Using the Monte Carlo model

...A Monte Carlo risk model typically phrases each instrument’s return in terms of a set of market factors. For example, one could use a simple linear model: instrument returns are calculated as a weighted sum of the values of the market factors. Different weights are chosen for each instrument for each market factor. The model is fitted for each instrument with a regression using historical data. On top of this, we’ll allow our instruments to have some optionality – each can be parameterized with a minimum and maximum value. This adds an element of non-linearity that the variance-covariance method has trouble handling, but which the Monte Carlo method can take in stride.

It’s also worth mentioning that we could have chosen a more complicated model, perhaps incorporating domain specific knowledge.  While the per-instrument model-fitting step of the computation is a good fit for Spark as well, we’ll leave it out here for the sake of brevity.

Now that we have our model for calculating instrument losses from market factors, we need a process for simulating the behavior of market factors. A simple assumption is that each market factor follows a normal distribution. To capture the fact that market factors are often correlated – when NASDAQ is down, the Dow is likely to be suffering as well – we can use a multivariate normal distribution with a non-diagonal covariance matrix. As above, we could have chosen a more complicated method of simulating the market or assumed a different distribution for each market factor, perhaps one with a fatter tail.

<pre><code>
Insert code

</code></pre>
