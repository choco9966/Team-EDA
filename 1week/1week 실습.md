# 1주일차: EDA (Exploratory Data Analysis) 실습

```
install.packages("dplyr") 
install.packages("ggplot2") 
install.packages("Hmisc") 
install.packages("PerformanceAnalytics") 
install.packages("corrplot")
install.packages("gridExtra")
```

```
library(gridExtra)
library(dplyr) 
library(ggplot2) 
library(Hmisc) 
library(PerformanceAnalytics) 
library(corrplot)
```

### 목표 : Titanic 데이터를 가지고 EDA가 무엇인지 살펴보기.

# *1.변수설명 확인*
* #### Data desciption 


|Variable   |Definition   |Key   |
|---|---|---|
|Survived|Survival|0 = No, 1 = Yes|
|pclass|Ticket class|1 = 1st, 2 = 2nd, 3 = 3rd|
|sex|Sex||
|Age|Age in years||
|sibsp|# of siblings / spouses aboard the Titanic||
|parch|# of parents / children aboard the Titanic||
|ticket|Ticket number||
|fare|Passenger fare||
|cabin|Cabin number||
|embarked|Port of Embarkation|C = Cherbourg, Q = Queenstown, S = Southampton|

* ####  Variable Notes

**pclass:** A proxy for socio-economic status (SES)
1st = Upper
2nd = Middle
3rd = Lower

**age:** Age is fractional if less than 1. If the age is estimated, is it in the form of xx.5

**sibsp:** The dataset defines family relations in this way...
**Sibling =** brother, sister, stepbrother, stepsister
**Spouse =** husband, wife (mistresses and fiancés were ignored)

**parch:** The dataset defines family relations in this way...
**Parent =** mother, father
**Child =** daughter, son, stepdaughter, stepson
Some children travelled only with a nanny, therefore parch=0 for them.

# *2.데이터 확인*

```
#이전에 설명했듯이 변수가 치우쳐있는 경우를 생각해 head/tail/sample등 여러 방식을 이용한다.
str(train)
head(train)
tail(train)
train[sample(nrow(train),6),]
```

```
> str(train) 
'data.frame': 891 obs. of  11 variables:  
#read.csv로 파일을 불러들였기 때문에 data.frame을 가짐.

$ survived: int  0 1 1 1 0 0 0 0 1 1 ... 
$ pclass  : int  3 1 3 1 3 3 1 3 3 2 ... 
#survived나 pclass 같은 경우에는 type이 int가 아닌 Factor임. 하지만 컴퓨터가 파일을 읽어들일때 이를 구별하지 못해서 따로 수정작업이 필요함!!!.


$ name    : Factor w/ 891 levels "Abbing, Mr. Anthony",..: 109 191 358 277 16 559 520 629 417 581 ... 
#name의 같은 경우 Factor로 해도 상관은 없으나, 891로 levels이 너무 많아서 chr로 처리해주는게 더 편함.

$ sex     : Factor w/ 2 levels "female","male": 2 1 1 1 2 2 2 2 1 1 ... 
$ age     : num  22 38 26 35 35 NA 54 2 27 14 ... 
#NA값의 처리에 대해서도 고민해봐야함.

$ sibsp   : int  1 1 0 1 0 0 0 3 0 1 ... 
$ parch   : int  0 0 0 0 0 0 0 1 2 0 ... 
$ ticket  : Factor w/ 681 levels "110152","110413",..: 524 597 670 50 473 276 86 396 345 133 ... 
$ fare    : num  7.25 71.28 7.92 53.1 8.05 ... 
$ cabin   : Factor w/ 148 levels "","A10","A14",..: 1 83 1 57 1 1 131 1 1 1 ... 
$ embarked: Factor w/ 4 levels "","C","Q","S": 4 2 4 4 4 3 4 4 4 2 ...
#똑같은 결측치인데 age는 NA로 들어가고, embarked은 ""으로 들어갔음. is.na(train$embarked) 명령어를 치면 ""은 NA로 인식되지 않음. 그래서 수정해줘야하는걸 확인. 
```

# *3.데이터 전처리*
```
# data type 변경. as.factor()/as.character()/as.numeric()
train$survived <- as.factor(train$survived) 

# lapply를 이용해서 한번에 여러개 변경가능. lapply는 모든 list,vector에 대해서 함수(여기서는 as.factor)를 적용
train[, c("survived","pclass","ticket","embarked")] <- lapply(train[, c("survived","pclass","ticket","embarked")], as.factor)
```

```
# NA값 처리
# levels을 이용한 방법
levels(train$embarked)[1] <- NA
levels(train$cabin)[1] <- NA
# ifelse(조건,조건이 TRUE일 경우 ,FALSE일 경우)
train$embarked <- ifelse(train$embarked == "", NA ,train$embarked)
train$cabin <- ifelse(train$cabin == "", NA ,train$cabin)
# Missing value를 추론해서 넣는것은 다음week에 진행.
```
# *4.기본적인 분석*
## *4.1기본적인 통계분석*

```
summary(train)
table(train$survived)
summary(survived ~ pclass+sex+sibsp+parch+embarked,data=train, method="reverse")
```

