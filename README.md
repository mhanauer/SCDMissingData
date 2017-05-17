---
title: "SCDMissingData"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Now we will do it with the actual data.  First we create the phase variable.  We do this by repeating a phase for the value before the actual intervnetion start point.  See matt patch 1 for explanation.

We then add the time variable
```{r}
setwd("~/Google Drive/MarkSCED/Data")
scdata = read.csv("SCDMark1.csv", header = TRUE)
scdata = scdata[c("person", "dep")]
dim(scdata)

phase = c(rep("A1", 5), rep("B1",15-5), rep("B1", 5), rep("A1",15-5), rep("A1", 7), rep("B1",15-7), rep("B1", 7), rep("A1",15-7), rep("A1", 8), rep("B1",15-8), rep("B1", 8), rep("A1",15-8))

# Create an age variable with the first and last doubled, because person one and two and five and six are teh same age.
age = c(rep(61, 30), rep(81, 15), rep(63, 15), rep(70, 30))
length(age)

site = c(rep(1,45), rep(2, 45))

height = c(rep(63, 15), rep(67, 15), rep(65, 15), rep(62, 15), rep(70,15), rep(64, 15))

weight = c(rep(114, 15), rep(174, 15), rep(194, 15), rep(163, 15), rep(160,15), rep(134, 15))

edu = c(rep(6, 15), rep(5, 15), rep(6, 15), rep(6, 15), rep(6,15), rep(3, 15))

timeVar = c(rep(1.5, 15), rep(23, 15), rep(7.66, 15), rep(6.5, 15), rep(5,15), rep(10, 15))
  
surgery = c(rep(1.5, 15), rep(23, 15), rep(7.66, 15), rep(6.5, 15), rep(5,15), rep(10, 15))

at = c(rep(1, 15), rep(1, 15), rep(0, 15), rep(0, 15), rep(1,15), rep(0, 15))

time = c(rep(1:15, 6))

dep = scdata$dep

scdata = cbind(scdata, phase, time, age, site, height, weight, edu, timeVar, surgery, at)
scdata = as.data.frame(scdata)
scdata = scdata[c("dep", "phase", "time", "age", "site", "height", "weight", "edu", "surgery")]
head(scdata)
```
Now we can predict the missing values for the dep phase with the amelia package.

So one problem we run into is that we would like to model the dep variable ordinal, so we don't negative values, but it is ordinal, because there are .5 integers, so we cannot model it as ordinal in amelia.

So for values below zero we round them to zero
```{r}
m <- 5
a.out <- amelia(x = scdata, m=m, ts = "time", noms = c("phase", "site"), ords = "edu")
a.out$imputations

write.amelia(obj = a.out, file.stem = "outdata")

```
