# Lesson 12 - Thursday 12/8/22

* In tonight's class, we consider the topic of censoring.
* It is useful to begin by distinguishing between the concepts of censoring and truncation.
* Censoring means that we have outcome information about each unit in the population (and the sample) but some observations have more information to contribute to the analysis than others. Tonight, we deal with the case of censoring above a certain threshold. For example, if the threshold is *t*, then we fully observe *y* if *y < t*; however, if *y > t*, then that is all we know about those observations. This problems is generally called "right-hand censoring."
* Truncation means that we only have outcome information for a subset of the population (and the sample) and that units are only available for observation within some restricted range of the outcome variable; as an example, if one is studying criminal behaviors during a particular time period but only persons who have at least some criminal involvement are included, the units available for analysis are a truncated subset of the original population of interest. A problem that often arises with truncation is that we don't know how many units have been truncated.
* We begin our analysis tonight by simulating a dataset:

```R
set.seed(24786741)
x <- rbinom(n=1787,size=1,p=0.5)
y <- 3.5+0.7*x+rnorm(n=1787,mean=0,sd=1)
df <- data.frame(x,y)
head(df)
```

* Here is the output:

```Rout
> set.seed(24786741)
> x <- rbinom(n=1787,size=1,p=0.5)
> y <- 3.5+0.7*x+rnorm(n=1787,mean=0,sd=1)
> df <- data.frame(x,y)
> head(df)
  x        y
1 0 4.551576
2 0 2.414939
3 0 3.510994
4 1 3.062330
5 1 4.444217
6 0 3.906655
> 
```

* Next, we estimate a linear regression of *y* on *x*:

```R
summary(lm(y~1+x,data=df))
```

which yields the following table:

```Rout
> summary(lm(y~1+x,data=df))

Call:
lm(formula = y ~ 1 + x, data = df)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.4559 -0.6687 -0.0371  0.6808  3.9376 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)  3.49455    0.03345  104.48   <2e-16 ***
x            0.71170    0.04737   15.03   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 1.001 on 1785 degrees of freedom
Multiple R-squared:  0.1123,	Adjusted R-squared:  0.1118 
F-statistic: 225.8 on 1 and 1785 DF,  p-value: < 2.2e-16

> 
```

* These results imply that the mean of *y* when *x=0* is 3.495 while the mean of *y* when *x=1* is 3.495+0.712 = 4.207.
* Of course, the difference between mean(y|x=1) and mean(y|x=0) is 0.712 with a standard error of 0.047.
* How do we interpret the t-statistic?
* Now, we confirm these results by calculating the means for the two subsamples (defined by the *x* strata):

```R
df.x0 <- subset(df,x==0)
mean(df.x0$y)
df.x1 <- subset(df,x==1)
mean(df.x1$y)
```

* which yields the following results:

```Rout
> df.x0 <- subset(df,x==0)
> mean(df.x0$y)
[1] 3.494549
> df.x1 <- subset(df,x==1)
> mean(df.x1$y)
[1] 4.206244
> 
> 
```

* You can also generate a histogram to see the distribution of *y* using the hist(y) function.
* Next, we introduce the problem of censoring.

```R
df.c <- df
df.c$y[df.c$y>5.4] <- 5.4

df.c$cen <- rep(NA,nrow(df.c))
df.c$cen[df.c$y<5.4] <- 0
df.c$cen[df.c$y==5.4] <- 1
table(df.c$y,df.c$cen,exclude=NULL)
```

* This script gives us a long frequency table, which I have abbreviated below:

```Rout
> df.c <- df
> df.c$y[df.c$y>5.4] <- 5.4
> 
> df.c$cen <- rep(NA,nrow(df.c))
> df.c$cen[df.c$y<5.4] <- 0
> df.c$cen[df.c$y==5.4] <- 1
> table(df.c$y,df.c$cen,exclude=NULL)
                   
                      0   1
  0.488782028836149   1   0
  0.674063748985781   1   0
  0.701776806480459   1   0
  0.750385185196489   1   0
  0.893798643608993   1   0
  0.998790149216489   1   0
  1.02014821771641    1   0
  1.05672189876297    1   0
  *
  *
  *
  5.37972189799171    1   0
  5.38849898670617    1   0
  5.39194156752744    1   0
  5.39306834106191    1   0
  5.39441155552531    1   0
  5.39522093745613    1   0
  5.4                 0 138
  ```

* I would ask you to generate another histogram, hist(df.c$y), so you can see the censored distribution of *y*.
* Now, I will estimate a linear regression of *y* on *x* using the censored data:


```R
summary(lm(y~1+x,data=df.c))
```

which gives us the following results:

