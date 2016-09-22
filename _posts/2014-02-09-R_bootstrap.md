---
layout: post
title:  "Bootstrapping with R."
date:   2014-02-09
---

 In Statistics, [bootstrapping][1] can refer to any test or metric that relies on random sampling with replacement. 
 Bootstrapping allows assigning measures of accuracy (defined in terms of bias, variance, 
 confidence intervals, prediction error or some other such measure) to sample estimates.
 This technique allows estimation of the sampling distribution of almost any statistic using very simple methods. 
 Generally, it falls in the broader class of resampling methods.

#### Example  
In this study, z-score for all the inhibitory codon pairs (ICP) family codon pairs (coding same dipeptide) are computed. 
The z-score is computed from each dipeptide family group. Then, z-scores distribution of all 14 ICPs and 
z-score distribution of all nonICPs from the same ICP dipeptide family were plotted and compared.
![distribution_icp_vs_nonicp_shift0][2]
![distribution_icp_vs_nonicp_shift1][3]
![distribution_icp_vs_nonicp_shift2][4]


To bootstrap over a data table, use the library `dplyr` function `sample_n()`; to bootstrap over a vector, use `sample()`

```r
library(dplyr)

setwd("/Users/ju/Works/Codon/analysis/work13.3")

#import the dataset
ICP_z_data <- read.csv("ICP_fam_CodonPairs_zscore.csv", head=T)

# Number of replicas
n = 1000

# Level of significance
alpha = 0.05

#Difference between t tests of bootstrapped dataset (subsets isICP and noICP)
ttest.bootstrap = NULL

bootstrap = 'pool'

if (bootstrap = 'pool') {
    a <- ICP_z_data
    for (i in 1:n) {
        #Sample with replacement
        a.bootstrap = sample_n(a, size = nrow(a), replace = TRUE)
    #     ttest.bootstrap[i] = mean(a.bootstrap$LogOdd.consv_rate._shift0)
        ttest.bootstrap[i] = t.test(a.bootstrap[a.bootstrap$isICP == 0, ]$LogOdd.consv_rate._shift0, 
                                    a.bootstrap[a.bootstrap$isICP == 1, ]$LogOdd.consv_rate._shift0)$statistic
    }
}

if(bootstrap != 'pool' ) {
    a <- ICP_z_data[ICP_z_data$isICP == 0,]
    b <- ICP_z_data[ICP_z_data$isICP == 1,]
    # Number of replicas
    n = 1000
    
    # Level of significance
    alpha = 0.05
    
    #Difference between t tests of bootstrapped dataset (subsets isICP and noICP)
    ttest.bootstrap = NULL
    
    for (i in 1:n) {
        #Sample with replacement
        a.bootstrap = sample_n(a, size = nrow(a), replace = TRUE)
        b.bootstrap = sample_n(b, size = nrow(a), replace = TRUE)
        #     ttest.bootstrap[i] = mean(a.bootstrap$LogOdd.consv_rate._shift0)
        ttest.bootstrap[i] = t.test(a.bootstrap$LogOdd.consv_rate._shift0, 
                                    b.bootstrap$LogOdd.consv_rate._shift0)$statistic
    }
}

# Confidence interval
quantile(ttest.bootstrap, c(alpha/2, 1 - alpha/2))
mean(ttest.bootstrap)

```




#### Other examples  
Bootstrap from two groups of dataset to get t.test quantile.  

```r
t.test.btsp <- function(data1,data2,n = 1000,n1,n2)
{
  t.values = numeric(n)
  for(i in 1:n)
  {
    group1 <- sample(data1$v1,size = n1,replace = T)
    group2 <- sample(data2$v1,size = n2,replace = T)
    t.values[i] <- t.test(group1,group2)$statistic
  }
  return(t.values)
}
a <- t.test.btsp(data1,data2,n=1000,n1 = 700, n2= 700)
quantile(a,c(0.05,0.95))
```

Bootstrap mean.

```r
# Heterozygous (BA)
a = c(86, 88, 89, 89, 92, 93, 94, 94, 94, 95, 95, 96, 96, 97, 97, 98, 98, 99, 99, 101, 106, 107, 110, 113, 116, 118)
 
# Homozygous (BB)
b = c(89, 90, 92, 93, 93, 96, 99, 99, 99, 102, 103, 104, 105, 106, 106, 107, 108, 108, 110, 110, 112, 114, 116, 116)
 
# Difference between means of observed datasets
diff.observed = mean(b) - mean(a)
 
# Level of significance
alpha = 0.05
 
# Number of replicates
n = 1000
 
# Difference between means of bootstrapped datasets (n replicates)
diff.bootstrap = NULL
 
for (i in 1 : n) {
    # Sample with replacement
    a.bootstrap = sample  (a, length(a), TRUE)
    b.bootstrap = sample  (b, length(b), TRUE)
 
    diff.bootstrap[i] = mean(b.bootstrap) - mean(a.bootstrap)
}
 
# Confidence interval
quantile(diff.bootstrap, c(alpha/2, 1 - alpha/2))
```

[1]:http://en.wikipedia.org/wiki/Bootstrapping_(statistics)
[2]:https://dl.dropboxusercontent.com/u/3637996/github_pages/post_2014-02-09-R_bootstrap/ICP_z_score_hist_shift0.png
[3]:https://dl.dropboxusercontent.com/u/3637996/github_pages/post_2014-02-09-R_bootstrap/ICP_z_score_hist_shift1.png
[4]:https://dl.dropboxusercontent.com/u/3637996/github_pages/post_2014-02-09-R_bootstrap/ICP_z_score_hist_shift2.png
