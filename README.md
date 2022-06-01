# Advanced-Econometrics-Project
This repository contains a project conducted at university for the advanced econometrics course. The project has been conducted using R and then converted to RMarkdown.
---
title: "Advanced_Econometrics_Project_Final_Script"
author: "Stefano Luca Salvigni"
date: "6 May 2022"
output:
  pdf_document: 
    fig_width: 9
    fig_height: 7.5
    fig_caption: yes
    number_sections: yes
  word_document: 
    toc: yes
    fig_width: 8
    fig_height: 7
editor_options:
  markdown:
    wrap: 72
---

Goldsmiths College, University of London.

Advanced Econometrics.

# **TIME SERIES**

# 1. INTRODUCTION

```{r message=FALSE, warning=FALSE, include=FALSE}
#Load the required packages
library(dynamac)          # For ARDL estimation
library(forecast)         # For ARIMA model estimation
library(tseries)          # for time series models and diagnostic checks
library(nlme)             # to estimate ARIMA models
library(pdfetch)          # to fetch data directly from online data bases
library(zoo)              # for the zoo function for daily time series
library(urca)             # for unit root tests
library(vars)             # for Granger tests and VAR estimation
library(car)              # for regression diagnostics and hypothesis testing
library(dynlm)            # for Vector Error Correction Model(VECM)
library(tsDyn)            # for linear and non-linear VAR and VECM models
library(gets)             # for Isat function: step and impsulse indicator saturation
library(readxl)           # to read Excel files and load the data from Excel files
library(aod)              # for Wald tests
library(aTSA)             # Engle-Granger cointegration test
library(rmarkdown)        # Rmarkdown package
library(tinytex)          # To render the Latex version of your project
```

# 2. FETCHING THE DATA

```{r message=FALSE, warning=FALSE, include=FALSE}
#Load Inflation Rate - Monthly - (Percent Change from a Year ago)
CPI = pdfetch_FRED("CPIAUCSL")
names(CPI) = "CPI"

Inflation = diff(log(CPI), lag = 12) * 100                   
names(Inflation) = "Inflation"

Inflation = ts(Inflation, start=c(1947, 1), frequency=12)
Inflation = na.omit(Inflation)                            
```

```{r message=FALSE, warning=FALSE, include=FALSE}
#Capacity Utilization: Total Index (TCU) - Percent of Capacity,
#Seasonally Adjusted - Monthly - (from January 1967):
TCU = pdfetch_FRED("TCU")
names(TCU) = "TCU"
TCU = ts(TCU, start=c(1967, 1), frequency=12)
```

```{r include=FALSE}
#Create the data.set for the project
data.set = na.omit(
  ts.intersect(
    
    Inflation,
    TCU, 
    
    dframe=TRUE))

Inflation   = ts( data.set$Inflation,    start=c(1967, 1),  frequency=12)
TCU         = ts( data.set$TCU,          start=c(1967, 1),  frequency=12)
```

```{r include=FALSE}
#Skip this next line if using package 'dynamac' for ARDL models. Otherwise
#run it:
data.set = ts(data.set, start=c(1967, 1), frequency=12)   
```

Now we plot the two time series in levels. The first graph shows the
Inflation Rate, the second graph shows the Total Capacity Utilization,
while the third graph shows the correlation between the two variables
over time.

```{r}
#Plot the data in your project:
par(mfrow=c(3,1))
plot(Inflation, col=2)
plot(TCU, col=4)
plot(Inflation ~ TCU, col="orange")


cor(Inflation,TCU) 
```

The correlation coefficient is 0.3605283, and by also looking at the
graph, does not suggest a strong correlation. There seems to be a
positive trend, but further tests are needed to learn more about the
relationship between the two variables.

# 3. UNIT ROOT TESTS

Unit root tests are conducted to identify any potential unit root in the
variables which will make them not-stationary. This is considered a
problem in statistical modelling because the presence of unit root will
make the coefficient 'add word'

# 3.1 Model with variables in levels

Firstly, we conduct the tests on the variables in levels.

# 3.1.1 Inflation

```{r}
wl = Inflation                     
```

We plot the chosen variable:

```{r}
plot(wl, col=2)  
```

The next plot shows the correlations in levels between the current value
of Inflation and the the first 6 lags respectively:

```{r}
lag.plot(wl, 6, do.lines=FALSE, col="orange")   
```

The graph shows the there is a significant persistence as there is long
term correlation. In other words, the series suggest the presence of
strong autocorrelation over time. This might imply the presence of a
unit root. We now compute the auto-correlation functions(ACF) and the
partial auto-correlation functions(PACF). The dashed lines indicate the
bounds for statistical significance at the 10% level.

```{r fig.height=7, fig.width=9}
par(mfrow=c(2,2))
acf     (wl,          main="ACF for wl (levels)")
pacf    (wl,          main="PACF for wl (levels)")
```

We can notice a very strong persistence given by the first graph, while
the second graph suggests that inflation in level might be a
autoregressive process of order (I).

We now proceed with the three Unit Root tests (ADF, PP, KPSS) to
determine whether or not inflation in levels has a process of unit root.
We calculate the optimal maximum number of lags using the following
formula:

```{r}
max.lags = trunc(  (12 * (  (length(wl)/100)^(1/4)  ) ) )
max.lags
```

After having set the maximum number of lags, which according to the
Hayashi formula it is equal to 19 for our case, we now run each unit
root test individually. Each test will be run with a drift.

## Augmented Dickey-Fuller(ADF) test

ADF: Ho = Residuals have a unit root = Non-stationary

```{r}
wl.adf.drift <- ur.df(wl, selectlags="BIC", type="drift", lags=max.lags )
summary(wl.adf.drift)
plot(wl.adf.drift)
```

Do not reject the null hypothesis at the 5% as t-stat is -2.79, so the
series is not-stationary and has a unit root.

## Philips-Perron(PP) test

PP: Ho = Residuals have a unit root = Non-stationary

```{r}
wl.pp <- ur.pp(wl, type="Z-tau", model="constant", lags="long") 
summary(wl.pp)
plot(wl.pp)
```

The t-stats is -2.57, which fails to reject the null hypothesis of
non-stationarity at the 5%.

## KPSS test:

Deterministic components: constant "mu"; or a constant with linear trend
"tau". KPSS: Ho = Residuals do not have a unit root = Stationary
[opposite of ADF and PP]

```{r}
wl.kpss <- ur.kpss(wl, type="mu",  lags="long" )
summary(wl.kpss)


wl.kpss <- ur.kpss(wl, type="tau", lags="long" )
print(summary(wl.kpss))
plot(wl.kpss)
```

With a constant we reject the null hypothesis of no unit root at any
significance level, with tau we also reject at 5% but on the borderline
(t-stat 0.15>0.146).

## Summary

There are evidence that the residuals of the Inflation Time Series in
levels have a unit root and the series is not-stationary I(1) in levels.

# 3.1.2 TCU

```{r}
zl = TCU                            
```

Plot the chosen variable:

```{r}
plot(zl, col = 4) 
```

The next plot shows the correlations in levels between the current value
of TCU and the the first 6 lags respectively:

```{r}
lag.plot(zl, 6, do.lines=FALSE, col="orange")       
```

Once again, the graph shows the there is a significant persistence as
there is long term correlation. In other words, the series suggest the
presence of strong autocorrelation over time. This might imply the
presence of a unit root. We now compute the auto-correlation
functions(ACF) and the partial auto-correlation functions(PACF). The
dashed lines indicate the bounds for statistical significance at the 10%
level.

```{r}
par(mfrow=c(2,2))
acf     (zl,          main="ACF for zl (levels)")
pacf    (zl,          main="PACF for zl (levels)")
```

We can notice a very strong persistence given by the first graph, while
the second graph suggests that TCU in levels might be a autoregressive
process of order (I).