```rout
> summary(lm(y~1+x,data=df.c))

Call:
lm(formula = y ~ 1 + x, data = df.c)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.3913 -0.6299 -0.0030  0.7195  1.9171 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)  3.48288    0.03114  111.83   <2e-16 ***
x            0.65877    0.04410   14.94   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 0.9322 on 1785 degrees of freedom
Multiple R-squared:  0.1111,	Adjusted R-squared:  0.1106 
F-statistic: 223.1 on 1 and 1785 DF,  p-value: < 2.2e-16

> 
```

* Notice that the regression coefficient for *x* is now attenuated (compared to its value in the uncensored data).
* We can also divide the censored data into strata defined by the 2 levels of *x* and verify the regression:

```R
df.x0.c <- subset(df.c,x==0)
mean(df.x0.c$y)
df.x1.c <- subset(df.c,x==1)
mean(df.x1.c$y)
```

* Here is the output:

```Rout
> df.x0.c <- subset(df.c,x==0)
> mean(df.x0.c$y)
[1] 3.482878
> df.x1.c <- subset(df.c,x==1)
> mean(df.x1.c$y)
[1] 4.141652
>
```

* You should verify that the difference between the two means is equal to the regression coefficient for *x*.
* Now, we turn to a statistical model that formally attends to the censored nature of these data. I use the linear regression model estimates as starting values:

```R
library(maxLik)

tm <- function(parms) {

  a <- parms[1]
  b <- parms[2]
  c <- parms[3]

  mu <- a+b*df.c$x
  sigma <- exp(c)

  pdf <- dnorm(x=df.c$y,mean=mu,sd=sigma)
  cdf <- pnorm((df.c$y-mu)/sigma)
  loglike <- (1-df.c$cen)*log(pdf)+df.c$cen*log(1-cdf)
  return(loglike)
}

tf <- maxLik(tm,start=c(3.49455,0.771170,log(1.001)),
  method="BHHH",finalHessian="BHHH")
summary(tf)
```

* Here are the results:

```Rout
> library(maxLik)
> 
> tm <- function(parms) {
+ 
+   a <- parms[1]
+   b <- parms[2]
+   c <- parms[3]
+ 
+   mu <- a+b*df.c$x
+   sigma <- exp(c)
+ 
+   pdf <- dnorm(x=df.c$y,mean=mu,sd=sigma)
+   cdf <- pnorm((df.c$y-mu)/sigma)
+   loglike <- (1-df.c$cen)*log(pdf)+df.c$cen*log(1-cdf)
+   return(loglike)
+ }
> 
> tf <- maxLik(tm,start=c(3.49455,0.771170,log(1.001)),
+   method="BHHH",finalHessian="BHHH")
> summary(tf)
--------------------------------------------
Maximum Likelihood estimation
BHHH maximisation, 3 iterations
Return code 8: successive function values within relative tolerance limit (reltol)
Log-Likelihood: -2498.221 
3  free parameters
Estimates:
      Estimate Std. error t value Pr(> t)    
[1,]  3.492260   0.034297 101.825  <2e-16 ***
[2,]  0.712695   0.047527  14.995  <2e-16 ***
[3,] -0.003258   0.017842  -0.183   0.855    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
--------------------------------------------
> 
```

