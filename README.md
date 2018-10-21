# Study-V003-MCMC-Python-R

> Random_Variable
<img src="https://user-images.githubusercontent.com/31917400/47187126-d62a8c80-d32a-11e8-90e2-a361686a2981.png" />

> Distribution families
<img src="https://user-images.githubusercontent.com/31917400/47054048-be76cb00-d1a7-11e8-8fbd-db042248333e.png" />

> Estimating the AVG & VAR
<img src="https://user-images.githubusercontent.com/31917400/47054380-5923d980-d1a9-11e8-8226-a2a5b1991117.png" />

> Likelihood
<img src="https://user-images.githubusercontent.com/31917400/47054381-5b863380-d1a9-11e8-98f0-08cd8524d5f0.png" />

> Estimating Maximum Likelihood 
<img src="https://user-images.githubusercontent.com/31917400/47054860-3d6e0280-d1ac-11e8-948e-da71c188bb54.png" />

> In bayesian approach, prediction is a `weighted AVG of output` of our model for all possible values of parameters while in frequentist approach, prediction is finding the best-fitted values of `parameters`. 

## Intro to Monte-Carlo
Monte-Carlo methods are methods for `generating random variables` directly or indirectly from a target distribution. 
 - **Applications**: 
   - 1> hypothesis testing 
   - 2> Bayesian computation
 - Monte Carlo is not strictly Bayesian nor strictly frequentist(Neyman-Pearson hypothesis testing), but it's **Parametric** sampling or bootstrapping (target distribution part of a family of distributions) as a **statistical simulation**, which helps us understand reality without needing to run multiple experiments or costly calculations.
 - It also helps us understand probabilities of **extreme events**. 
 - The use of Monte-Carlo methods to **calculate p-values** has become popular because:
   - Many test statistics do not have a standard asymptotic distribution. Even if a standard asymptotic distribution does exist, it may not be reliable in realistic sample sizes. 
   - In contrast, Monte-Carlo methods can be used to obtain an **Empirical p-value** that approximates the exact p-value without relying on asymptotic distributional theory or exhaustive enumeration. 

__Example 1> hypothesis testing (in R):__ Contingency table with too small sample_size
```
study = matrix(data = c(21, 2, 15, 3), nrow = 2, ncol = 2, byrow = TRUE,
               dimnames = list(c("surgery", "radiation"), c("controlled", "not controlled")))
```
<img src="https://user-images.githubusercontent.com/31917400/47085511-bf8f1300-d20e-11e8-91f8-d8b517cc484a.png" />

 - The contingency table above shows results of patients who underwent cancer treatment and either saw their cancer controlled or not. The two treatments are "surgery" and "radiation". 
 - The question we are interested in is **"Q. Is there a difference between treatment and controll of cancer?""**. Of course, a Chi-squared test would usually be used for this type of analyses.
   - There are two ways that the Chi-squared test is used:
     - 1> test the **Goodness of fit** of the `theoretical distribution` to the `observations`.
     - 2> test for **independence** between different factors(row, col)??

 - **A disadvantage of the Chi-squared test** is that it requires a `sufficient sample size` in order for the chi-square approximation to be valid. When `cell_counts` are low (below 5), asymptotic properties do not hold well. Therefore, a simple Chi-squred test may report an **invalid p-value** which would increase a **Type-I, II error** rate. 
## Here, Monte-Carlo Method can solve this issue. 
   - **Simulating contingency tables**: This function below takes some characteristics of contingency table and generates lots of contingency tables, creating a distribution of those tables. Then we can use it to calculate the Chi-squred statistics. 
   - Here, `r2dtable(n, r, c)` refers **Random 2-way Tables with Given Marginals**  where 
     - `n`: giving the **number of tables** to be drawn.
     - `r`: giving the **row totals**, to be coerced to integer. Must sum to the same as c.
     - `c`: giving the **column totals**, to be coerced to integer.