We now proceed with the three Unit Root tests (ADF, PP, KPSS) to
determine whether or not TCU in levels has a process of unit root. We
calculate the optimal maximum number of lags using the following
formula:

```{r}
max.lags = trunc(  (12 * (  (length(wl)/100)^(1/4)  ) ) )
max.lags
```

After having set the maximum number of lags, which according to the
Hayashi formula it is equal to 19 for our case, we now run each unit
root test individually. Each test will be run with a drift.

## Augmented Dickey-Fuller(ADF)

ADF: Ho = Residuals have a unit root = Non-stationary

```{r}
zl.adf.drift <- ur.df(zl, selectlags="BIC", type="drift", lags=max.lags )
summary(zl.adf.drift)

plot(zl.adf.drift)
```

We reject the null hypothesis of unit root at the 5%, but we fail to
reject it at the 1%. So we proceed with further tests.

## Philips-Perron(PP) test

PP: Ho = Residuals have a unit root = Non-stationary

```{r}
zl.pp <- ur.pp(zl, type="Z-tau", model="constant", lags="long") 
summary(zl.pp)
plot(zl.pp)
```

We reject the null hypothesis of unit root at all significance levels,
suggesting that TCU in levels is stationary.

## KPSS tests on the chosen series

Deterministic components: constant "mu"; or a constant with linear trend
"tau". KPSS: Ho = Residuals do not have a unit root = Stationary
[opposite of ADF and PP]

```{r}
zl.kpss <- ur.kpss(zl, type="mu",  lags="long" )
summary(zl.kpss)


zl.kpss <- ur.kpss(zl, type="tau", lags="long" )
print(summary(zl.kpss))
plot(zl.kpss)
```

We reject the null hypothesis of no unit root in the model with the
constant at all signinifance levels, but we fail to reject it in the
model with constant and linear trend at all significance levels.

## Summary

There are evidence that the residuals of the TCU Time Series in levels
might have a unit root, but the results of the tests are ambiguous and
inconsistent. We proceed by assuming that the series is not-stationary
I(1) in levels.

# 3.2 Model in first differences

We espect that the two series in first difference will not show signs of
presence of unit root.

# 3.2.1 Inflation

```{r}
w = diff(Inflation)                     
```

Plot of the chosen variable:

```{r}
plot(w, col = 2)    
```

Plot of the lag correlations in first differences:

```{r}
lag.plot((w), 6, do.lines=FALSE, col="lightblue")       
```

By running the model in first differences, the correlation is weakened
and the persistence disappears. This might imply that the process of
unit root might have been removed. Auto-correlation functions(ACF) and
Partial auto-correlation functions(PACF). The dashed lines indicate the
bounds for statistical significance at the 10% level.

```{r}
par(mfrow=c(2,2))
acf     ((w),    main="ACF for w (first difference)")
pacf    ((w),    main="PACF for w (first difference)")
```

As we can see above, the autocorrelation has been reduced significantly
by taking the first differences. There are still signs of
autocorrelation of the residuals, but later in the paper further tests
will be done.

We now proceed with the three Unit Root tests (ADF, PP, KPSS) to
determine whether or not Inflation in first differences has a process of
unit root. We calculate the optimal maximum number of lags using the
following formula:

```{r}
max.lags = trunc(  (12 * (  (length(w)/100)^(1/4)  ) ) )
max.lags
```

After having set the maximum number of lags, which according to the
Hayashi formula it is equal to 19 for our case, we now run each unit
root test individually. Each test will be run with a drift.

## Augmented Dickey-Fuller(ADF) test

ADF: Ho = Residuals have a unit root = Non-stationary

```{r}
w.adf.drift <- ur.df(w, selectlags="BIC", type="drift", lags=max.lags)
print(summary(w.adf.drift))
plot(w.adf.drift)
```

As expected, we reject the null hypothesis of unit root at every
significance level.

## Philips-Perron(PP) test

PP: Ho = series has a unit root = integrated I(1) process

```{r}
w.pp <- ur.pp(w, type="Z-tau", model="constant", lags="long") 
summary(w.pp)
plot(w.pp)
```

Even in this test we reject the null hypothesis of unit root at every
significance level.

## KPSS test

Deterministic components: constant "mu"; or a constant with linear trend
"tau". KPSS: Ho = series does not have a unit root = Stationary
[opposite of ADF and PP]

```{r}
w.kpss1 <- ur.kpss(w, type="mu",  lags="long" )
summary(w.kpss1)
plot(w.kpss1)

w.kpss2 <- ur.kpss(w, type="tau", lags="long" )
print(summary(w.kpss2))
```

In this case we fail to reject the null hypothesis of stationarity,
which confirm the previous results.

## Summary

Inflation in first differences is stationary I(0).

# 3.2.2 TCU

```{r}
z = diff(TCU)                            
```

Plot of the chosen variable:

```{r}
plot(z, col = 4)    
```

Plot of the lag correlations in first differences:

```{r}
lag.plot((z), 6, do.lines=FALSE, col="lightblue")       
```

By running the model in first differences, the correlation is weakened
and the persistence disappears. This might imply that the process of
unit root might have been removed. Auto-correlation functions(ACF) and
Partial auto-correlation functions(PACF). The dashed lines indicate the
bounds for statistical significance at the 10% level.

```{r}
par(mfrow=c(2,2))
acf     ((z),    main="ACF for z (first difference)")
pacf    ((z),    main="PACF for z (first difference)")
```

As we can see above, the autocorrelation has been reduced significantly
by taking the first differences. There are still signs of
autocorrelation of the residuals, but later in the paper further tests
will be done.

We now proceed with the three Unit Root tests (ADF, PP, KPSS) to
determine whether or not Inflation in first differences has a process of
unit root. We calculate the optimal maximum number of lags using the
following formula:

```{r}
max.lags = trunc(  (12 * (  (length(z)/100)^(1/4)  ) ) )
max.lags
```

After having set the maximum number of lags, which according to the
Hayashi formula it is equal to 19 for our case, we now run each unit
root test individually. Each test will be run with a drift.

## Augmented Dickey-Fuller(ADF) test

ADF: Ho = Residuals have a unit root = Non-stationary

```{r}
z.adf.drift <- ur.df(z, selectlags="BIC", type="drift", lags=max.lags)
print(summary(z.adf.drift))
plot(z.adf.drift)
```

Even in this test we reject the null hypothesis of unit root at every
significance level.

## Philips-Perron(PP) test

PP: Ho = Residuals have a unit root = Non-stationary

```{r}
z.pp <- ur.pp(z, type="Z-tau", model="constant", lags="long") 
summary(z.pp)
plot(z.pp)
```

As expected, we reject the null hypothesis of unit root at every
significance level.

## KPSS test

Deterministic components: constant "mu"; or a constant with linear trend
"tau". KPSS: Ho = Residuals do not have a unit root = Stationary

```{r}
z.kpss <- ur.kpss(z, type="mu",  lags="long" )
summary(z.kpss)


z.kpss <- ur.kpss(z, type="tau", lags="long" )
print(summary(z.kpss))
```

In this case we fail to reject the null hypothesis of stationarity,
whiich confirm the previous results.

## Summary

TCU in first differences is stationary I(0).

# 4. COINTEGRATION ANALYSIS

Let us plot the graph of the gap between the two series:

```{r}
g=Inflation - TCU
plot(g)
```

After having discovered that both time series show signs of unit root in
levels, but are stationary in first differences, we need to test for
cointegration in the residuals. We might find a vector of the linear
combination of the two series whose residuals might be stationary,
suggesting cointegration.

# 4.1 Optimal Lag Selection For Cointegration tests

First, we check the optimal lag length according to the information
criteria:

