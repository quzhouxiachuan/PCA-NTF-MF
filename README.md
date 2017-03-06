# PCA-NTF-MF
###some important file for the analysis 
use file /home/ydw529/other/phenotype0720/data_csv_processed/LimitedHF_final_combined.csv
for linear regression prediction, you used "../pipeline_runs/phenotype_range_1-160_alpha_1_gamma_0.001_0.08_0.07/results/auc/phenotypes" files. The original file to produce this is?

final_NTF_input.removesymbol.csv
```
x=read.csv("LimitedHF_final_combined.csv",header=F)
library(dplyr)
med= x %>% group_by(V1) %>% count(V3)
dia= x %>% group_by(V1) %>% count(V2)
library(tidyr)
diaa= spread(dia, V2, n)
medd= spread(med, V3, n)
dd=merge(diaa, medd, by='V1',all=T)
dd[is.na(dd)]=0
write.csv(dd,'raw_feature_prediction_merge.csv')
savehistory(file = ".Rhistory")
```

###use randome forest to do prediction 
```
library(doParallel)
#library(kernlab)
set.seed(415)
#edit the following directory to your auc folder directory
filenames=list.files("../pipeline_runs/phenotype_range_1-160_alpha_1_gamma_0.001_0.08_0.07/results/auc/phenotypes",pattern='*raw_feature_prediction.csv',full.names=TRUE)
#args= commandArgs(trailingOnly=TRUE)
print('this job is running...')
df=vector()
print(filenames)
registerDoParallel(cores=20)
df=foreach (i= 1:length(filenames),.combine=data.frame, .export=c('trainControl', 'train'), .packages='caret') %dopar%
{
        print (paste("read file",i))
        data=read.csv(filenames[i],row.names=1)
        data=subset(data, select=-c(pt_id))
        data$outcome_label=factor(data$outcome_label)
        inTrain=createDataPartition(y=data$outcome_label, p=0.75, list=FALSE)
        training=data[inTrain,]
        testing=data[-inTrain,]
        library(pROC)
        control <- trainControl(method="repeatedcv", number=6, repeats=3)
        mtry=sqrt(ncol(data))
        tunegrid=expand.grid(.mtry=mtry)
        fit=train((outcome_label) ~., training, tuneGrid=tunegrid, method="rf",trControl=control,ntree=70)
        p=predict(fit, testing, type='prob')
        auc=roc( testing$outcome_label,p$case)
        auc=as.character(auc[[9]])
        #phenotype=strsplit(strsplit(filenames[i],'/')[[1]][7],'_')[[1]][2]
        phenotype='raw_feature'
        data.frame(row=c(phenotype, auc))
        df=rbind(df,row)
}

df
write.csv(df,'randomforest.plot.csv')
```