```
simulateChisq <- function(B, E, sr, sc){
    results = numeric(B)
    for(i in 1:B){
        dataa = unlist(r2dtable(1, sr, sc))   
        M = matrix(dataa, ncol = length(sc), nrow = length(sr))
        Chi_val = sum(sort( (M - E)^2 / E, decreasing = TRUE))
        results[i] = Chi_val
    }
    return(results)
}

ChisqTest <- function(data, Simulations){
    x = data                                           ## data
    B = Simulations                                    ## number of simulations to generate
    n <- sum(x)                                        ## total number of observations
    sr <- rowSums(x)                                   ## sum of rows
    sc <- colSums(x)                                   ## sum of cols
    E <- outer(sr, sc, "*")/n                          ## ORDER MATTERS
    dimnames(E) <- dimnames(study)
    tmp <- simulateChisq(B, E, sr, sc)                 ## simulated data
    Stat <- sum(sort((x - E)^2/E, decreasing = TRUE))  ## chi^2 statistic
    pval <- (1 + sum(tmp >=  Stat))/(B + 1)            ## MC p-value
    rawPVal = pchisq(q = Stat, df = 2, lower.tail = FALSE)
    
    out = list(PearsonStat = Stat, MonteCarloPVal = pval, rawPVal = rawPVal)
    return(out)
}
```
<img src="https://user-images.githubusercontent.com/31917400/47089124-fa497900-d217-11e8-9fee-8f89523dfa9a.png" />

__Example 2> Bayesian Computation (in R)__
> This is the traditional way of Inferencing on a single proportion as a population parameter without Bayesian.
 - Let's say, a simple random sample of 1,028 US adults in March 2013 found that 56% support nuclear arms reduction. Damn + 6% !!! **"Q. Does this provide convincing evidence that a majority of Americans supported nuclear_arms_reduction at the 5% significance level?"**
 - Using a **Pearson-frequentist perspective**, we might simply do the following:
   - the number of US people supporting nuclear_arms_reduction(X) follows ~ Bin(n, p), and when `n` goes to infinity, our X also follows ~ N(np, npq). We call this `Normal Approximation for Binomial Distribution`.  
   - the proportion 'p' of them (X/n) follows ~ N(p, pq/n)
<img src="https://user-images.githubusercontent.com/31917400/47097156-b743d180-d228-11e8-975d-36c5e808ec3a.png" />

 - Under the Null-hypothesis, `p0 = q0`, so `np0 = nq0`, then `1028*0.5 = 514 > 10` which is the **mean** number of people supporting nuclear_arms_reduction.
 - Based on the normal model, the test statistic can be computed as the Z-score of the point estimate: `Z = (p_obv - p0)/SE`.  
 - SE can be computed: `SE = sqrt(p0*q0/n)` and the Null-hypothesis `p0 = 0.5` is used again here because this is a hypothesis test for a single proportion `sqrt(0.5*0.5/1028) = 0.016`, so our Z is `(0.56-0.5)/0.016 = 3.75`.
 - p-value is `1 - pnorm(q=3.75) = 8.841729e-05`. We can then look up the upper tail area, the p-value, and see that it is less than 0.001. With a `p-value < 0.05`, we reject the null hypothesis and conclude that the **poll provides evidence that a majority (greater than 50%) of Americans supported the nuclear_arms_reduction**. The 95% CI for `p_obv` would be `0.56 + c(-1,1)*1.96*0.016`. So on AVG, around from 52% to 59% of US people support the nuclear_arms_reduction. 

> Another perspective on this problem(Inferencing on a single proportion as a population parameter) is that of a Bayesian. 
<img src="https://user-images.githubusercontent.com/31917400/47188651-d463c780-d330-11e8-85b3-5e5d5d34acc9.png" />

 - Look, we have a likelihood which is Binomial. 
 - `Beta` is a conjugate prior for the `Binomial` likelihood. That's why we choose Beta as our prior...then what the posterior will be? 
 - If the posterior distributions `p(θ|x)` are in the same **distribution family** as the prior distribution `p(θ)`:
   - the prior and posterior are then called **conjugate distributions**, 
   - the `**likelihood function**` is usually well-determined from a statement of the data-generating process.
 - Let's see our prior. Prior expresses one's beliefs about this quantity before some evidence is taken into account. Here, the prior could be the probability distribution representing the relative proportions of advocaters who will support nuclear_arms_reduction. 
   - we chose Beta(1,1) as our prior and this is equivalent to Unif(0,1) and this is a non-informative prior, which means we don't have any prior information to add to this model. 

__Q. So..for Binomial Likelihood, why choose "Beta" as a prior?__ how to elicit prior distribution?
 - Theoretically, a prior(the form of the conjugate prior can generally be determined by) is a **CDF for the parameter θ distribution**. In practice, based on likelihood we have, we choose a conjugate prior from a conjugate family that's sufficiently flexible such that a member of the family will represent our prior beliefs(of course in general, if one has enough data, the information in the data will overwhelm this invasion of prior). And **then any reasonable choice of prior will lead to approximately the same posterior**.
   - However, Notice!! there are somethings that can go wrong. In the Bayesian context, events with `P(θ)=0` will have `P(θ|y)=0`. And events with `P(θ)=1` will have `P(θ|x)=1`. Thus a good bayesian will not assign probability of `0` or `1` to any event that has already occurred or already known not to occur. 
   