```{r}
#Using the "VARS" package
VARselect(data.set, lag.max=10, type="none",  season = NULL, exogen = NULL)$selection
VARselect(data.set, lag.max=10, type="const", season = NULL, exogen = NULL)$selection	  
VARselect(data.set, lag.max=10, type="trend", season = NULL, exogen = NULL)$selection	  
VARselect(data.set, lag.max=10, type="both",  season = NULL, exogen = NULL)$selection	  	             
```

According to the Hannon-Queen critirion, the optimal number of lags is
2.

```{r}
#Set the number of lags that will be used in our estimations
optimal.lags = 2
```

# 4.2 Engle-Granger Methodology

In the Engle-Granger Methodology, H0 = Residuals are no-cointegrated.

```{r}
coint.test(Inflation, TCU,           d = 0, nlag = NULL, output = TRUE)                             
```

The p-value of the Engle-Granger Cointegration test is lower than 1%
rejecting the null hypothesis of nocointegration.

## Summary

Inflation and TCU seem to be cointegrated in levels.

# 4.3 Johansen Test

Now let's see what the Johansen test says. Null hypothesis: Ho = no
cointegration (r=0) You need to reject the null to get cointegration
(r=1). r = number of cointegration relations

```{r}
#With constant in the cointegrating vector:
johansen.const = ca.jo(data.set, type="eigen", ecdet="const",  K = optimal.lags, spec="longrun")
summary(johansen.const)


johansen.const = ca.jo(data.set, type="trace", ecdet="const",  K = optimal.lags, spec="longrun")
summary(johansen.const)


plot(johansen.const)

plotres(johansen.const)
```

The t-tests in both tests are greater than the p-values at any
significance level. This reject the null hypothesis of zero
cointegrating vectors (r=0) but also reject the presence of only one
cointegrating vector (r=1). Since we only have two variables, it does
not make sense to have more than one cointegrating relations.

## Summary

The results of the tests are ambiguous and contrasting. Even though The
johansen test is more powerful than the Engle-Granger test, both tests
are lowpower test. For these reasons, we proceed by assuming that
Inflation and TCU are no cointegrated in levels, as no strong supporting
evidence for cointegration. Moreover, the residuals from the Johansen
cointegration tests seem not to be well behaved.

# 5. AUTO-REGRESSIVE DISTRIBUTED LAG (ARDL) MODEL: IMPACT OF TCU SHOCK ON INFLATION RATE

So far we know that the variables are non-stationary in levels but are
stationary in first differences. Moreover, since the cointegration tests
were inconclusive, we assumed that the variables are not cointegrated.

```{r}
#Recall the dataset to get rid of issue with "dynamac" package:
data.set = na.omit(
ts.intersect(

Inflation,
TCU, 

dframe=TRUE))
```

We now test how inflation rate reacts to shock in the weakly exogenous
variable TCU. We do not expect exhaustive results, as the inflation rate
depend on a multitude of different other variables, and cannot be
explained only by changes in the Total Capacity Utilization Rate.
Nevertheless, we can compute the long term effects of shock in the TCU.

```{r}
#Because stochastic values are involved in the simulations, we set a seed
#to ensure our results are replicable.
set.seed(123)

lags = 2
```

```{r}
ARDL = dynardl(	
Inflation ~ TCU,                                               
lags = list("TCU" = 1,          "Inflation" = 1             ),
diffs =    c("TCU"                                          ),
lagdiffs = list("TCU" = c(1:lags),  "Inflation" = c(1:lags)     ),
	  
ec = TRUE,
constant = TRUE,
trend = FALSE,
	  
simulate = TRUE,                                 
shockvar = "TCU",                              

range = 50,
sims = 1000,
fullsims = TRUE,
	  
data = data.set)
	
	
summary(ARDL)
```

The adjusted R squared is 0.20 whiich is not very high, giving first
hints about the significance of the impact of tCU on Inflation. We can
now check the long-run relationship between the two variables.

# 5.1 ARDL-bounds procedure

Since the Engle-Granger two-step method and Johansen likelihood-based
approach too often conclude cointegration when it does not exist, and
also in our case were ambiguous and contrasting, we test for the
Long-run relationship testing using the Pesaran, Shin, and Smith (2001)
ARDL-bounds procedure. If the F-statistic exceeds the upper I(1)
critical value there is evidence of cointegration; if the F-statistic
falls below the I(0) critical value then all regressors are in fact
stationary and hence no cointegration; if the F-statistic is between the
lower I(0) and upper I(1) critical values then we conclude that the
results are inconclusive, cointegration may exist, but further testing
and re-specification of the model is needed

```{r}
pssbounds(ARDL)
	
pssbounds(ARDL, restriction=TRUE)
```

In the first version of the test, the F-test is 7.19, which falls
between I[0] and I[1] at the 1% significance level, suggesting that
results are inconclusive. The t-test is -2.74, which is lower than I[0]
at the 1% significance level, so we do not reject the null hypothesis of
no cointegration. By turning the restriciton on, the test suggests no
cointegration, as the F-statistic of 4.90 is lower than I[0], but still
on the borderline. We now check for the behaviour of the residuals of
the ARDL model.

```{r}
lags = 2
	
dynardl.auto.correlated(ARDL) 
```

The Breusch-Godfrey test tests for Autocorrelation, with the null
hypothesis being Ho: No autocorrelation. The p-value is 0.073 so we do
not reject the null hypothesis of no autocorrelation at the 5%. The
Shapiro-Wilk test for Normality suggests that the residuals are not
normally distributed, as we reject the null hypothesis of normality in
the residuals at less than 1%.

# 5.2 ARDL Model Residuals and Additional Tests

We proceed then by computing additional tests on the residuals.

```{r}
ARDL.residuals =    ARDL$model$residuals
ARDL.residuals = ts(ARDL$model$residuals,  start=c(1967, 1) ,  frequency=1)
```

```{r}
par(mfrow = c(2, 2))
plot( ARDL.residuals )                 
abline( h=0, col="red" )
hist( ARDL.residuals, col="green")    
acf( ARDL.residuals )                 
pacf( ARDL.residuals )                 
```

The histogram of the residuals show that the residuals are not normally
distributed as the graph is slightly left skewed and there is a presence
of few outliers. By looking at the Autocorrelation and Partial
autocorrelation functions, we fail to conclude that there is
autocorrelation in the residuals.

```{r}
jarque.bera.test(ARDL.residuals)  # Jarque-Bera test for normal residuals
	
bds.test(ARDL.residuals)        # BDS test Ho: series of i.i.d. random variable
	
bptest(ARDL$model)              # Breusch-Pagan test against heteroskedasticity
```

The p-value in the Jarque Bera Test is close to 0, confirming that the
residuals are not normally distributed. The Breush-Pagan test for
heteroskedasticity suggests heteroskedasticity in the residuals, as the
p-value is smaller than 0.01. We can conclude that the residuals are not
well behaved.

# 5.3 ARDL Model with robust standard errors

Se the residuals are not normally distributed and present
heteroskedasticity, we can compute the ARDL model with robust standard
errors that correct for heteroskedasticity and auto-correlation.

```{r}
coeftest(ARDL$model, vcov.=vcovHAC) # Estimated coefficients with HAC standard errors
```

##Impulse-Response plots
We can now plot the impulse response functions
for the ARDL model. This is a counterfactual simulation of a response to
a shock in some exogenous variable.

```{r}
lags = 2
```

```{r}
ARDL = dynardl(	
Inflation ~ TCU,                                                

lags = list("TCU" = 1,          "Inflation" = 1             ),
diffs =    c("TCU"                                          ),
lagdiffs = list("TCU" = c(1:lags),  "Inflation" = c(1:lags)     ),
	  
ec = TRUE,
constant = TRUE,
trend = FALSE,
	  
simulate = TRUE,                                 
shockvar = "TCU",                               
range = 50,
sims = 1000,
fullsims = TRUE,
	  
data = data.set)
summary(ARDL)
```

