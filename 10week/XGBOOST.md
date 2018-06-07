# 10주차 xgboost를 이용한 regression
### Author: 김현우,박주연,이주영,이지예,주진영,홍정아

```
#private = 0.11381
library(dplyr)
library(data.table)
library(readr)
library(xgboost)
```

```
cat("reading the train and test data\n")
train <- fread("real_train.csv")
test  <- fread("real_test.csv")
store <- fread("store.csv")
```

```
# removing the date column (since elements are extracted) and also StateHoliday which has a lot of NAs (may add it back in later)
train <- merge(train,store)
test <- merge(test,store)
```

```
train <- train[ which(train$Open=='1'),]
train <- train[ which(train$Sales!='0'),]

# seperating out the elements of the date column for the train set
train$Date <- as.Date(train$Date)
train$month <- as.integer(format(train$Date, "%m"))
train$year <- as.integer(format(train$Date, "%y"))
train$day <- as.integer(format(train$Date, "%d"))
train <- train %>% select(-Date)

test$Date <- as.Date(test$Date)
test$month <- as.integer(format(test$Date, "%m"))
test$year <- as.integer(format(test$Date, "%y"))
test$day <- as.integer(format(test$Date, "%d"))
test <- test %>% select(-Date)
```

```
names(train)
feature.names <- names(train)[c(1,2,5:10,12:13,17:23)]
feature.names
```

```
for (f in feature.names) {
  if (class(train[[f]])=="character") {
    levels <- unique(c(train[[f]], test[[f]]))
    train[[f]] <- as.integer(factor(train[[f]], levels=levels))
    test[[f]]  <- as.integer(factor(test[[f]],  levels=levels))
  }
}
```

```
cat("train data column names after slight feature engineering\n")
names(train)
cat("test data column names after slight feature engineering\n")
names(test)
```

```
RMPSE<- function(preds, dtrain) {
  labels <- getinfo(dtrain, "label")
  elab<-exp(as.numeric(labels))-1
  epreds<-exp(as.numeric(preds))-1
  err <- sqrt(mean((epreds/elab-1)^2))
  return(list(metric = "RMPSE", value = err))
}
```

```
nrow(train)

h<-sample(nrow(train),100000)

dval<-xgb.DMatrix(data=data.matrix(train[h,]),label=log(train$Sales+1)[h])

dtrain<-xgb.DMatrix(data=data.matrix(train[-h,]),label=log(train$Sales+1)[-h])

watchlist<-list(val=dval,train=dtrain)

param <- list(  objective           = "reg:linear", 
                booster = "gbtree",
                eta                 = 0.02, # 0.06, #0.01,
                max_depth           = 10, #changed from default of 8
                subsample           = 0.9, # 0.7
                colsample_bytree    = 0.7 # 0.7
                #num_parallel_tree   = 2
                # alpha = 0.0001, 
                # lambda = 1
)

clf <- xgb.train(   params              = param, 
                    data                = dtrain, 
                    nrounds             = 3000, #300, #280, #125, #250, # changed from 300
                    verbose             = 0,
                    early.stop.round    = 100,
                    watchlist           = watchlist,
                    maximize            = FALSE,
                    feval=RMPSE
)
```

```
pred1 <- exp(predict(clf, data.matrix(test[,feature.names]))) -1 
```

```
submission <- data.frame(Id=test$Id, Sales=pred1)
cat("saving the submission file\n")
write_csv(submission, "rf3.csv")

TEST <- test %>% filter(Open==0) %>% select(Id,Sales)
TEST$Sales[is.na(TEST$Sales)] <- 0
submission1 <- submission %>% filter(!(Id %in% TEST$Id))
submission2 <- bind_rows(submission1,TEST)
write_csv(submission2, "rf4.csv")
```
