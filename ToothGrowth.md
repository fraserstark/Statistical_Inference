---
title: "Effect of Vitamin C on Tooth Growth in Guinea Pigs"
author: "Fraser Stark"
date: "09/04/2020"
output: 
  pdf_document: 
   keep_md: true
---






## Synopsis

The purpose of this document is to present an analysis on the effect of vitamin C supplements on tooth growth of guinea pigs. We will use the dataset provided from the R library `datasets` which contains the results from an experiment studying the effect of vitamin C on tooth growth in 60 Guinea pigs. Each animal received one of three dose levels of vitamin C (0.5, 1, and 2 mg/day) by one of two delivery methods, (orange juice or ascorbic acid (a form of vitamin C and coded here VC)).

  

## Overview of Dataset

The Tooth Growth dataset is provided by the library in a `data.frame` as below.  


```
## 'data.frame':	60 obs. of  3 variables:
##  $ len : num  4.2 11.5 7.3 5.8 6.4 10 11.2 11.2 5.2 7 ...
##  $ supp: Factor w/ 2 levels "OJ","VC": 2 2 2 2 2 2 2 2 2 2 ...
##  $ dose: num  0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 ...
```

As we can see there are 60 observations of 3 variables. Before we perform further analysis we can process & tidy the dataset by renaming column headers and factor variables.


```r
tg <- ToothGrowth
names(tg) <- c("length", "supplement", "dose")                # Rename columns
levels(tg$supplement) <- c("Orange Juice", "Ascorbic Acid")   # Rename factors
```

Here are the first few observations from the data:
  



\begin{tabular}{rlr}
\toprule
length & supplement & dose\\
\midrule
4.2 & Ascorbic Acid & 0.5\\
11.5 & Ascorbic Acid & 0.5\\
7.3 & Ascorbic Acid & 0.5\\
\bottomrule
\end{tabular}


## Exploratory Data Analysis

The boxplot below highlights the effect on tooth length across the six distinct groups of supplement and dose. It suggests the delivery of Vitamin C via Orange Juice has some effect on length for doses of 0.5 and 1. For dosage of 2, it appears there is no significant gain in tooth growth between supplement delivery. 

![](ToothGrowth_files/figure-latex/boxplot-1.pdf)<!-- --> 


## Hypothesis Tests


### Assumptions
We consider the following assumption in our analysis, that these subjects are a relevant sample from a population of subjects that we're interested in - and the samples are approximately normally distributed. 

### Null Hypothesis 
Firstly we want to test our Null Hypothesis (**H~0~**), which is a tentative assumption that there is no effect of Vitamin C dosage or administration type on guinea pig tooth growth (and that any measured difference is down to chance).

Since here we are dealing with two independent samples, i.e. the guinea pigs were randomly assigned to each treatment group, we will use the unpaired form of t.test by specifying `paired = FALSE`, and the final argument specifies that we want to conduct a two-tailed t.test. 



```r
t.test(length ~ supplement, data=tg, paired = FALSE, var.equal = FALSE, alternative = 'two.sided')
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  length by supplement
## t = 1.9153, df = 55.309, p-value = 0.06063
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.1710156  7.5710156
## sample estimates:
##  mean in group Orange Juice mean in group Ascorbic Acid 
##                    20.66333                    16.96333
```

As the p value returned (0.06) is greater than the significance level of 0.05, we should tentatively accept the null hypothesis unless we can find stronger evidence in the next test. 


### Alternative Hypothesis 
Given the data analysis above, our alternative hypothesis (**H~A~**) will be that varying doses of Vitamin C via orange juice *only* will have a significant impact on tooth growth within the sample population. We can perform a similar t.test but with a subset of the data for each dose level. 


```r
t.5 <- t.test(length ~ supplement, data=tg[tg$dose == 0.5,], 
              paired = FALSE, var.equal = FALSE)

t1 <- t.test(length ~ supplement,data=tg[tg$dose == 1,], 
             paired = FALSE, var.equal = FALSE)

t2 <-t.test(length ~ supplement,data=tg[tg$dose == 2,], 
            paired = FALSE,  var.equal = FALSE)
```

We can store the results of these tests in a dataframe as below.  
\  
\  


\begin{tabular}{lrrrr}
\toprule
dose & t.statistic & p.value & conf\_lower & conf\_upper\\
\midrule
0.5 & 3.1697328 & 0.0063586 & 1.719057 & 8.780943\\
1 & 4.0327696 & 0.0010384 & 2.802148 & 9.057852\\
2 & -0.0461361 & 0.9638516 & -3.798071 & 3.638070\\
\bottomrule
\end{tabular}
\  
\  
  
Given the p.value results for the first two tests are well below 0.05, we can conclude that dosages of 0.5 and 1 of Orange Juice have significant effects on tooth growth. This is also apparent from our initial data analysis where we can see that there is some overlap in the difference when dose is 2.


## Conclusions

* We could find no strong evidence that there is a difference in delivery methods of Vitamin C when the dose is 2 - i.e. There is no significant difference in effect between Orange Juice and Absorbic Acid.

* However we did find strong evidence that for dosages of 0.5 and 1 of Orange Juice have a greater effect on tooth growth than the same doses of Ascorbic Acid, and therefore we can reject the null hypothesis. 


\pagebreak
## Appendix: Supporting Code

Libraries used


```r
library(datasets)
library(ggplot2)
library(kableExtra)
```

Loading initial data


```r
tg <- ToothGrowth
str(tg)
```

Table showing First few observations


```r
kable(head(tg, 3), "latex", booktabs = T)
```

Plot: Tooth Length by Dosage & Supplement


```r
ggplot(tg, aes(x = factor(dose), y = length)) + 
  facet_grid(.~supplement) +
  geom_boxplot(aes(fill = supplement)) + 
  labs(title = "Tooth Length by Dosage & Supplement") + 
  labs(x = "Dose", y = "Tooth Length")
```

Storing final results 

```r
results <- data.frame(dose = c('0.5','1','2'),
                      t.statistic = c(t.5$statistic, t1$statistic, t2$statistic),
                      p.value = c(t.5$p.value, t1$p.value, t2$p.value),
                      conf_lower = c(t.5$conf.int[[1]],
                                     t1$conf.int[[1]], 
                                     t2$conf.int[[1]]),
                      conf_upper = c(t.5$conf.int[[2]], 
                                     t1$conf.int[[2]], 
                                     t2$conf.int[[2]])
                      )

kable(results, "latex", booktabs = T, position = 'center')
```