```{r}
dynardl.all.plots(ARDL) # all plots together
```

By looking at the cumulative change in Inflation due to a one unit shock
in TCU, the graph suggests that there are weak evidence of a positive
cumulative impact that leads the inflation rate to rise over the 10
months following the shock.

##Summary 
These evidence are weak: as previously mentioned, the ARDL
model with only the Inflation and TCU is not specified correctly. The
residuals are not white noise With 2 lags, and the residuals are not yet
normally distributed. One possible solution would be tp add more
variables into the ARDL model that in conjuction with TCU can help
explaining the behaviour of inflation. We now proceed to test the VAR in
first differences with only two variables and assuming no cointegration.

# 6. BIVARIATE VAR(p) MODEL

Because previous tests suggest the series have unit root, but the tests
for cointegration do not provide enough strong evidence for
cointegration, we proceed in estimating the VAR in first differences.

```{r}
data.set = na.omit(
ts.intersect(

Inflation,
TCU, 

dframe=TRUE))
```

```{r}
#Run the model in first differences
diffInflation = diff(Inflation)
diffTCU = diff(TCU)

        
data.set2 = na.omit(
ts.intersect(

diffInflation,
diffTCU, 

dframe=TRUE))


diffInflation   = ts( data.set2$diffInflation,    start=c(1967, 2),  frequency=12)
diffTCU         = ts( data.set2$diffTCU,          start=c(1967, 2),  frequency=12)
data.set2=ts(data.set2, start=c(1967, 2), frequency=12)
```

```{r fig.height=8, fig.width=8}
par(mfrow=c(3,1))
plot(diffInflation, col =2)
plot(diffTCU, col = 4)
plot(diffInflation ~ diffTCU, col = "orange")

plot(data.set2, plot.type="single", col=1:ncol(data.set))
legend("topright", colnames(data.set), col=1:ncol(data.set), lty=1, cex=.65)
```

```{r}
#Check the optimal lag length according to the information criteria:
#Using the "VARS" package
VARselect(data.set2, lag.max=10, type="none",  season = NULL, exogen = NULL)$selection
VARselect(data.set2, lag.max=10, type="const", season = NULL, exogen = NULL)$selection	  
VARselect(data.set2, lag.max=10, type="trend", season = NULL, exogen = NULL)$selection	  
VARselect(data.set2, lag.max=10, type="both",  season = NULL, exogen = NULL)$selection	  	             
```

According to the Hannon-Queen criterium, the optimal lags is now 1.

```{r}
optimal.lags = 1
```

```{r}
#Correlation between the two variables:
cor(data.set)   
```

The correlation between the two variables in first difference is 0.36
which is still not that strong. We now calculate the reduced form of the
VAR model:

```{r fig.height=8, fig.width=8}
var.model.const  <- VAR(data.set2, p=optimal.lags, type="const", exogen=NULL)
summary(var.model.const)
plot(var.model.const)
```

Check the contemporaneus correlation matrix of VAR residuals
Contemporaneous effects across edogenous variables operate through the
residual correlations If correlations are significantly different from
zero then the forecast error of the endogenous variables are correlated

When there are several lags in a VAR, using the IRF to analyze dynamic
interactions is more informative than reporting the coefficient
estimates.

# 6.1 Granger and Instantaneous causality test for the VAR model

We start by running the models without using a robust heteroskedasticity
variance-covariance matrix for the Granger test:

## Inflation --\> TCU

```{r}
causality(var.model.const, cause="diffInflation", boot=FALSE, boot.runs=1000)
```

## TCU --\> Inflation

```{r}
causality(var.model.const, cause="diffTCU", boot=FALSE, boot.runs=1000)
```

The previous Granger causality tests do not control for
Heteroskedasticity, so we run the Granger tests using the HC covar
matrix using a robust heteroskedasticity variance-covariance matrix for
the Granger test:

## Inflation --\> TCU

```{r}
causality(var.model.const, cause="diffInflation", boot=FALSE, boot.runs=1000, vcov.=vcovHC(var.model.const))
```

The p-value is 0.10, suggesting that Inflation might Granger-cause TCU
in first differences.

## TCU --\> Inflation

```{r}
causality(var.model.const, cause="diffTCU", boot=FALSE, boot.runs=1000, vcov.=vcovHC(var.model.const))
```

The p-value here is much larger (0.30), so we do not reject the Null
Hypothesis Ho: TCU do not Granger-cause Inflation. 

## Summary 
add 
comments on the granger causality tests ran wth and without robust
standard errors.

# 6.2 Recursive VAR

Now we use a VAR model that supposes that every variable is endogenous
to one another by using the Cholesky decompositions for orthogonal
errors. Because we previously stated that the causality runs from
Inflation to TCU, we order the data set accordingly and estimate the
ordered VAR with a constant.

```{r}
#Ordering: Inflation --> TCU
ordered.data.set2 = data.set2[, c("diffInflation","diffTCU")]
```

```{r}
#Estimate the ordered VARs:
var.ordered.const  <- VAR(ordered.data.set2,  p=optimal.lags,   type="const",   exogen=NULL)	
```

# 6.3 Forecast error variance decomposition (in percentage terms)

## Short run (non-cumulative):

```{r}
plot(irf(var.ordered.const, n.ahead=20, ortho=TRUE, cumulative=FALSE, boot=TRUE, ci=0.90, runs=100))
		
plot(irf(var.ordered.const,  impulse="diffInflation", response="diffTCU", n.ahead=20, ortho=TRUE, cumulative=FALSE, boot=TRUE, ci=0.90, runs=100, seed=NULL),
main="TCU to Inflation", xlab="Lag", ylab="", sub="", oma=c(3,0,3,0))	

plot(irf(var.ordered.const,  impulse="diffTCU", response="diffInflation", n.ahead=20, ortho=TRUE, cumulative=FALSE, boot=TRUE, ci=0.90, runs=100, seed=NULL),
main="TCU to Inflation", xlab="Lag", ylab="", sub="", oma=c(3,0,3,0))
```

In the short run, the impulse response functions are returning back to
zero because the variables in fist differences are now stationary.
Because the variables are stationary, also the Granger-causality tests
are meaningful.

## Long run (cumulative):

```{r}
plot(irf(var.ordered.const, n.ahead=20, ortho=TRUE, cumulative=TRUE, boot=TRUE, ci=0.90, runs=100))

plot(irf(var.ordered.const,  impulse="diffInflation", response="diffTCU", n.ahead=20, ortho=TRUE, cumulative=TRUE, boot=TRUE, ci=0.90, runs=100, seed=NULL),
main="Inflation to TCU", xlab="Lag", ylab="", sub="", oma=c(3,0,3,0))

plot(irf(var.ordered.const,  impulse="diffTCU", response="diffInflation", n.ahead=20, ortho=TRUE, cumulative=TRUE, boot=TRUE, ci=0.90, runs=100, seed=NULL),
main="TCU to Inflation", xlab="Lag", ylab="", sub="", oma=c(3,0,3,0))
```

The long-run impulse functions, being cumulative, do not need to revert
back to zero. These functions show that .......

# 6.4 Diagnostic Testing for the estimated VAR models:

We now need to check for the residuals of the model, and we do so by
computing all the required tests.

## Serial correlation test

First we start from testing for serial correlation using a constant. The
null hypothesis here is Ho = residuals do not have serial correlation

```{r}
serialtest <- serial.test(var.model.const, type = "PT.asymptotic")
serialtest
	
serialtest <- serial.test(var.model.const, type = "PT.adjusted")
serialtest
	
serialtest <- serial.test(var.model.const, type = "BG")
serialtest
	
serialtest <- serial.test(var.model.const, type = "ES")
serialtest
		
plot(serialtest)
```

