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

A Monte Carlo risk model typically phrases each instrument’s return in terms of a set of market factors. For example, one could use a simple linear model: instrument returns are calculated as a weighted sum of the values of the market factors. Different weights are chosen for each instrument for each market factor. The model is fitted for each instrument with a regression using historical data. On top of this, one can allow the instruments to have some optionality – each can be parameterized with a minimum and maximum value. This adds an element of non-linearity that the variance-covariance method has trouble handling, but which the Monte Carlo method can process with relative ease.

It is possible to choose more complicated models, perhaps incorporating domain specific knowledge.  The per-instrument model-fitting step of the computation is well suited to Spark.  The process chosen is a simplier model.

The model also needs a process for simulating the behavior of market factors. A simple assumption is that each market factor follows a normal distribution. To capture the fact that market factors are often correlated – when NASDAQ is down, the Dow is likely to be suffering as well – multivariate normal distribution can be used with a non-diagonal covariance matrix. As above, a more complicated method of simulating the market or a different distribution for each market factor can be assumed, perhaps one with a fatter tail.

To summarize, trial conditions are drawn from a multivariate normal distribution:



The value of a particular instrument in a particular trial is the dot product of the trial conditions and the instrument’s factor weightswi bounded by the instrument’s minimum and maximum value, ni and xi:



The portfolio’s value for the trial is the sum of all instrument values for that trial:


<pre><code>
Insert code

</code></pre>
