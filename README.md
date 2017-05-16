Now we will do it with the actual data.  First we create the phase variable.  We do this by repeating a phase for the value before the actual intervnetion start point.  See matt patch 1 for explanation.

We then add the time variable
```{r}
scdata = read.csv("SCDMark1.csv", header = TRUE)
scdata = scdata[c("person", "dep")]
dim(scdata)

phase = c(rep("A1", 5), rep("B1",15-5), rep("B1", 5), rep("A1",15-5), rep("A1", 7), rep("B1",15-7), rep("B1", 7), rep("A1",15-7), rep("A1", 8), rep("B1",15-8), rep("B1", 8), rep("A1",15-8))

scdata = cbind(scdata, phase)
scdata
time = c(rep(1:15, 6))

scdata = cbind(scdata, time)
head(scdata)
```
Now we can predict the missing values for the dep phase with the amelia package.

So one problem we run into is that we would like to model the dep variable ordinal, so we don't negative values, but it is ordinal, because there are .5 integers, so we cannot model it as ordinal in amelia.

So for values below zero we round them to zero
```{r}
m <- 5
a.out <- amelia(x = scdata, m=m, ts = "time", noms = "phase")
a.out$imputations

write.amelia(obj = a.out, file.stem = "outdata")

```