The p-values in the Portmanteau tests are both zero, so we reject the
null hypothesis of no serial correlation, while in the Breusch-Godfrey
and Edgerton-Shukur tests we fail to reject the null hypothesis of no
serial correlation at the 5%, but we do at the 10%. The tests show
presence of serial correlation but the results are not convincing.

## Normality test

We now check for normality in the residuals by employing the
Jarque-Baquera test.

```{r}
normalitytest <- normality.test(var.ordered.const)
normalitytest
plot(normalitytest)
```

The p-value is zero so the residuals are not normally distributed.

## Stability

We can quickly check for the stability of the coefficients:

```{r}


var.stabil <- stability(var.model.const, type = "OLS-CUSUM", dynamic = TRUE)
plot(var.stabil)


var.stabil <- stability(var.model.const, type = "Rec-CUSUM", dynamic = TRUE)
plot(var.stabil)


var.stabil <- stability(var.model.const, type = "Score-CUSUM")
plot(var.stabil)
```

Not issues in the stability of the coefficients, so we can proceed now
with the Out-of-sample prediciton.

## Out-of-sample Prediction

Number of period ahead = 10

```{r}
var.prd.const <- predict(var.model.const, n.ahead = 10, ci = 0.95)
plot(var.prd.const)
```

The graphs above show a prediction of the two series for the next 10
months. The blue line represents the predicted value, while the red line
the confidence interval.

# 7. ARIMA MODELS FOR FORECASTING

We now use the ARIMA model. Because the ARIMA model aims at using one
variable and its own past values to identify how the variable can
predict itself in the future, we need to choose one time series. Even if
the time series has a unit root, we will proceed using the time series
in levels, as the purpose of the model is prediction and not to
establish causality. We will use Inflation and test whether the model
can predict inflation accurately.

```{r}
#Transform the data set from first differences to levels once again
data.set = na.omit(
ts.intersect(
Inflation,
TCU, 

dframe=TRUE))

Inflation   = ts( data.set$Inflation,    start=c(1967, 1),  frequency=12)
TCU         = ts( data.set$TCU,          start=c(1967, 1),  frequency=12)

data.set=ts(data.set, start=c(1967, 1), frequency=12)	  
```

```{r include=FALSE}
#Select one of the two series
w = Inflation

par(mfrow=c(2,2))	
acf     (w,          main="ACF for z (levels)")
pacf    (w,          main="PACF for z (levels)")
```

# 7.1 Automatic ARIMA model selection

Moreover, we us the Automatic ARIMA and SARIMA model selection in order
to select the best ARIMA model according to either AIC, AICc or BIC
value. The function conducts a search over possible models within the
constraints provided.

```{r}
arima.model = forecast::auto.arima(w,
	                                   
D = 1,
stationary = FALSE,
ic = c("aicc", "aic", "bic"),
stepwise = FALSE,
approximation = FALSE,
seasonal = TRUE,
allowdrift = TRUE   ) 
		
arima.model	
plot(arima.model)
```

Auto.arima selects ARIMA(2,0,0)(2,1,0)[12] for the Inflation series,
using seasonal dummies monthly. Moreover, as can be seen from the plot
above, the unit roots are within the circle. We can now use the
estimated ARIMA model to forecast future values of the time series.

# 7.2 Tests

After having selected the ARIMA(2,0,0)(2,1,0)[12] for the Inflation
series, we conduct routine tests on the residuals.

```{r}
#Plot the histogram of the residuals, together with the ACF and PACF of
#the residuals:

residuals = resid(arima.model)
		
par(mfrow = c(2, 2))
plot( residuals )                 
abline( h=0, col="red" )
hist( residuals, col="green")    
acf( residuals )                 
pacf( residuals )                 	
```

## Ljung-Box

H0: series is independent; H1: series has dependence across lags Test
statistic is distributed as a chi-squared random variable Desired
result: p-value of the Ljung-Box test on the ARIMA residuals is high
and, hence, the ARIMA residuals are not auto-correlated over time. But
if p-value is small then reject the null and conclude that the residuals
are auto-correlated and, hence, are not white noise.

```{r}
Box.test(resid(arima.model), lag = 2, fitdf=0)
	
Box.test(resid(arima.model), lag = 1, type= "Ljung", fitdf=0)
	
Box.test(resid(arima.model), lag = 2, type= "Ljung", fitdf=0)
	
Box.test(resid(arima.model), lag = 3, type= "Ljung", fitdf=0)
	
Box.test(resid(arima.model), lag = 4, type= "Ljung", fitdf=0)
	
Box.test(resid(arima.model), lag = 5, type= "Ljung", fitdf=0)
	
Box.test(resid(arima.model), lag = 6, type= "Ljung", fitdf=0)
```

The ARIMA model has selected the series in first differences, so we do
not expect the residuals to be serially correlated. By running the
Box-Ljung test, the p-values for the first 6 lags are greater than 0.01,
so we fail to reject the null hypothesis of autocorrelation.

## Shapiro-Wilk Normality Test

Ho = series is normaly distributed = kurtosis is zero and skewness is
zero Ha = series is not normally distributed

```{r}
shapiro.test(resid(arima.model)) 
```

According to the Shapiro-Wilk test, we reject the null hypothesis of
normality, so the residuals are not normally distributed.

## Jarque-Bera Normality Test

Ho = series is normally distributed = kurtosis is zero and skewness is
zero Ha = series is not normally distributed Using package "tseries"

```{r}
jarque.bera.test(residuals)             
jarque.bera.test(resid(arima.model))
```

According to the Jarque-Bera test, we reject the null hypothesis of
normality in the residuals, which confirm that the residuals are not
normally distributed.

## Out-of-sample forecasting

```{r}
ARIMA.forecast = forecast::forecast(arima.model, h = 10)
	
plot(forecast::forecast(arima.model))
```

The plot above shows the prediction that we get with the model computed
above. It predicts that inflation rate might decline in the next 4
months of roughly 2%, for then rise back up close to 9%.

## In-sample forecasting

```{r}
library(smooth)
```

We use the seasonal ARIMA model in order to test whether the model is
able to predict itself. We exclude the last 10 months of the series, and
check whether the model is able to predict correctly the last 10 months
of the series based on its own past values.

```{r}
sarima.model = smooth::msarima(wl, orders=list(
	  
ar = c(2,2),
i = c(0,1),
ma = c(1,0)),
lags = c(1,12),
h = 10,         
holdout = TRUE)
	
	
summary(sarima.model)
	
values = sarima.model
	
greybox::graphmaker(wl,values$forecast,values$fitted,values$lower,values$upper,level=0.95,legend=TRUE)
```

The model is not able to predict the last 10 months that we discarded so
can not be considered as a valid predictive model.

# 9. CONCLUSIONS

# **PANEL DATA**

# 1. INTRODUCTION

This section of the project aims at analysing the impact that Income Per Capita jointly with Safe Water Access, Health Expenditure, and Pregnant Women with anemia has on Maternal Mortality Rate across the given countries over the years. There are over 1,300 indicators available from WDI but for the purpose of this project we arere only interested in the following indicators:

NY.GDP.MKTP.KD.ZG GDP growth (annual %) 
SH.H2O.SAFE.ZS Improved watersource (% of population with access) 
SH.XPD.TOTL.ZS Health expenditure,total (% of GDP) 
SH.STA.MMRT Maternal mortality ratio (modeled estimate,per 100,000 live births) S
H.PRG.ANEM Prevalence of anemia among pregnant women (%)
SH.MED.NUMW.P3 Nurses and midwives (per 1,000 people)

We will use this dataset to understand the causes of maternal mortality
rates around the world. We will calculate how maternal mortality rates
change over time and across countries and what factors are related to
this change.