# Scenario_01. No-data? "Estimate data points" (with respect to θ : **Prior Predictive Distribution** for X)
<img src="https://user-images.githubusercontent.com/31917400/47260255-aa84df00-d4af-11e8-9d2c-eee68bd26b2c.png" />

   - Before observe data points, we compute a prior predictive interval (such that 95% of new observations are expected to fall into it). It's an interval for the `data points` rather than an interval for parameter we've been looking at. Prior predictive intervals are useful because they reveal the `consequences of the θ` at the data (observation) level. See, our predictive distribution of `data points` is **marginal:** `P(x) = S P(θ,x)dθ = S P(x|θ)P(θ)dθ `. 
   - To find this data point intervals, we first work with `prior` **before we observe any data**. 
   - For example, Bin(n,θ): 
     - Flip a coin 'n' times and count the number of 'H' we see. This, of course, will depend on the coin itself. "What's the probability that it shows up 'H's?" which is referring a `θ` distribution. `X` for the number of 'H's(success), as `X` being the sum of y: `X = SUM(y...)` and as we go from '1 to n' of y which is each individual coin flip(HEAD: y=1, TAIL: y=0)...but now set this aside for a while.  
     - Let's start. As for the prior(θ,θ,θ,θ,...), if we first assume that **all possible coins are equally likely**(all same θ), then `p(θ) = 1 where {0 <= θ <= 1}`, which means the probability of θ: `p(θ)` will flat...over the interval from θ=0 to θ=1. We first assume our prior is `p(θ) = 1`. 
     - So now go back to `X` and ask "what's our **predictive distribution of X** (for the number of 'H'. of course, `X` can take possible values 0, 1, 2,..up to n). The **marginal**: `P(X) = S P(X|θ)P(θ)dθ = S P(θ,X)dθ` so if n=10, we have 
     <img src="https://user-images.githubusercontent.com/31917400/47243997-d2127380-d3eb-11e8-87e0-717f9f022b50.png" />

     - Because we're interested in X now, it's important that we distinguish between a binomial density and a Bernoulli density. So here we just care about the total count rather than the exact ordering which would be Bernoulli's. But **for most of the analyses we're doing, where we're interested in θ rather than x, the binomial and the Bernoulli are interchangeable** because Binomial distribution is a distribution of sum of i.i.d. Bernoulli random variables and their likelihoods are equivalent, thus we will get the same posterior at the end. But here we care about x for a predicted distribution so we do need to specify that we're looking at **binomial** because we're looking H-counts. 
     - If we `simplify` the pdf of our data points distribution, first recall that we can write `n! = gamma(n + 1)`, then this model look like a Beta density. `Z ~ Beta(a,b)`:
     <img src="https://user-images.githubusercontent.com/31917400/47245353-d725f180-d3f0-11e8-8695-59a63dead938.png" />
     
     - Because it's a Beta density, we know all densities integrate up to 1. Thus we see that **if we start with a uniform prior, we then end up with a discrete uniform predictive density for X**. If all possible coins or all possible probabilities(θ) are equally likely, then all possible X outcomes are equally likely. That's why when we choose Beta(1,1) as our prior and this is equivalent to Unif(0,1) and this is a non-informative prior. 
     <img src="https://user-images.githubusercontent.com/31917400/47245743-73042d00-d3f2-11e8-83d4-ccded86edcb5.png" />
### **Hey! we just found the form of the prior function!! which is `sth x Beta(a,b)` !!! 

> Note: **What is Beta(α,β)?** it represents a distribution of probabilities(Random Variable is probability values) that is, it represents all the possible values of a probability. 
 - Imagine we have a baseball player, and we predict what his season-long batting average `θ` will be. You might say we can just use his batting average so far- but this will be a very poor measure at the start of a season! Given our batting average problem, which can be represented with a `binomial(X: Sum of successes)`, the best way to represent these prior expectations is with the `Beta(X: probability)` - it's saying, before we've seen the player take his first swing, what we roughly expect his batting average to be. The domain of the Beta distribution is [0, 1], because X value is a probability. We may expect that the player's season-long batting average will be most likely around `0.27`, but that it could reasonably range from `0.21 to 0.35`. This can be represented with a Beta with parameters `α=81` and `β=219`(where α=No.success, β=No.failure and the mean of Beta is `α/(α+β)=0.27`) 