```
>summary(survived ~ pclass+sex+sibsp+parch+embarked,data=train, method="reverse")
```
![](https://choco9966.github.io/Team-EDA/1week/image/c1.png)
```
해석은 survived가 0이냐 1이냐에 따라서 값들을 분류해놓은것임.
첫번째줄인 pclass 가 1,2,3 중 1일때, survived가 0인 관측치는 80개 이고, 
1인 관측치는 136개임. 15%는 80/(80+97+372) 즉, 죽은사람들(survived=0) 중에서
15%가 여기에 속하는걸 알 수 있음.


이러한 summary는 유용한 정보를 얻을 수 있는데 
1. pclass가 좋을수록(1등급일수록) 생존확률이 높은 것을 확인 가능.
2. 죽은사람의 85%가 male인것을 봐서 여성일 경우 많이 생존한것을 확인 가능.
3. sibsp = 0 인 경우보다 1 or 2인 경우에 생존률이 월등히 높은걸 확인 가능.
기타 등등...
```
## *4.2기본적인 시각화분석*

|데이터   |요약통계   |시각화   |
|---|---|---|
|카테고리-카테고리|교차테이블|모자이크플롯
|수치-수치|상관계수|산점도
|카테고리-수치|카테고리 별 통계값|박스플롯

```
# 카테고리 - 카테고리
table(train$survived,train$sex)
mosaicplot(survived ~ sex  , data=train , color = TRUE) # 모자이크 플롯
mosaicplot(survived ~ sex + pclass , data=train , color = TRUE)
```
![](https://choco9966.github.io/Team-EDA/1week/image/mosaic.png)

```
# 상관분석
cor(train$age,train$sibsp) #NA가 뜰 경우 NA값이 있어서 그런거임. 그럴 경우 use = "complete.obs"를 붙여줌.
plot(train$age,train$sibsp)

# package 이용한 방법.
num_data <- train %>% select("age","sibsp","parch","fare")
m <- cor(num_data,use="complete.obs") # use="complete.obs"는 NA값을 무시하고 진행하라는 명령어.
corrplot(m,method="number") #method에 따라서 그림이 다름. circle 치면 원형태로 나옴.
```
![](https://choco9966.github.io/Team-EDA/1week/image/cor.png)

```
# 상자 그림
boxplot(age ~ embarked, data= train)
```
![](https://choco9966.github.io/Team-EDA/1week/image/boxplot.png)

```
위에서 확인한 summary정보 or 개인적으로 만든 가정들을 시각화를 통해 확인할 수 있음. 
ex) pclass가 높을수록 생존률이 높은지 / 성별에 따른 죽은 사람 / 자식여부에 따른 생존 등등 그래프로 확인
```

```
# pclass 등급에 따른 생존비교
p1 <- ggplot(train, aes(x = pclass, fill = survived)) +
  geom_bar(stat='count', position='dodge') +
  labs(x = 'pclass, train data') + geom_label(stat='count', aes(label=..count..)) + theme(legend.position="none") + theme_grey()

p1
# pclass가 높을수록 생존률이 높은거 확인가능.
```
![](https://choco9966.github.io/Team-EDA/1week/image/c2.png)

```
# 성별에 따른 비교
p2 <- ggplot(train, aes(x = sex, fill = sex)) +
  geom_bar(stat='count', position='dodge') + theme_grey() +
  labs(x = 'train data') +
  geom_label(stat='count', aes(label=..count..)) +
  scale_fill_manual("legend", values = c("female" = "pink", "male" = "green"))

p3 <- ggplot(train[!is.na(train$survived),], aes(x = sex, fill = survived)) +
  geom_bar(stat='count', position='dodge') + theme_grey() +
  labs(x = 'Training data only') +
  geom_label(stat='count', aes(label=..count..))

grid.arrange(p2,p3, nrow=1) # 그래프를 한번에 보여주기 위한 명령어임.
# 남성보다 여성이 생존률이 높은거 확인가능.
```
![](https://choco9966.github.io/Team-EDA/1week/image/c3.png)

```
# sibsp에 따른 비교
p4 <- ggplot(train, aes(x = sibsp, fill = survived)) +
  geom_bar(stat='count', position='dodge') +
  labs(x = 'sibsp, train data') +
  theme(legend.position="none") + theme_grey()

p4
# 자식이 있을수록 생존률이 높은거 확인가능.
```
![](https://choco9966.github.io/Team-EDA/1week/image/c4.png)

```
# age에 따른 비교
p5 <- ggplot(train, aes(x=age, fill= survived)) + geom_histogram(binwidth=5) 
p5
# 어린아이들이 생존률이 비교적 높음
```
![](https://choco9966.github.io/Team-EDA/1week/image/c5.png)

## *4.3외부 정보활용*
```
cabin사진을 통해 대략적인 구조확인
```
![](https://choco9966.github.io/Team-EDA/1week/image/cabin.PNG)

```
pclass사진을 통해 빨간색으로 동그라미 친 구명보트와 가까운 위치의 숙박사람들이 생존률이 높을거라는걸 예상가능.
주황색:pclass=1 , 초록색:pclass=2 , 파란색:pclass=3
```
![](https://choco9966.github.io/Team-EDA/1week/image/pclass.PNG)

```
embarked 위치를 확인함으로써 ticket비용, pclass 위치가 다를거라는 예상을 할 수 있음.
```
![](https://choco9966.github.io/Team-EDA/1week/image/embarked.png)