```{r, warning=FALSE, message=FALSE}
#Load the necessary packages:
library(lmtest)             
library(texreg)              
library(tidyr)              
library(dplyr)              
library(pdfetch)            
library(foreign)            
library(car)                
library(gplots)             
library(tseries)            
library(sjPlot)             
library(huxtable)           
library(ivreg)              
library(plm)                
```

# 2. LOADING THE DATA


```{r include=FALSE}
#Load the dataset
wdi = read.csv("wdi.csv", na.strings = "NA")
```

The variables available in the WDI file are the following:

Country (total of 192 countries) 
Year (from 1960 to 2015)
IncomePerCapita (constant 2010 US) 
GDPGrowth (annual %) 
GDPPerCapita (constant 2010 US\$) 
OilRents (oil rents as % of GDP) 
SafeWaterAccess (% of population with access to safe water) 
NursesMidwives (nurses and midwives per 1,000 people) 
PregnantWomenWithAnemia (pregnant women with anemia in %) 
MaternalMortality (maternal mortality ratio, modeled estimate, per 100,000 live births) 
HealthExpenditure (total health expenditure as % of GDP) 
PovertyGap (poverty gap at \$1.90 a day, 2011 PPP, in %) 
InfantMortality (mortality rate, infant, per 1,000 live births)

Note that this is an unbalanced panel, as the dataset in use is missing a
high number of data.

```{r eval=FALSE, include=FALSE}
#Show the first rows only
head(wdi)    
#Show the first 50 rows
wdi[1:50,]   
#Show the entire data set
View(wdi)    
```
Please find below a list of the countries covered by the WDI dataset:
```{r}
#View the countries covered by the wdi dataset
unique(wdi$Country)           
```

```{r}
#Count the number of countries covered by the wdi dataset
length(unique(wdi$Country))    
```
The dataset countains data for 192 countries around the world.
It follows below a table with all the key statistics of each variable in the dataset such as max, min, average, median etc.
```{r}
#Summary of the key statistics of the dataset (max, min, average, median etc.)
summary(wdi)                  
```

```{r eval=FALSE, include=FALSE}
##Transform the dataset into a panel dataset 
wdi2 = pdata.frame(wdi, index = c("Country", "Year"))
head(wdi2)
wdi2[1:50,]
View(wdi2)
```

# 3. POOLED OLS, FE, AND RE MODELS

Now we can estimate each model one at a time. For our analysis, we will include in the regression IncomePerCapita. We expect that increasing income per capita will reduce maternal mortality rate.

# 3.1 Pooled OLS Model

The first step is to run a Pooled OLS model that does not take into
account the panel heterogeneity. This ignore both the panel and fixed effects, and the table that will be generated will simply stack the data and run OLS. We will then compare this simple OLS model with the Fixed Effects (FE) and Random Effects (RE) models thaat will be introduced later. 

```{r}
pooled_OLS = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita, 
data   = wdi, 
index  = c("Country", "Year"), 
model  = "pooling")


summary(pooled_OLS)
```
After running the pooled OLS model, the first thing that comes at our attention is the sign of the variable health expenditure: we expect a negative sign for this coefficient, as intuitively increasing health expenditure should decrease maternal mortality rate. The coefficient shows a positive sign which is arguably counterintuititve. Moreover, all variables except Income Per Capita are significant at the 0.1% level. This could be because an endogeneity problem might persist, or the pooled OLS model could not be specified correctly. Further test are required.

# 3.2 Fixed Effects Model

Our dataset includes a number of indicators that we believe could be related to maternal mortality rates such as healthcare expenditure, proportion of population with access to safe water. But we also know that a number of other factors could have a profound effect on maternal mortality rates but they are either not easily available or not
observable at all. Some of these factors could include a lack of
awareness of preventable diseases and attitudes towards prevention or
prenatal care that vary greatly from country to country but don't change
over time within the same country. Without access to datasets
controlling for these factors, our model would suffer from omitted
variable bias. But a fixed effect (FE) model allows us to control for
these variables that are constant over time. Firstly, we run the within model, and the effect is only controlling for the individual effects and ignoring the time effects. Nevertheless, it will partially controlling for endogeneity problem by taking into account the unobserved effect.

```{r}
fixed_effects = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita, 
                    data   = wdi, 
                    index  = c("Country", "Year"), 
                    model  = "within", 
                    effect = "individual")

summary(fixed_effects)
```

By computing the country FE model we get the expected signs for the betas, and Income Per Capita coefficient is now significant at the 0.1% significance level too although still shows a positive sign. The adjusted R squared results quite low.


In addition to country FE, a number of factors could affect maternal mortality rates that are not specific to each country. For example, development of new vaccines or medicines to fight infections, or investments in awareness campaigns by aid organizations such as WHO about preventable diseases likely vary over time but have similar effects around the world. We now run a within Fixed Effect model that controls for the time effect.

```{r}
time_effects = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita, 
                   data = wdi, 
                   index = c("Country", "Year"), 
                   model = "within", 
                   effect = "time")

summary(time_effects)
```

But with time fixed effects, the model does not distinguish between
observations across different countries. And we get unexpected signs for
the betas. When we are not using country fixed effects, the fluctuations
in the series over time make it seem as if an increase in
HealthExpenditure is correlated with increase in MaternalMortality, but
in fact it's just an artifact of time fixed effects. On the other side, it seems that an increase in income per capita over time contributes to lower maternal mortality, which is what we expect, but the coefficient is not statistically significant.

In order to control for both country and time fixed effects at the same time, we need to
estimate a model using the the two ways effect:

```{r}
twoway_effects = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita, 
data = wdi, 
index = c("Country", "Year"), 
model = "within", 
effect = "twoways")

summary(twoway_effects)
```

Once we include both individual (country) and time FE, we get the
expected signs for the betas. Now we can create a consolidated table
comparing the 3 FE models side-by-side and include the Pooled OLS model for comparison. The coefficient of all variables are statistically significant at the 0.1% significance level, but Income Per Capita still presents a positive sign. Nevertheless, the coefficient is close to zero, which might suggest thaat increasing income per capita might not have a direct effect on Maternal Mortality Rate.

```{r}
screenreg(list(pooled_OLS,
fixed_effects, 
time_effects, 
twoway_effects), 
          
custom.model.names = c("Pooled OLS", 
"Country FE", 
"Time FE", 
"Two-way FE"))
```

All four explanatory variables are statistically significant when controlling for individual fixed effects and when controlling for both time and fixed effect, but income per capita dis not significant when controlling for time effect and in the pooled OLS. Out of the four models, the Two way FE model seems to be the most meaningful. The coefficients for our explanatory variable in the two-way fixed effect model are close to the country fixed effects indicating that these factors vary greatly across countries than they do across
time.

# 3.3 Random Effects Model

In the RE model, the country-specific components are random and uncorrelated with the regressors. In this case we will compute the "Two-way RE" model, with both individual and time effects. The Error components will be Idiosyncratic, Individual, and Time.

```{r}
RE_twoway = plm( MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita,
 
data   = wdi, 
index  = c("Country", "Year"), 
model  = "random", 
effect = "twoways",
random.method = "swar" )

summary(RE_twoway)
```
The coefficients seems to have the expected signs except for IncomePerCapita, and are all statistically significant at the 0.1%.

We create a table to compare the "FE two-way" and the "RE two-way" models, including also the Pooled OLS model:

```{r}
screenreg(list(   
pooled_OLS,
twoway_effects,
RE_twoway), 
          
custom.model.names = c("Pooled OLS", 
"Two-way FE",
"Two-way RE" ))
```

# 3.4 Lagged Dependent Variables (LDV) and Dynamic Models

