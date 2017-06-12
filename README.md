---
title: "SCED Missing Data Example"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(
	echo = TRUE,
	message = FALSE,
	warning = FALSE
)
```
Here is an example of using the multiple imputation package Amelia to predict missing values in Single Case Experimental Designs (SCEDs).  In this artificial data set, there are six participants that were paired to randomly crossover to the other intervention (i.e. the person that started in intervention A would switch to intervention B and vice versa) at the same data point.  The first pair crossed over at data point 5, the second pair at 7, and the third pair at 8.  The data are placed in long format meaning that there is a participant id variable that is repeated once for each data point.  For example, the person variable has 15 ones to indicate that each data point in that row is associated with person one.  There is a total of 90 data points 15 for each person.  

Because Amelia uses covariates to predict data points, SCED researchers should include as many covariates as possible. Below we have included the following covariates: age, site, height, weight, and time, which is a variable that goes from one to fifteen for each participant.

Finally, we can insert some missing values in the dependent variable for Amelia to predict.
```{r, message=FALSE, warning=FALSE}
dep = round(rnorm(90),0)

person = c(rep(1,15), rep(2,15), rep(3,15), rep(4,15), rep(5,15), rep(6,15))

phase = c(rep("A1", 5), rep("B1",15-5), rep("B1", 5), rep("A1",15-5), rep("A1", 7), rep("B1",15-7), rep("B1", 7), rep("A1",15-7), rep("A1", 8), rep("B1",15-8), rep("B1", 8), rep("A1",15-8))
phase = as.data.frame(phase)

age = c(rep(61, 30), rep(81, 15), rep(63, 15), rep(70, 15), rep(69, 15))

site = c(rep(1,45), rep(2, 45))

height = c(rep(61, 15), rep(64, 15), rep(66, 15), rep(61, 15), rep(71,15), rep(65, 15))

weight = c(rep(110, 15), rep(145, 15), rep(200, 15), rep(156, 15), rep(123,15), rep(150, 15))


time = c(rep(1:15, 6))

scdata = cbind(dep, person, phase, time, age, site, height, weight)
scdata = as.data.frame(scdata)
scdata = scdata[c("dep", "person", "phase", "time", "age", "site", "height", "weight")]
scdata = as.data.frame(scdata)
scdata$dep[c(4,50,88)] = NA
is.na(scdata$dep)
sum(is.na(scdata$dep))
head(scdata)
```
Now we can run the Amelia package with the SCED data set to the data set we created above to conduct the multiple imputation to produce values for the missing values.  One reason Amelia makes the most sense for SCED researchers is that it can appropriately model time in the multiple imputation process. To set up our data, we set x to the full data frame, ts to the time variable to indicate that this variable represents time.  We then identify nominal variables by using the following argument: noms for nominal variables.  Although, in this case, setting the nominal variables to noms does not matter, because none of the covariates are missing, if covariates were missing for some reason, identifying the variables’ structure is important, because Amelia selects the appropriate regression model based upon the variable's structure.  In this particular case we will create five data sets which are written into five different csv files using the write.amelia function.
```{r, message=FALSE, warning=FALSE}
library(Amelia)
m <- 5
a.out <- amelia(x = scdata, m=m, ts = "time", noms = c("phase", "site"))
head(a.out$imputations$imp1$dep)
write.amelia(obj = a.out, file.stem = "outdata")
```
Let us now assume that we ran the SCED data through a statistical analysis.  For example, let us assume that we ran each of the five data sets through a Tau-U analysis and produced five Tau-U statistics along with their standard errors.  We can them use mi.meld function to appropriately average the results from the five different data sets and produce one overall result for the SCED design.  
```{r, message=FALSE, warning=FALSE}
parEst = rnorm(5)
parEst = as.data.frame(parEst)
standErr = abs(rnorm(5))
standErr = as.data.frame(standErr)

combinded.results = mi.meld(q = parEst, se = standErr)
combinded.results
```



