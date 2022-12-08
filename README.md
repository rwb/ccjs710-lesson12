# Lesson 13 - Thursday 12/2/21

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
* Tobit models are quite sensitive to departures and will generally yield inconsistent (biased) estimates when assumptions are not met.
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
