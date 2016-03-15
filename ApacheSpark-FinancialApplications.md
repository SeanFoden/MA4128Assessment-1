Apache Spark
=========

![alt text]( http://spark.apache.org/images/spark-logo-trademark.png)

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

Other Considerations
-------------------

Interactive Analysis
So far we’ve presented the computation as a batch job; however, Spark also supports interactive settings. For example, analysts and traders might wish to see what happens when they tweak model parameters, filter the set of instruments considered to those matching some particular criteria, or add a trade that they’re about to execute into the mix. After broadcasting, Spark will keep the set of instruments in memory on every machine in the cluster, making them available for servicing interactive queries. If filtering on particular instrument attributes is routinely done, it might make sense to store the instruments as a map indexed by those attributes.

Huge Instrument Data
While it’s rare for a single portfolio to be too large to fit in entirety on every machine, working with huge portfolios composed of instruments like small business loans might require splitting up the portfolio data across machines.

In its most general form the Monte Carlo method:
* Defines a relationship between market conditions and each instrument’s returns
* Poses “trials” consisting of random market conditions
* Calculates the portfolio loss for each trial, and uses the aggregated trial data to build up a profile of the portfolio’s risk characteristics.

Using the Monte Carlo model
-----------------------------

A Monte Carlo risk model typically phrases each instrument’s return in terms of a set of market factors. For example, one could use a simple linear model: instrument returns are calculated as a weighted sum of the values of the market factors. Different weights are chosen for each instrument for each market factor. The model is fitted for each instrument with a regression using historical data. On top of this, one can allow the instruments to have some optionality – each can be parameterized with a minimum and maximum value. This adds an element of non-linearity that the variance-covariance method has trouble handling, but which the Monte Carlo method can process with relative ease.

It is possible to choose more complicated models, perhaps incorporating domain specific knowledge.  The per-instrument model-fitting step of the computation is well suited to Spark.  The process chosen is a simplier model.

The model also needs a process for simulating the behavior of market factors. A simple assumption is that each market factor follows a normal distribution. To capture the fact that market factors are often correlated – when NASDAQ is down, the Dow is likely to be suffering as well – multivariate normal distribution can be used with a non-diagonal covariance matrix. As above, a more complicated method of simulating the market or a different distribution for each market factor can be assumed, perhaps one with a fatter tail.

Running on Spark
-----------------

An issue with the Monte Carlo method is that it is computationally intensive. This means that getting accurate results for a large portfolio can require a large number of trials, and simulating each trial can be computationally involved. Spark proves its worth here.

It allows you to express parallel computations across many machines using simple operators. A Spark job consists of a set of transformations on parallel collections. We simply pass in Scala (or Java or Python) functions and Spark handles distributing the computation across the cluster.  It is also fault tolerant, so if any machines or processes fail while the computation is running, we don’t need to restart from scratch.

A general sketch of our computation looks like:

1. Broadcast our instrument data to every node on the cluster. While a large portfolio might consist of millions of instruments, the most memory this should take is in the 10s of gigabytes, which is easily enough to fit into main memory on modern machines.
2. Create a parallel collection (RDD) of seeds for our random number generators.
3. Create a new parallel collection of portfolio values under random trial conditions by applying a function to each seed that generates a set of random trial conditions, applies them to each instrument to calculate the its value under those conditions, and then sums over all instruments.
4. Find the boundary between the bottom 5% of trial values and the rest.
5. Subtract the portfolio at this boundary from the current value to find the value at risk.

The nameoffunction 'trialValues' takes all of the instruments and a number of trials to run, and spits out an array containing the portfolio values for each trial.


<pre><code>
def trialValues(seed: Long, numTrials: Int, instruments: Seq[Instrument],
      factorMeans: Array[Double], factorCovariances: Array[Array[Double]]): Seq[Double] = {
    val rand = new MersenneTwister(seed)
    val multivariateNormal = new MultivariateNormalDistribution(rand, factorMeans,
      factorCovariances)
 
    val trialValues = new Array[Double](numTrials)
    for (i <- 0 until numTrials) {
      val trial = multivariateNormal.sample()
      trialValues(i) = trialValue(trial, instruments)
    }
    trialValues
  }
 
  def trialValue(trial: Array[Double], instruments: Seq[Instrument]): Double = {
    var totalValue = 0.0
    for (instrument <- instruments) {
      totalValue += instrumentTrialValue(instrument, trial)
    }
    totalValue
  }
 
  def instrumentTrialValue(instrument: Instrument, trial: Array[Double]): Double = {
    var instrumentTrialValue = 0.0
    var i = 0
    while (i < trial.length) {
      instrumentTrialValue += trial(i) * instrument.factorWeights(i)
      i += 1
    }
    Math.min(Math.max(instrumentTrialValue, instrument.minValue), instrument.maxValue)
  }

</code></pre>

The Spark code that runs it is on a cluster.  This gives a collection of simulated losses over many trials  One can then compute the VaR by investigating what happens at the bottom 5%.

The simulation also contains far more information about risk characteristics of the portfolio.  This information can also be used to further ones understanding of the portfolio and its inherent risk.

Other Considerations
--------------------

* Interactive Analysis
Spark supports interactive settings.  One can tweak model parameters along with filtering the set of instruments considered to some particular criteria.  

* Huge Instrument Data
While it’s rare for a single portfolio to be too large to fit in entirety on every machine, working with huge portfolios composed of instruments like small business loans might require splitting up the portfolio data across machines.

Conclusion
------------
Spark can be utilised to for Monte Carlo simulations in financial applications.  It provides a flexible framework for leveraging the processing power of compute clusters that allow traders to investigate the level of risk in a given portfolio.

[Link to blog](https://blog.cloudera.com/blog/2014/07/estimating-financial-risk-with-apache-spark/)

[Link to Monte Carlo video](https://www.youtube.com/watch?v=3gcLRU24-w0)
