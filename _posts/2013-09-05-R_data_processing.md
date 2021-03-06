---
layout: post
title:  "Data processing with R."
date:   2013-09-05
---

**R script to grab subset of data**  
In previous work, all the variants are plotted with three variables: folding free energy at 37C (dGflex_37C), 
temperature senstivity normalized score (TS_delta_norm_socre), and GFP seq activity (delta2_reflow_normalized).  
![scatter plot][1]
In this work, all the variants that are considered as outliers will be extracted from the original data set. 
The outliers that are the upper left Quadran and lower right Quadran, where temperature senstivity is not able to be explained by folding free energy at 37C.  

_**R scirpt**_:  

```{r}
#load library
library(ggplot2)
library(pROC)

#setup working dir and input files.
workdir = "/Users/ju/Works/Sup4ochre/analysis/work19.4"
infile = "tRNA_7_31_14_Ju_processed_10-29-14b.csv" 

setwd(workdir)

#read in original csv file as raw_data
raw_data <- read.csv(infile, header = TRUE, sep = ",", quote = "\"", dec = ".", 
					fill = TRUE)

#setup filtering settings
f_reads = 100
f_cell_counts = 30
f_mutation_counts = 3
f_GFP_seq = 0.056
f_isTS_delta = 0.50
f_isTS_wt = 0.65
f_isRTD = 2.0

#subsetters
sub_mutation = 1
sub_dG_flex = -22.5 
sub_TS_norm_delta = 0.50
sub_TS_norm_wt = 0.65

#select come columns from raw_data
collist <- c("ID", "WT2.reflow.normalized.GFPseq", "delta2.reflow.normalized.GFPseq", 
			 "X37deg.2.reflow.normalized.GFPseq", "delta2.37deg.reflow.normalized.GFPseq",
			 "Mutation.Count","WT2.Total.Counts", "delta2.Total.Counts", 
			 "X37deg.2.Total.Counts","delta2.37deg.Total.Counts", "WT2.Total.Cells",  
			 "delta2.Total.Cells",  "X37deg.2.Total.Cells", "delta2.37deg.Total.Cells",
			 "ID2", "pos", "mut", "loc", "mature_ddG_rigid_22C", "mature_ddG_flex_22C", 
			 "mature_ED_22C", "mature_ddG_rigid_28C", "mature_ddG_flex_28C", "mature_ED_28C", 
			 "mature_ddG_rigid_37C", "mature_ddG_flex_37C", "mature_ED_37C", 
			 "mature_ddG_rigid_45C", "mature_ddG_flex_45C", "mature_ED_45C", "dGflex_28C", 
			 "dGflex_37C", "ddG_flex37.28", "Sequence")
 
data <- raw_data[collist]

#filter the data: 
filtered_data <- subset(data, Mutation.Count <= f_mutation_counts & 
						WT2.Total.Counts >= f_reads & delta2.Total.Counts >= f_reads & 
						X37deg.2.Total.Counts >= f_reads & delta2.37deg.Total.Counts >= f_reads &  
						WT2.Total.Cells >= f_cell_counts & delta2.Total.Cells >= f_cell_counts & 
						X37deg.2.Total.Cells >= f_cell_counts & delta2.37deg.Total.Cells >= f_cell_counts &
						!grepl("loop", loc))

write.csv(filtered_data, "tRNA_7_31_14_Ju_processed_10-29-14b_filtered_data.csv", row.names=FALSE)

#calculate TS normalized scores for delta and wt, and create a binary variable isTS
filtered_data["TS_delta_norm_score"] <- (filtered_data$delta2.reflow.normalized.GFPseq - 
										 filtered_data$delta2.37deg.reflow.normalized.GFPseq)/
										 filtered_data$delta2.reflow.normalized.GFPseq

filtered_data$isTS_delta <- ifelse(filtered_data$TS_delta_norm_score < f_isTS_delta, 0, 1)

filtered_data["TS_wt_norm_score"] <- (filtered_data$WT2.reflow.normalized.GFPseq - 
									  filtered_data$X37deg.2.reflow.normalized.GFPseq)/
									  filtered_data$WT2.reflow.normalized.GFPseq

filtered_data$isTS_wt <- ifelse(filtered_data$TS_wt_norm_score < f_isTS_wt, 0, 1)

#calculate RTD normalized scores for delta and wt, and create a binary variable isTS
filtered_data["RTD_28C_score"] <- filtered_data$delta2.reflow.normalized.GFPseq / 
   								  filtered_data$WT2.reflow.normalized.GFPseq
filtered_data$isRTD_28C <- ifelse(filtered_data$RTD_28C_score < f_isRTD, "notRTD", "RTD")

filtered_data["RTD_37C_score"] <- filtered_data$delta2.37deg.reflow.normalized.GFPseq / 
 								  filtered_data$X37deg.2.reflow.normalized.GFPseq
filtered_data$isRTD_37C <- ifelse(filtered_data$RTD_37C_score < f_isRTD, "notRTD", "RTD")

#filter for delta batch, using cutoff of GPF_seq
filtered_data_delta <- subset(filtered_data, delta2.reflow.normalized.GFPseq >= f_GFP_seq)
filtered_data_wt <- subset(filtered_data, WT2.reflow.normalized.GFPseq >= f_GFP_seq)

#get subset of data
#Get subset of variants:filtered_data_delta (single, dGlfex_37C <=-22.5kcal/mol, 
#TS_norm_score_delta >= 0.5) OR (dGlfex_37C > -22.5kcal/mol, TS_norm_score_delta < 0.5)
outlier_data_delta <- subset(filtered_data_delta, (Mutation.Count <= sub_mutation & 
					  dGflex_37C <= -22.5 & TS_delta_norm_score >= 0.5) |
  					  (dGflex_37C > -22.5 & TS_delta_norm_score < 0.5))
write.csv(outlier_data_delta, "tRNA_7_31_14_Ju_processed_10-29-14b_outlier_data_delta.csv", 
		  row.names=FALSE)
write.table(outlier_data_delta[,c("ID", "Sequence")], "tRNA_7_31_14_Ju_processed
			_10-29-14b_outlier_data_delta_seq.in", row.names=F, sep="\t", quote=F)
		  
#Get subset of variants:filtered_data_wt (single, dGlfex_37C <=-22.5kcal/mol, TS_norm_score_delta >= 0.65) 
#OR (dGlfex_37C > -22.5kcal/mol, TS_norm_score_delta < 0.65)

outlier_data_wt <- subset(filtered_data_wt, (Mutation.Count <= sub_mutation & 
						  dGflex_37C <= -22.5 & TS_delta_norm_score >= 0.65) |
						  (dGflex_37C > -22.5 & TS_delta_norm_score < 0.65))
write.csv(outlier_data_wt, "tRNA_7_31_14_Ju_processed_10-29-14b_outlier_data_wt.csv", 
		  row.names=FALSE)
write.table(outlier_data_wt[,c("ID", "Sequence")],"tRNA_7_31_14_Ju_processed
			_10-29-14b_outlier_data_wt_seq.in", row.names=F, sep="\t", quote=F)

```

[1]: https://dl.dropboxusercontent.com/u/3637996/github_pages/post_2013-09-05-R_data_processing/met22delta_TS.png