* Notice that these results are very close to the values we observed in the original linear regression on the uncensored data (although with ever so slightly higher standard errors).
* This model works because the data really did come from a normal, homoscedastic distribution.
* Tobit models are quite sensitive to departures from normality and will generally yield inconsistent (biased) estimates when assumptions are not met.
* Here is an [article](https://www.jstor.org/stable/1914031?seq=1#metadata_info_tab_contents) that does a good job of laying out the key issues surrounding the Tobit model.
* Contemporary statistical software usually has a canned procedure for estimating Tobit models. In R, we have to invoke the VGAM library:

```R
library(VGAM)
ctm <- vglm(y~1+x,tobit(Upper=5.4),data=df.c)
summary(ctm)
```

which yields the following results:

```Rout
> library(VGAM)
Loading required package: stats4
Loading required package: splines
> ctm <- vglm(y~1+x,tobit(Upper=5.4),data=df.c)
> summary(ctm)

Call:
vglm(formula = y ~ 1 + x, family = tobit(Upper = 5.4), data = df.c)

Coefficients: 
               Estimate Std. Error z value Pr(>|z|)    
(Intercept):1  3.492257   0.033414 104.514   <2e-16 ***
(Intercept):2 -0.003258   0.017946  -0.182    0.856    
x              0.712685   0.047583  14.978   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Names of linear predictors: mu, loglink(sd)

Log-likelihood: -2498.221 on 3571 degrees of freedom

Number of Fisher scoring iterations: 6 

No Hauck-Donner effect found in any of the estimates

> 
```

* Of course, the real problem here is that we are trying to develop an inference about the outcome distribution difference between two groups (x=0 and x=1) when *y* is censored. An issue that further complicates the situation is that the *y* values tend to be higher for those cases where *x=1* so, it follows that we have more censoring in one group than in the other. We can create a table to see this:

```R
table(df.c$cen,df.c$x,exclude=NULL)
```

and here are the results:

```Rout
> table(df.c$cen,df.c$x,exclude=NULL)
   
      0   1
  0 874 775
  1  22 116
>
> 116/(775+116)
[1] 0.1301908
> 22/(874+22)
[1] 0.02455357
> 
```

* So, we can see that the censoring rate is about 5-6 times higher for the *x=1* group than for the *x=0* group. This is a partial source of the bias in our estimated difference between the mean values of *y* for each *x* group.
* A useful way to proceed in this setting begins with the realization that fewer than 50% of the sample is censored within each *x* group. Since all the censoring in this problem is right-hand censoring, the medians for each group are identified (and, therefore, the difference between the two medians is identified):

```R
median(df.x0.c$y)
median(df.x1.c$y)
median(df.x1.c$y)-median(df.x0.c$y)
```

* Here is the output:

```Rout
> median(df.x0.c$y)
[1] 3.469414
> median(df.x1.c$y)
[1] 4.151299
> median(df.x1.c$y)-median(df.x0.c$y)
[1] 0.6818856
> 
```

* As an exercise, compare these results to those from the original dataset to verify you get the same results.
* A question that arises is how to test for the significance of the difference between the medians for the two groups?
* We will use the bootstrap to investigate this issue.
* To build intuition about the bootstrap, we will first use it to test for the difference between the two group means.

```R
delta.mn <- vector()

for(i in 1:100000){
  j <- sample(1:nrow(df.c),size=nrow(df.c),replace=T)
  b <- df.c[j,]
  bx0 <- subset(b,x==0)
  bx1 <- subset(b,x==1)
  ybar0 <- mean(bx0$y)
  ybar1 <- mean(bx1$y)
  delta.mn[i] <- ybar1-ybar0
  }

mean(delta.mn)
sd(delta.mn)
quantile(delta.mn,0.025)
quantile(delta.mn,0.975)
t.test(df.x1.c$y,df.x0.c$y)
```

* Here are the results:

```Rout
> delta.mn <- vector()
> 
> for(i in 1:100000){
+   j <- sample(1:nrow(df.c),size=nrow(df.c),replace=T)
+   b <- df.c[j,]
+   bx0 <- subset(b,x==0)
+   bx1 <- subset(b,x==1)
+   ybar0 <- mean(bx0$y)
+   ybar1 <- mean(bx1$y)
+   delta.mn[i] <- ybar1-ybar0
+   }
> 
> mean(delta.mn)
[1] 0.6586611
> sd(delta.mn)
[1] 0.04401594
> quantile(delta.mn,0.025)
     2.5% 
0.5720082 
> quantile(delta.mn,0.975)
    97.5% 
0.7450124 
> t.test(df.x1.c$y,df.x0.c$y)

	Welch Two Sample t-test

data:  df.x1.c$y and df.x0.c$y
t = 14.938, df = 1783.6, p-value < 2.2e-16
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 0.5722800 0.7452686
sample estimates:
mean of x mean of y 
 4.141652  3.482878 

> 
```

* Next, we turn to the difference between the two group medians:

```R
delta.md <- vector()

for(i in 1:100000){
  j <- sample(1:nrow(df.c),size=nrow(df.c),replace=T)
  b <- df.c[j,]
  bx0 <- subset(b,x==0)
  bx1 <- subset(b,x==1)
  mx0 <- median(bx0$y)
  mx1 <- median(bx1$y)
  delta.md[i] <- mx1-mx0
  }

mean(delta.md)
sd(delta.md)
quantile(delta.md,0.025)
quantile(delta.md,0.975)
```

* Notice the parallel structure of this script compared to the one we used above for the means.
* Here are the results:

```Rout
> delta.md <- vector()
> 
> for(i in 1:100000){
+   j <- sample(1:nrow(df.c),size=nrow(df.c),replace=T)
+   b <- df.c[j,]
+   bx0 <- subset(b,x==0)
+   bx1 <- subset(b,x==1)
+   mx0 <- median(bx0$y)
+   mx1 <- median(bx1$y)
+   delta.md[i] <- mx1-mx0
+   }
> 
> mean(delta.md)
[1] 0.6660716
> sd(delta.md)
[1] 0.06219054
> quantile(delta.md,0.025)
     2.5% 
0.5480255 
> quantile(delta.md,0.975)
    97.5% 
0.7911954 
> 
```

* This gives us a 95% confidence interval for the difference between the two medians and since: (1) the censoring fraction is less than 50% for the 2 groups; and (2) the only censoring is on the right-hand side of the distribution of *y*, the medians and the difference between the medians is identified. The bootstrap gives us a principled way to measure the statistical significance of the difference between the 2 medians.


### Final Exam - Due by end of the day on Friday 12/16/22

Part 1: Logistic Regression

```r
set.seed(4217)
a <- round(16+abs(rnorm(n=5722,mean=0,sd=20)),0)
m <- c(rep(1,4989),rep(0,5722-4989))
u <- runif(n=5722,min=0,max=1)
e <- log(u/(1-u))
s <- -0.366302+0.665119*m-0.033536*a+0.007317*a*m+e
y <- rep(0,5722)
y[s>0] <- 1
df <- data.frame(y,m,a)
```

1. Use this dataset to check for age differences (a) between males (m=1) and females (m=0).

2. Estimate a logistic regression model for *y* with age as the only predictor variable. Interpret the logistic regression coefficient for age.

3. Calculate the mean and median of age for the sample. Calculate and interpret the derivative of the logistic response function at the mean and median of age for the sample.

4. Estimate a logistic regression model with main effects for age and sex plus an interaction term with age multiplied by sex. 

5. Using the mean and median of age for the sample you calculated in problem 3 above, calculate the derivatives of the logistic response function separately for the males and females at those ages. 

6. Identify the 95% confidence interval of the difference between the male and female derivatives at the sample mean and median ages. Comment on whether these 95% confidence intervals overlap with each other.

7. Create a histogram and boxplot showing the sampling distribution of the difference between the male and female derivatives at the sample mean of age; carry out the same calculations for the sample median of age.

Part 2: Binomial (bounded count) regression

```r
set.seed(031840942)
x <- rnorm(n=1000,mean=0,sd=0.8)
y <- rbinom(n=1000,size=10,prob=exp(x)/(1+exp(x)))
df <- data.frame(x,y)
table(y)
```

1. Estimate a properly specified binomial regression model to study the joint distribution of x and y.

2. Use the statistical model results to calculate the predicted probability of an event occurring in a single Bernoulli trial (p) as a function of x. Create a plot showing how p is expected to vary across the range of x.

2. Check on the adequacy of the statistical chi-square goodness of fit test.

3. Calculate the expected value of y across the range of x. Create a plot showing the relationship we expect to see between x and E(y) based on the statistical model.

Part 3: Unbounded count regression

```r
set.seed(03184092)
x <- rnorm(n=1750,mean=0,sd=1)
z <- ifelse((0.3*x+rnorm(n=1750,mean=5,sd=1))>5,1,0)
w <- runif(n=1750,min=0,max=1)
y <- rpois(n=1750,lambda=exp(-0.3+0.76*x+0.32*z+0.1*x*z+0.3*w))
df <- data.frame(x,z,y)
```

1. Assess the inadequacy of the linear model.
2. Measure overdispersion in the dataset.
3. Estimate Poisson, geometric, and negative binomial regression looking at the covariates x and z and the outcome variable y.
4. Study the fit of each of these distributions considering log-likelihood, AIC, and BIC metrics.
5. Graph the predicted values of the rate parameter, lambda, as a function of the covariates.

Part 4: Censored data

```r
set.seed(24786741)
x <- rbinom(n=2371,size=1,p=0.5)
y <- 3.2+0.8*x+rnorm(n=2371,mean=0,sd=1)
df <- data.frame(x,y)
```

1. Estimate a linear regression of y on x.
2. Set the censoring threshold to 5.5.
3. Estimate a second linear regression based on the censored data.
4. Use the VGAM library to estimate a tobit model using the censored data.
5. Calculate the fraction of censored cases for each level of x.
6. Check on the difference between the median of y across the 2 levels of x.
7. Use the bootstrap to calculate a 95% confidence interval for the difference between the medians.


### Exam-Related Issues

1. 12/9/22 (9:33am): a student came to me and noted that the ages were not in integer format for part 1. I have slightly modified the data creation code to ensure that the ages are now in an integer format. 

2. 12/9/22 (11:02am): a student pointed out that for Part 1, problem 4 the question originally pertained to a main-effects only model; in class (lesson 8), however, I had included an interaction term. I have now changed problem 4 to include both the main effects and the interaction term so the problem is similar to what we worked on previously.

3. 12/11/22 (8:42pm): a student pointed out that the original code for Part 2 produced a *y* distribution ranging from 5-10. This is confusing given that we did not work with an example like this in class. I've replaced this code with code that ensures that the full range of *y* is populated which is more similar to what we did in class. 

4. 12/12/22 (2:05pm): this is a clarification: a student indicated that there is a question about section 3 since it is possible the negative binomial model is not the best fitting model. The question is around problem 5 where you are asked to "Graph the predicted values of the rate parameter, lambda, as a function of the covariates." My response to this question is that the calculation of the rate parameter, lambda, is the same regardless of which model you use (i.e., the Poisson, geometric, and negative binomial models all use the same formula to calculate the rate parameter, lambda).
