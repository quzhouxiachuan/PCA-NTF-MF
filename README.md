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

### use randome forest to do prediction 
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
Call:
roc.default(response = testing$label, predictor = p$case)

Data: p$case in 162 controls (testing$label case) > 1399 cases (testing$label ctrl).
Area under the curve: 0.8166

###double check random forest model using NTF to do prediction 

### PCA analysis 
#### construct input file 
```
library(dplyr)
library(tidyr)
x=read.csv("final_NTF_input.removesymbol.csv",header=F)
med= x %>% group_by(V1,V2,V3) %>% count(V3)
med$V2_V3=paste(med$V2,med$V3,sep='_')
med=subset(med, select=c(V1,V2_V3,n))
medd=spread(med, V2_V3,n )
write.csv(medd,'pca_feature_forrandomeforest.csv')

```
#### extract principal component and do prediction
```
preProc <- preProcess(mm[,-7047],method="pca",pcaComp=2)
trainPC <- predict(preProc,log10(training[,-58]+1))
modelFit <- train(training$type ~ .,method="glm",data=trainPC)
```

when use raw feature, logistic regression, have such warning message: 
```
Warning message: In predict.lm(object, newdata, se.fit, scale = 1, type = ifelse(type ==  : prediction from a rank-deficient fit may be misleading; 
```
I tried set.seed(1234), auc is 0.6266, the other time is 0.5886; the result is not very stable. 