<img src="https://user-images.githubusercontent.com/31917400/47266266-efe6f200-d52b-11e8-96e4-3ad67d0d988d.png" />
 
### Next,
 - __prior -> posterior__
   - When our prior for a Bernoulli likelihood(such as `p(θ) = 1`) is a `uniform`, we get a beta posterior with hyper-parameter:  
   <img src="https://user-images.githubusercontent.com/31917400/47256574-970b5100-d47a-11e8-8e66-d182c48ac514.png" />
   - In fact, the uniform distribution is a `Beta(1,1)`, and any beta distribution is conjugate for the Bernoulli distribution.
   - In posterior, the hyper-parameters are transformed: `a + sum(x), b + n - sum(x)` 
   
# Scenario_02. Don't go with a flat prior! We have some data-point (with respect to θ: **Posterior Predictive Distribution** for X)
<img src="https://user-images.githubusercontent.com/31917400/47260902-f2a9fe80-d4bb-11e8-8c80-6944cbcdf2cf.png" />
 
   - What about after we've observed data? Suppose we observe, after one flip, we got a 'H' the first time. We want to ask, what's our **predicted distribution for the second flip(H or T)**, given that we saw a 'H' on the first flip? 
   - `P(y2|y1) = S P(θ|y1, y2)dθ = S P(y2|θ,y1)P(θ|y1)dθ = S P(y2|θ)P(θ|y1)dθ`: using posterior distribution instead of prior, and `'y1 and y2' is independent` so conditional is meaningless..so we take y1 out, then..
   <img src="https://user-images.githubusercontent.com/31917400/47248777-bb2c4b00-d404-11e8-9b5d-2c67ff7f3d24.png" />
   
   - We can see here, that the posterior is a combination of the information in the prior and the information in the data. In this case, our prior is like having two data points, one 'H' and one 'T'. Saying we have a uniform prior for θ, is actually equivalent to saying we have observed one 'H' and one 'T'. And then, when we do go ahead and observe one head, it's like we now have seen two heads and one tail, and so our posterior predictive distribution for the second flip, says, if we have two heads and one tail, then we have a probability of two-thirds of getting another head, and a probability of one-third of getting a tail. 

### posterior mean & sample size
 - The effective sample size(`a+b`) gives you an idea of how much data you would need to make sure that you're prior doesn't have much influence on your posterior. If `a+b` is small compared to `sample_size: n` (non-informative prior), then the posterior will largely just be driven by the data `X`. If `a+b` is large relative to `sample_size: n` (informative prior), then your posterior will be largely driven by the prior `θ`. 
 - posterior_mean is `(prior_weight*prior_mean) + (data_weight*data_weight)`
 <img src="https://user-images.githubusercontent.com/31917400/47261034-aca26a00-d4be-11e8-897a-a9270ac5d2cb.png" />
 <img src="https://user-images.githubusercontent.com/31917400/47261080-c09a9b80-d4bf-11e8-84ef-0f8910747894.png" />

### Find posterior
 - When a family of `conjugate priors` exists, choosing a prior from that family simplifies calculation of the posterior distribution.
 - `Parameters` of prior distributions are a kind of `hyperparameter`. For example, if one uses `Beta(a,b)` to model the distribution of the parameter `p` of Bernoulli, then:
   - `p` is a parameter of the underlying system (Bernoulli), and
   - `a` and `b` are parameters of the prior distribution (Beta); hence hyperparameters
 - Sometimes hyper-parameters themselves in prior have `hyper distributions` expressing beliefs about their values in the posterior. A Bayesian model with more than one level of prior like this is called a `hierarchical Bayes model`.  

table of conjugate distribution
<img src="https://user-images.githubusercontent.com/31917400/47190665-91a6ed00-d33a-11e8-8f51-c3ab391a4871.png" />

 - find the posterior