Another way to address auto-correlation (AR) is by modelling the time dependence directly. We can think of a dynamic model as one that takes into account whether changes in the predictor variables have an immediate effect on our dependent variable or whether the effects are
distributed over time. We do so by introducing a lagged value of the dependent variable in the right hand side of the equation hence including a AR(1) process among the explanatory variables.

```{r}
ldv_model = plm(MaternalMortality ~ lag(MaternalMortality) + SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita, 

data = wdi, 
index = c("Country", "Year"), 
model = "within", 
effect = "twoways" )

summary(ldv_model)
```
By includng the lagged value of the dependent variable, the significance of the variables drops, and the sign of some variables come out counterintuitive: Pregnant Women With Anemia has now negative signs. Moreover, Health Expenditure is statistically insignificant, and Safe Access To Water is statistically significant only at a 10% level. 

We now create a table to compare all models constructed so far:
```{r}
screenreg(list(   
pooled_OLS,
twoway_effects,
RE_twoway,
ldv_model), 
          
custom.model.names = c("Pooled OLS", 
"Twoway FE",
"Twoway RE",
"LDV Model"))
```
By comparing all the models run so far, we can conclude that more work needs to be done on the regression models as the results are not convincing and ambiguous: probably more variables need to be included, we need to control for endogeneity problems, and also introduce Instrumental Variables. We will now conduct some tests before explore the data further.

# 4. TESTS

## Poolability Test

The poolability test is a F test for individual fixed effects.

Null hypothesis: Pooled OLS better than FE  
Alt hypothesis: Significant individual FE 

```{r}
pFtest(fixed_effects, pooled_OLS)


pFtest(twoway_effects, pooled_OLS) 
```

The p-values are low so we reject the null hypothesis, and we use the FE
model instead of Pooled OLS.

We need to check whether there are indeed any country fixed effects to
begin with, so we run the plmtest() function which can test for the presence of
individual or time effects. 

Ho = no significant fixed effects 

```{r}
plmtest(fixed_effects, effect="individual")
```

The p-value suggests that we can reject the null hypothesis and that
there are indeed country fixed effects present in our model.

Let's run the same test on the time_effects model to see if there are
time FE in our model. 
Ho = no time fixed effects
```{r}
plmtest(time_effects, effect="time")
```

The p-value is very high and thus we cannot reject the null hypothesis of no time fixed effects. 

We also test for FE in the twoway model: 
```{r}
plmtest(twoway_effects, effect="twoways")
```
The low p-value suggests that we need to control for both individual and time effects in the
"within" FE model.

The conclusions of the second round of tests is that firstly, we should control for individual effects; secondly, time effects by themselves are not significant; lastly, the last test suggests that we should control for both time and individual effects at the same time.

## Hausman Test

Let us compare the FE and RE models. The Hausman test checks for the
correlation between the regressors and the (individual or time) unobserved effects. Note that the Hausman test by itself does not provide a definitive answer to whether we should run the FE or the RE models.

Ho: Errors are not correlated with the regressors 
No significant difference between FE and RE 
RE is more efficient and is preferred

Ha: Errors are correlated with the regressors FE is preferred

```{r}
phtest(twoway_effects, RE_twoway)
```

The p-value is not low so we can't reject Ho, and conclude that RE is
better than FE. But if we include the lagged dependent variable (LDV)
then the FE model is better than the RE model:

```{r}
phtest(ldv_model, RE_twoway)
```
This could be related to the fact that by including the lagged dependent variable, a model with higher R squared is computed.

## Serial Correlation Test

For time series data we need to address the potential for serial
correlation (AR) in the error term. We will test for serial correlation
with the Breusch-Godfrey test using the function "pbgtest()".
Ho = no AR in the (idisyncratic) error terms 
```{r}
pbgtest(twoway_effects)
pbgtest(RE_twoway)
pbgtest(ldv_model)
```
The p-values are low, so we rehect the null hypothesis of no serial correlation and conclude thwat the idyosincratic errors include an AutoRegressive component.

We can correct for serial correlation using "coeftest()" similarly to how we correct for heteroskedastic errors. We'll use the "vcovHC()" function for obtaining a heteroskedasticity-consistent covariance matrix, but since we're interested in correcting for autocorrelation (AR) as well, we will specify method = "arellano" which corrects for
both heteroskedasticity and autocorrelation (HAC).

```{r}
#HAC = heteroskedasticity and autocorrelation consistent estimators and standard errors:
twoway_effects_hac = coeftest(twoway_effects, 
                              vcov = vcovHC(twoway_effects, 
                                            method = "arellano", 
                                            type   = "HC3"))
```

Now create a consolidated table to compare the estimates:

```{r}
screenreg(list(
  twoway_effects, 
  twoway_effects_hac),
  
  custom.model.names = c("Twoway Fixed Effects", 
                         "Twoway Fixed Effects (HAC)"))
```

Variables that are initially significant might turn out to be
insignificant once we correct for AR and HC using the HAC estimator.
Notice that the coefficient estimates do not change but the standard
deviations and p-values do change once we introduce the HAC covariance
matrix to get the robust standard deviations. In this case HealthExpenditure and PregnantWomenWithAnemia are not statistically significant anymore, and IncomePerCapita is statistically significant only at the 5% significance level. Adjusted R squared are extremely low too, while the standard deviations have increased significantly.

## Cross Sectional Dependence (XSD) Test

When we have panel data with many countries, there might be a factor that is affecting all the countries at the same time, but we might not have a variable to control for such factor. To test for cross sectional dependence and capture the effect of such missing variable, we use the Pesaran cross sectional dependence test: pcdtest().
Ho = no XSD

```{r warning=FALSE}
pcdtest(twoway_effects)

pcdtest(RE_twoway)

pcdtest(ldv_model)
```
The p-values are low, so we do reject the null hypothesis of no XSD and conclude that there is a factor affecting all countries at the same time, but we do not have a specific variable that control for such effects.

If there is XSD the we need to correct for it. We can use the Beck and Katz (1995) method: "Panel Corrected Standard Errors (PCSE)". We can obtain Panel Corrected Standard Errors (PCSE) by first obtaining a robust variance-covariance matrix for panel models with the Beck and
Katz (1995) method using the vcovBK() and passing it to the familiar coeftest() function:

```{r}
twoway_effects_pcse = coeftest(twoway_effects, 
                               vcov = vcovBK(twoway_effects, 
                                             type = "HC3", 
                                             cluster = "group")) 

twoway_effects_pcse
```

The results from PCSE are sensitive to the ratio between the number of
time periods in the dataset (T) and the total number of observations
(N). The coefficients have now lost significance except for SafeWaterAccess. In large datasets, the T/N ratio is small. To deal with the limitations of the Beck and Katz's PCSE method, we now implement the Discroll and Kraay (1998) SCC method. This allows us to to obtain heteroskedasticity and autocorrelation consistent (HAC) errors that are also robust to
cross-sectional dependence. We can get SCC corrected covariance matrix
using the vcovSCC() function:

```{r}
twoway_effects_scc = coeftest(twoway_effects,
                              vcov = vcovSCC(twoway_effects, 
                                             type = "HC3", 
                                             cluster = "group"))

twoway_effects_scc
```
Even by using the Discroll and Kraay method, the coefficients have still low significance. IncomePerCapita has gained significance at the 5%.

We can create a consolidated table with a comparison of all the
different models computed so far.

```{r}
huxreg.matrix  =  huxreg(  
  
  "Pooled OLS" = pooled_OLS,
  "RE two-way effects" = RE_twoway, 
  "Lagged dep. variable" = ldv_model,
  "FE two-way effects" = twoway_effects,
  "Arellano HAC FE" = twoway_effects_hac,
  "Beck-Katz two-way FE" = twoway_effects_pcse, 
  "Driscoll-Kraay two-way FE" = twoway_effects_scc,
  "FE indiv. effects" = fixed_effects, 
  "FE time effects" = time_effects, 
  stars = c(`*` = 0.1, `**` = 0.05, `***` = 0.01))

huxreg.matrix
```

