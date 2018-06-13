`week5.pdf` : 실습자료


15129의 관측치와 21 variables를 가진 train.csv 파일을 가지고 test.csv의 price를 예측하는 실습. method는 linear regression을 이용해야 하고, 평가 방법은 RMSLE라는 것을 이용해서 구할 것임.

[기존]
[1] RMSLE : 0.9280684  
[2] Adjusted R-squared:  0.7013   

[이후]
[1] Rmsle: 0.2488122    
[2] Adjusted R-squared:  0.7701    

한계:  
RMSLE에 대해서는 좋은 상승을 이루었으나, R-squared 같은 경우는 상승폭이 미비함. 이는 Feature engineering과 Asuumptions의 문제로 데이터 전처리 과정이 부족했다고 생각함.

![](https://choco9966.github.io/Team-EDA/5week/image/1.PNG)

실제로 Skewness와 Heteroscedasticity는 p-value가 매우높은것을 볼 수 있음.
