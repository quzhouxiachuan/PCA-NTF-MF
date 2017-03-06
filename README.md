# PCA-NTF-MF
###some important file for the analysis 
use file /home/ydw529/other/phenotype0720/data_csv_processed/LimitedHF_final_combined.csv
for linear regression prediction, you used "../pipeline_runs/phenotype_range_1-160_alpha_1_gamma_0.001_0.08_0.07/results/auc/phenotypes" files. The original file to produce this is?
```
x=read.csv("LimitedHF_final_combined.csv",header=F)
library(dplyr)
med= x %>% group_by(V1) %>% count(V3)
dia= x %>% group_by(V1) %>% count(V2)
spread(ww, V2, n)
library(tidyr)
diaa= spread(dia, V2, n)
medd= spread(med, V3, n)
dd=merge(diaa, medd, by='V1',all=T)
dd[is.na(dd)]=0
write.csv(dd,'raw_feature_prediction_merge.csv')
savehistory(file = ".Rhistory")
```