# 5. INSTRUMENTAL VARIABLES, ENDOGENEITY, AND GMM ESTIMATION


# 5.1 Baseline non-IV model

The baseline model chosen for this project will be the Fixed effects (FE) model with two-way effects (individual and time effects) and with no instruments previously computed.

```{r}
twoway_effects = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita, 
                     
                     data = wdi, 
                     index = c("Country", "Year"), 
                     model = "within", 
                     effect = "twoways")

summary(twoway_effects)
```

We also compute the heteroskedasticity-robust standard errors.

```{r}
coeftest(twoway_effects, vcov = vcovHC, type = "HC1")
coeftest(twoway_effects, vcov = vcovHC, type = "HC2")
coeftest(twoway_effects, vcov = vcovHC, type = "HC3")
coeftest(twoway_effects, vcov = vcovHC, type = "HC4")
```

# 5.2 Two-stage least squares (2SLS) with external IV

Suppose we have reasons to believe that one of the regressors is not strictly exogenous. In this case we would need to find an instrumental variable (IV) for it. Endogeneity makes OLS estimates biased and inconsistent, and therefore must be remedied. The instrument must be
correlated with the regressor but not with the error (it must be exogenous). Also, the instrument must affect the dependent variable only indirectly via the regressor, and not directly. We must have at least the same number of instruments as endogenous regressors, or maybe more IV than endogenous regressors, but not less instruments than endogenous regressors. The 2SLS estimator is consistent and normally distributed when the sample size is large.

We expect correlation between the regressor and the instrument to be high, 
the correlation between the dependent variable and the instrument should be
low, and the correlation between the instrument and the residuals of the non-IV
baseline model should be close to zero

Suppose "HealthExpenditure" is endogenous to "MaternalMortality". We now
need to find an IV for "HealthExpenditure". We will choose an external IV that is in the dataset but not already in the model. The instrument must be correlated with
"HealthExpenditure" but uncorrelated with the error term, hence is must
be an exogenous variable. As an IV for "HealthExpenditure" we use "".

Run the panel IV regression:

```{r}
Two_Stage_IV = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita
                   | . - HealthExpenditure + NursesMidwives + GDPPerCapita,                
                   
                   data = wdi, 
                   index = c("Country", "Year"), 
                   model = "within", 
                   effect = "twoways",
                   inst.method = "bvk" )                                                                     

summary(Two_Stage_IV)
```
```{r}
Two_Stage_IV_1 = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita
                   | . - HealthExpenditure + OilRents + GDPPerCapita,                
                   
                   data = wdi, 
                   index = c("Country", "Year"), 
                   model = "within", 
                   effect = "twoways",
                   inst.method = "bvk" )                                                                     

summary(Two_Stage_IV_1)
```
```{r}
Two_Stage_IV_2 = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita
                   | . - HealthExpenditure + OilRents,                
                   
                   data = wdi, 
                   index = c("Country", "Year"), 
                   model = "within", 
                   effect = "twoways",
                   inst.method = "bvk" )                                                                     

summary(Two_Stage_IV_2)
```
```{r}
Two_Stage_IV_3 = plm(MaternalMortality ~ SafeWaterAccess + HealthExpenditure + PregnantWomenWithAnemia + IncomePerCapita
                   | . - HealthExpenditure + OilRents + GDPPerCapita + NursesMidwives,                
                   
                   data = wdi, 
                   index = c("Country", "Year"), 
                   model = "within", 
                   effect = "twoways",
                   inst.method = "bvk" )                                                                     

summary(Two_Stage_IV_3)
```
```{r warning=FALSE}
huxreg.matrix  =  huxreg(  
  
  "Two Stage IV " = Two_Stage_IV,
  "Two Stage IV 1" = Two_Stage_IV_1,
  "Two Stage IV 2" = Two_Stage_IV_2,
  "Two Stage IV 3" = Two_Stage_IV_3,
  "Twoway effects" = twoway_effects,
   
  stars = c(`*` = 0.1, `**` = 0.05, `***` = 0.01))

huxreg.matrix
```

The standard errors are much larger in the IV model. By running a combination of different Instrumental variables, the most convincing one seems to be IV 1, in whch only OilRents is used. The standard errors of most of the variables is pretty close to the two way effects model with no instrumental variables, although the standard error of HealthExpenditure has increase significantly. It could be argued that none of the variables available are actually instrumental variables, so further research need to be done. It could be suggested that variables that measure the average distance from the hospital of the average household, number of hospital per number of population, could be used for improving the model.

Below we can find heteroskedasticity-robust standard errors:

```{r}
coeftest(Two_Stage_IV, vcov = vcovHC, type = "HC1")
coeftest(Two_Stage_IV, vcov = vcovHC, type = "HC2")
coeftest(Two_Stage_IV, vcov = vcovHC, type = "HC3")
coeftest(Two_Stage_IV, vcov = vcovHC, type = "HC4")
```

# 5.3 GMM Estimation with internal IVs

Now we will estimate the GMM model with lagged regressors as internal
instruments. The generalized method of moments (GMM) is mainly used in
panel data econometrics to estimate dynamic models with a lagged
endogenous variable. In a GMM estimation, there are normal instruments and GMM instruments. All the variables of the model which are not used as GMM instruments are
used as normal instruments with the same lag structure as the one
specified in the model

```{r warning=FALSE}
GMM = pgmm(         MaternalMortality ~ 
                      
                      lag(MaternalMortality,       1:1) +       
                      lag(SafeWaterAccess,         0:2) +
                      lag(HealthExpenditure,       0:2) +
                      lag(PregnantWomenWithAnemia, 0:2) +
                      lag(IncomePerCapita,         0:2)
                    | lag(MaternalMortality,       2:6),         
                    
                    data = wdi, 
                    index = c("Country", "Year"), 
                    model = "onestep", 
                    effect = "individual"   )                                                                      

summary(GMM) 
```
By looking at the table, only the first lag of MaternalMortality is sufficiently significant. Results are not convincing, and suggest that different variables should be included in the model. Moreover, the Sargan test suggests that the instruments as a group are exogeneous and valid, as the p-value is greater than 0.25 (p-value = 1).

Computing the GMM using logs and lags of logs:

```{r warning=FALSE}
GMM.logs = pgmm(         log(MaternalMortality) ~ 
                           
                           lag(log(MaternalMortality),       1:1) +      
                           lag(log(SafeWaterAccess),         0:2) +
                           lag(log(HealthExpenditure),       0:2) +
                           lag(log(PregnantWomenWithAnemia), 0:2) +
                           lag(IncomePerCapita,              0:2)
                         | lag(log(MaternalMortality),       2:6),        
                         
                         data = wdi, 
                         index = c("Country", "Year"), 
                         model = "onestep", 
                         effect = "individual"   )                                                                      

summary(GMM.logs) 
```
By computing the logs, none of the regressors are statistically significant, suggesting the the GMM model using logs is not a valid model to be used for analysis, and the elasticities are meaningless in this scenario.

# 6. COMPARISONS OF THE IV MODELS

We create a final table to compare the main models computed so far:
```{r warning=FALSE}
screenreg(list(
  twoway_effects,
  Two_Stage_IV_1,
  GMM,
  GMM.logs
),
custom.model.names = c(
  "FE two-way",
  "Two Stage IV 1",
  "GMM",
  "GMM.logs"
))
```
Among the models computed through this project, the FE Two-Way model s the one that has a higher number of significant coefficient of the independent variables. Nevertheless, none of the models is correctly specified, and we recommend to do further research and include more exhaustive variables in the model that could help to better analyse the causes of MaternalMortality across countries over time.