<img src="https://user-images.githubusercontent.com/31917400/47119888-9c8f4e00-d264-11e8-9846-a03b7cb95e4d.png" />

 - the Bayesian Data model is `y|θ ~ Bin(n,θ)` thus `θ ~ Beta(a,b)`
 - the resulting posterior is then `θ|y ~ Beta(y+a, n-y+b)`. **We can now simulate the posterior distribution**, to choose `θ` !
 - Did you find the posterior? then build a Credible Interval. 
   - In Confidence Interval, the true value(population_parameter) is not a random variable. It is a fixed but unknown quantity. In contrast, our estimate is a random variable as it depends on our data x. Thus, we get different estimates each time, repeating our study. 
   - In Credible Intervals, we assume that the true value(population parameter θ) is a random variable. Thus, we capture the uncertainty about the true parameter value by a **imposing a prior distribution** on the true parameter vector. <img src="https://user-images.githubusercontent.com/31917400/47217688-0ad92b00-d3a1-11e8-9e2e-9efd544efc06.png" />

   - Using bayes theorem, we construct the posterior distribution for the parameter vector by blending the prior and the data(likelihood) we have, then arrive at **a point estimate** using the posterior distribution(use the mean of the posterior for example). However, the true parameter vector is a random variable, we also want to know the extent of uncertainty we have in our point estimate. Thus, we construct a 95% credible interval such that the following holds: `P( l(θ)<=θ<=u(θ) ) = 0.95` 

```
N = 10^4
set.seed(123)
x = rbeta(n = N, shape1 = 576 + 1, shape2 = 1028 - 576 + 1)
d = density(x)
hist(x = x, probability = TRUE, main = "Beta Posterior Distribution",
     xlab = expression(theta), ylab = "Density",
     ylim = c(0,40), col = "gray", border = "white")
lines(x = d$x , y = d$y, type = "l", col = 2) # add chart
abline(v = median(x), lty = 3, col = "3")

print("Median: ")
print(quantile(x = x, probs = c(0.025, 0.5, 0.975)))
```
<img src="https://user-images.githubusercontent.com/31917400/47120410-7e2a5200-d266-11e8-9613-75a2fe8e6995.png" />

### other prior
 - __Normal__
   - If you are collecting normal data to make inferences about `μ`, but you don't know the true `σ^2` of the data, three options available to you are:
     - 1. Fix σ^2 at your best guess.
     - 2. Estimate σ^2 from the data and fix it at this value.
     - 3. Specify a prior for σ^2 and μ to estimate them jointly
   - Options 1 and 2 allow you to use the methods "pretending you know the true value of σ^2 and this leads to a simpler posterior calculation for μ.
   - Option 3 allows you to more honestly reflect your uncertainty in σ^2, thereby protecting against overly confident inferences. 
 

### Two catagories of prior
 - __Informative prior__
   - An informative prior is a prior that is not dominated by the likelihood and that has an impact on the posterior distribution. If a prior distribution dominates the likelihood, it is clearly an informative prior.
   - A reasonable approach is to make the prior a `normal distribution` with expected value equal to the given mean value, with variance equal to the given variance. 
   - pre-existing evidence which has already been taken into account is part of the prior and, as more evidence accumulates, the posterior is determined largely by the evidence rather than any original assumption, provided that the original assumption admitted the possibility of what the evidence is suggesting.
   
 - __Non-informative prior__
   - Roughly speaking, a prior distribution is non-informative if the prior is "flat". 
   - Non-informative priors can express "objective" information such as "the variable is positive" or "the variable is less than some limit". The simplest rule for determining a non-informative prior is the principle of indifference, which assigns equal probabilities to all possibilities, thus the use of an uninformative prior typically yields results which are not too different from conventional statistical analysis.
   - However, Non-informative priors are useful when 'stronger' priors would unjustifiably favor some hypotheses in a way that's inconsistent with your actual (lack of) knowledge/beliefs. 
   - Or Priors can also be chosen according to some principle, such as symmetry or maximizing entropy given constraints; examples are `Jeffreys' prior` for the Bernoulli random variable.
   - Jeffreys prior
     - Choosing a uniform prior depends upon the particular parameterization. For example, thinking about normal distribution. Suppose 
     <img src="https://user-images.githubusercontent.com/31917400/47270910-5e4aa500-d56a-11e8-8831-e8c25df1f9ca.png" />
     
     - ss


   
   
   
   
   
   
   
   
   
   
   
   
   
   
---------------------------------------------------------------------------------------------------------   
Now, here is the thing. We saw **direct simulation from a posterior distribution**. However, there are some posteriors that will not be as easily identifiable. 
### Monte-Carlo methods will be helpful for generating samples from difficult to sample target distributions.
How? by generating random number from target distributions through **transformation methods**??
 - Monte Carlo simulation uses random sampling and statistical modeling to mimic the operations of complex systems. 
   - Monte Carlo models a system as a series of probability density functions.
   - Monte Carlo repeatedly samples from the PDF.
   - Monte Carlo computes the statistics of interest.

















































































