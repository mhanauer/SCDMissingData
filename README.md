---
title: "SCDMissingData"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here is an example that we will work from.
```{r}
# SCDMissingData
library(Amelia)
data(africa)
head(africa)
m <- 5
a.out <- amelia(x = africa, m=m, cs = "country", ts = "year", logs = "gdp_pc")
b.out<-NULL
se.out<-NULL
for(i in 1:m) {
  ols.out <- lm(civlib ~ trade ,data = a.out$imputations[[i]])
  b.out <- rbind(b.out, ols.out$coef)
  se.out <- rbind(se.out, coef(summary(ols.out))[,2])
}
combined.results <- mi.meld(q = b.out, se = se.out)
```
Create a fake data with three people with three variable, dependent score, time, and phase (A1, B1, A2, B2)
```{r}
data = data.frame(dep = rnorm(40), time = c(1:40), phase = c(rep("A1", 10), rep("B1", 10), rep("A2", 10), rep("B2", 10)))
data[1:3,1] = NA
data[3:5,2] = NA
data[4:6,3] = NA
data
```
Now let us see how it imputes missing quantitative and qualitative data
Need to set qualitative variables as different structure.

We values are completely missing they remain completely missing.  

We need to change the missingness to only the dependent variable after this, because that is the only reasonable missing value.  Here we are just trying to see how nomial variables are imputted.

When you include time as a variable it takes it out and does not include it in the analysis.  So it seems like we shouldn't specifcy it.

Ok so for th actual run, we will have all of the time series information 
```{r}
m <- 5
a.out <- amelia(x = data, m=m, ts = "time", noms = "phase")
a.out$imputations

a.out <- amelia(x = data, m=m, noms = "phase")
a.out$imputations

```
Here we create a set of of possible Tau-U estimates that are taken from the five data sets that were run through the Tau-U SCED online calculator.
```{r}
set.seed(1234)
betas = rnorm(10)
mean(betas)
betas = as.matrix(betas)
se = as.matrix(rnorm(10))
mean(se)

combinded = mi.meld(q = betas, se = se)
```

Now we will do it with the actual data.  First we create the phase variable.  We do this by repeating a phase for the value before the actual intervnetion start point.  

For person one they are in A1 until point six then for the rest of their 15 data points they are in B1
```{r}
scdata = read.csv("SCDMark1.csv", header = TRUE)
scdata = scdata[c("person", "dep")]
scdata

person1 = c(rep("A1", 5), rep("B1",15-5))
length(person1)

# Person two starts at point 16 with B1 start A1 at 6 so therefore B1 is the first five data points.  So it is the same as person 1 execpt B1 and A1 are reversed
person2 = c(rep("B1", 5), rep("A1",15-5))

# Person three goes with A1 for 7 values and then B1 from 8 until 15
person3 =  c(rep("A1", 7), rep("B1",15-7))

# Same as person four with reverse starting
person4 = c(rep("B1", 7), rep("A1",15-7))

# A1 ends at 8 
person5 = c(rep("A1", 8), rep("B1",15-8))

# Same as person5 with reverse for A and B phases

person6 = c(rep("B1", 8), rep("A1",15-8))

phase = rbind(person1, person2, person3, person4, person5, person6)
phase = as.matrix(phase)

scdata = cbind(scdata, phase)

scdata

```




