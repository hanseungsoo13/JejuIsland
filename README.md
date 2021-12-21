# JejuIsland

## Summary

최근 제주도가 음식물 쓰레기 문제로 인해 어려움을 겪고 있다는 기사를 많이 접할 수 있었다. 이에 제주시는 음식물 쓰레기 처리 시설에 대한 효율성을 필요로 하고 있다. 이에 본 프로젝트에서는 군집분석과 회귀적 피쳐셀렉션을 통해 음식물 쓰레기 배출에 영향을 주는 요인들을 분석하고 ARIMA와 LSTM등 여러가지 시계열 방법론을 적용하여 배출량을 예측한다. 

## DataPreparing

- 활용 데이터
    1. 음식물 쓰레기 데이터(2018.01~2021.06)
    2. 내국인 유동인구(2018.01~2021.06)
    3. 외국인 유동인구(2018.01~2021.06)
    4. 음식관련 카드소비(2018.01~2021.06)
    5. 코로나19 데이터 (2020.03~2021.08)

## EDA

- **코로나 데이터 사용 배경**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e188384-64e6-489e-9fb1-608d3882cb22/Untitled.png)
    
    - 제주도 전체 음식물 쓰레기 배출량의 변화량을 보면 **2020년 이후 음식물 쓰레기 배출량이 전년도에 비해 증가**하고 있음을 알 수 있다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/873a10af-4699-47c2-bb91-676ba9c76e3f/Untitled.png)
    
    - 이에 대한 원인을 찾아본 결과, 위 기사와 같이 **코로나19로 인해** 해외여행에 가지 못하는 내국인 관광객들이 **제주도로 몰리게 된 것**이 원인으로 나타남
    - 따라서 코로나19의 발생현황이 제주도 관광에 영향을 주어 음식물 쓰레기의 증가에 크게 영향을 준다고 판단하여 **코로나19 데이터를 활용**하기로 결정
- **시계열 분석**
    
    제주시 건입동에 대해 **음식물 쓰레기 배출량에 대한 시계열 분해**를 진행
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8aa75d6f-cf55-4602-bd3f-19b7c711d052/Untitled.png)
    
    - 시계열 분해 결과 각 데이터가 **일정한 주기를 가지고 변화**하는 것을 볼 수 있다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7952b3a9-3853-47e4-a1c0-546274295a4c/Untitled.png)
        
    - 원데이터를 시각화 해보았을 때, 대체적으로 **1년 주기로 비슷한 패턴**이 나타났다.
    - 시계열 모델을 통한 분석이 용이할 것이라고 판단하였다.
- **Clustering의 필요성**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db9d395e-1899-4159-8e26-3292c804393f/Untitled.png)
    
    - 각 읍면동 단위별로 **음식물 쓰레기 배출량의 크기가 다르기 때문**에 스케일의 특성이 담긴 군집들로 나누어 분석을 진행할 피료가 있다고 판단하였다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd19176a-8ea3-4dea-ae53-1bb82677b06a/Untitled.png)
        
    - 각 동별로 음식물 쓰레기 배출량의 스케일이 다르기 때문에 스케일링을 진행하고 시각화를 한 결과이다.
    - 스케일이 일정하더라도 **읍면동의 특징에 따라 월별 음식물 쓰레기 배출량의 분포가 다른 것**을 볼 수 있다.
    - 분포가 비슷한 읍면동들끼리 군집을 형성하여 분석을 진행할 필요가 있다고 판단하였다.
    
    ## Clustering
    
    - **Robust Scaling**
        - 중앙값(median)과 IQR을 사용한 Scaler
        - 아웃라이어의 영향을 최소화 하여 극단값의 영향을 받지 않는다.
        - StandardScaler에 비해 표준화 후 동일 값을 더 넓게 분포할 수 있다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5943229-dd7c-4310-b8e0-9fe1f56c3b8d/Untitled.png)
        
                                       Robust Scaling전 (위)            /            Robust Scaling후(아래)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ce2792b0-eecd-45c3-9401-00f351d29892/Untitled.png)
        
        Robust Scaling을 진행한 후 군집별로 색상을 달리하여 합계를 확인한 결과, 이전보다 색상별로 뚜렷하게 군집의 특성을 반영한다고 판단
        
    - **K-means Clustering**
        - 비계층적 군집분석 방법 중 하나로 중심기반(Center-based)클러스터링 방법이다.
        - 원하는 군집 수(k)만큼 초기중심값을 정하고 각 개체를 가까운 초기값과 군집을 형성한 뒤, 형성된 군집의 중심값을 다시 구하고, 다시 개체들을 가까운 군집의 중심값과 군집을 형성시키는 과정을 반복하여 k개의 최종 군집을 형성하는 과정이다.
        - **군집 수 k를 정하는 과정**
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/60eb106a-3d86-4c0b-a6a9-474f7fac50a2/Untitled.png)
            
            **SSE와 Silhouette score가 가장 일치**하는 k=6이 가장 적절한 군집수로 보인다.
            
        - **Silhouette Score**
            - 군집 **내** 비유사성은 **작고**, 군집**간** 비유사성은 **큰** 클러스터를 찾는 것을 목적으로 한다.
            - 아래의 식처럼 **각 데이터 포인트와 주위 데이터 포인트들과의 거리 계산**을 통해 값을 구한다.
            - i의 실루엣 계수
                
                ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24f34417-88a0-409d-a425-4cdd547fb683/Untitled.png)
                
                - a(i): 데이터 포인트 i가 속한 클러스터 내 데이터 포인트들과 거리의 평균
                - b(i): 데이터 포인트 i가 속하지 않은 클러스터의 데이터 포인트들과의 거리 평균
            - s(i)가 0에 가깝다면 두 클러스터의 거리가 비슷한 것을 의미하며 군집화가 잘 되지 않은 것이다.
            - s(i)가 -1에 가깝다면 데이터 포인트가 할당된 군집보다 더 이웃한 군집이 있는 것이다. 잘못 클러스터링 된 경우이다.
            - s(i)가 1에 가까울수록 군집화가 잘 된것이라고 생각할 수 있다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/48dc14fa-5a49-4143-bc28-694527a741f6/Untitled.png)
        
        실루엣 스코어를 통해 보았을 때 **군집 6이 가장 1에 가까운 점수**를 보였다.
        
        두가지의 지표를 통해 군집의 갯수는 6개로 선정하였다.
        

## Feature Engineering

- 각 데이터별 **기초통계량을 활용**
- 날짜별, 동별로 범주형 변수에 해당하는 항목에 대해 수**치형 변수에 대한 평균과 합계**를 내어 구체적인 정보를 담음
- 외국인의 유동인구 특성상 sparse한 데이터였기 때문에 **군집을 형성**하여 feature로 활용

## Feature Selection

- 선형회귀 피쳐 셀렉션
    
    전체적인 Feature Selection과정은 선형회귀분석 방식을 활용한다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4e77893-d8fc-4a67-b367-686cb7aecd05/Untitled.png)
    
    1. **F검정**
        - F검정을 통해 **모델의 유의성**에 대해 검정한다.
        - F검정의 p-value가 0.05보다 작으면 회귀모형이 유의하다는 것으로 판단한다.
    2. **T검정**
        - T검정을 통해 **피쳐의 유의성**에 대해 검정한다.
        - 각 변수별로 p-value값이 0.05이상이면 유의하지 않다고 판단하고 제거한다.
    3. **데이터의 선형성 확인**
        - 잔차그래프를 통하여 **데이터의 선형성을 확인**한다.
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a1a663b-9e98-48db-9c9a-b87cbfbaf8c8/Untitled.png)
            
        - 다음과 같이 비선형성은 나타나지 않는 것으로 보인다.
    4. **오차의 상관성 확인**
        - **더빈-왓슨 검정**을 통해 **오차들간의 자기상관이 있는지 확인**한다.
        - Durbin-Watson 검정
            - **오차항에 자기상관이 있는지** 없는지를 판단하기 위해 사용하는 검정법이다.
            - 더빈 왓슨 통계량이 2에 가깝다면 자기상관이 없다고 판단
    5. **이분산성 확인**
        - 회귀식의 가정에서 **오차의 등분산성**을 만족하는지를 확인하는 과정이다.
        - 잔차그래프를 통해 이분산성이 있는지를 확인한다.
        - 이분산성이 확인되면 **log변환**을 통해 이 문제를 해결한다.
    6. **이상치와 영향점 확인**
        - 이상치 판단기준으로는 Cook’s Distance를 활용한다.
        - **Cook’s Distance**
            - 데이터의 레버리지와 잔차를 동시에 보기 위한 기준
            - 레버리지나 잔차가 커지면 Cook’s Distance값도 커진다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/98948147-a312-4c27-86ca-7f3b38676147/Untitled.png)
        
        영향점이나 이상치는 존재하지만 이 수치가 시계열분석에서는 유의미하다고 판단되어 제거하지 않음
        
    7. **다중공산성 확인**
        - **다중공산성**
            - **변수들간의 상관관계가 높은 것**을 다중공산성이라고 한다.
            - 회귀분석의 가정 중 하나인 **변수들간의 비상관성을 위배**하는 것을 의미함
        - 다중공산성은 **VIF**값을 통해 판단
        - VIF값이 10보다 크면 다중공산성이 존재한다고 판단하고 변수들을 제거한다.

## Modeling

- **Overview**
    
    전체적인 모델링 과정은 다음과 같다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1641cfef-b982-40a2-85f1-25e2d875a227/Untitled.png)
    
    1. ARIMA, Linear Regression, Machine Learning, Deep Learning 모델들을 이용하여 ValidationSet인 2021년6월의 데이터를 예측
    2. 각 모델별 최적의 예측 방법론 선정 및 모델을 구축
    3. 실제 타겟인 2021년 7,8월 예측
    4. 예측결과 앙상블
- **ARIMA**
    - ARMA모형에 차분(Difference)를 이용한 모델로, 시계열의 비정상성을 설명하기에 용이
    - 과거의 데이터가 지니고 있던 ‘추세’까지 반영
    - **ARIMA 모델링**
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c014cbb1-d51f-4ee0-ba51-6ebf73700ee8/Untitled.png)
        
        1. 코싸인 유사도를 이용한 데이터셋 구축
            - 시계열 데이터의 특성상 연속성을 갖는 데이터셋을 구축하는 것이 중요하였다.
            - 2021년 8월을 예측함에 있어 2021년 7월의 데이터가 없는 것이 치명적이었다.
            - 2021년 6월의 데이터와 코싸인 유사도를 갖는 달들을 통해 2021년 7월과 유사할 것이라고 판단되는 달을 선정하여 평균내어 대치하였다.
        2. 정상성 검정
            - ADF 검정을 통해 정상성을 확인한다.
            - 정상성이 있지 않다고 판단되면 차분(Difference)를 통해 정상성을 확보한다.
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9c8ea2ac-b182-4b43-bb91-f36a5571f5dc/Untitled.png)
            
        3. Auto ARIMA
            - ARIMA는 차수(p,d,q)에 큰 영향을 받는다.
            - AIC를 기준으로 자동으로 최적의 차수(p,d,q)를 찾는 방법
        4. ARIMAX
            - 기존의 Arima 모형에서 외부변수를 활용하는 다변량 시계열 모형
        5. 잔차분석
            - 뚜렷한 잔차의 패턴이 보이지 않는다면 적절하게 분석이 이루어진 것이다.
- **Linear regression**
    - **선형회귀 모델링**
        - 최소자승법을 이용하는 선형회귀 모형을 통해 미래를 예측한다.
            - **최소자승법 (OLS)**
                
                잔차제곱합(RSS)를 최소화하는 가중치 벡터를 구하는 방법
                
- **Machine Learning**
    
    머신러닝을 이용한 예측과정은 다음과 같다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee264cde-c232-4c02-b466-1aeaa734aa24/Untitled.png)
    
    - **Models**
        1. **XGB Regressor**
            - Gradient Boosting을 활용한 지도학습 회귀 예측 모델로 시계열 예측에 활용될 수 있는 머신러닝 모델
        2. **LGBM Regressor**
            - XGB와 마찬가지로 Gradient Boosting방식의 지도학습 회귀 예측 모델로 메모리를 적게 차지하고 속도가 빠른 장점을 가진 머신러닝 모델
    - **Parameter Tuning**
        1. **Bayesian Optimizer**
            - 사후분포의 확률을 기반으로 하는 베이즈 확률론을 활용한 파라미터 튜닝 방법
            - 지속적인 학습을 통해 예측값과 분산을 바탕으로 기대  향상을 얻어낸다.
    - **After Processing**
        - **이상치 제거**
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f223299-14c4-4a35-95ee-eed4e01e8451/Untitled.png)
            
            이상치 제거 전, 후의 변화이다. 이상치를 제거하여 예측값을 Smooth하게 변환할 수 있었다.
            
        - **Ensemble**
            
            가장 적절한 예측값을 뽑아내기 위해 각각의 방법론, 모델들을 모두 고려한 앙상블을 진행하였다.
            
- **Deep Learning**
    
    딥러닝을 이용한 예측과정은 다음과 같다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2bb6ded6-f5ad-441b-84af-6e14b6a293c4/Untitled.png)
    
    - Models
        1. **LSTM**
            - RNN이 출력과 먼 위치에 있는 정보를 기억할 수 없다는 단점을 보완하여 장,단기 기억을 가능하게 설계한 신경망 구조
        2. **GRU**
            - LSTM을 구성하는 Time-Step의 cell을 간소화한 버전

## Ensemble

- **Combination**
    
    자체 Validation Set인 2021년 6월의 예측값을 통해 각 군집별로 최적의 모델 조합을 찾는다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/951dc0a7-817a-4abb-b927-6d7527531936/Untitled.png)
    

## Conclusion

- **7월 예측**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac74fe8e-5797-4e73-8c13-486b558340b2/Untitled.png)
    
- **8월 예측**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5f98e8d-9b9c-4e45-b7bf-2758b79bcf19/Untitled.png)
    

### Solution

- **공통특성**
    - 외국인 인구 관련 Feature들의 중요도가 높게 나타났다.
        
        ⭐특히 단기 체류가 아닌 장기 체류 외국인이 많았다.
        
    - 관광객이 많기 떄문에, 배달 및 외식 건수가 많았다.
    - 카드 소비 관련 Feature들이 중요도가 높게 나타남
- **각 군집별 특징**
    1. **Cluster 0**
        - 관광지가 존재하지만, 주요 관광지보다는 상대적으로 유동인구가 적음
            
            ⭐관광객 보다는 실거주자들의 영향이 큰 것으로 예상
            
        - ‘80대 이상의 거주인구’의 중요도가 높게 나왔다.
            
            ⭐고령인구가 상대적으로 많음
            
    2. **Cluster1**
        - 점심시간 근로자 유동인구 관련 변수의 중요도가 높다.
            
            ⭐직장인이 많을 것으로 추정
            
        - 다른 군집에 비해 음식 카테고리 중 한식 비중이 높다
        - 배달음식을 많이 시키는 것으로 나타난다.
        - 인구가 많은 편에 속한다.
    3. **Cluster2**
        - 주로 번화가 및 거주지에 해당한다.
        - 유동인구가 많다.
        - 아침에 근로자 유동인구 관련 변수의 중요도가 높다.
            
            ⭐출근시간 유동인구가 많았다.
            
        - 마트, 슈퍼마켓 이용 건수가 많다.
        - 해수욕장이 있어, 여행 성수기인 7,8월에 음식물 쓰레기 배출량이 크게 증가한다.
    4. **Cluster3**
        - 제주항, 제주시외버스터미널, 제주국제공항들이 위치하여 유동인구는 많으나 음식물 쓰레기 배출량이 적음
        - 관광지로서의 특징이 다소 적음
    5. **Cluster4**
        - 성산읍과 애월읍 모두 관광지에 해당
        - 새벽에 방문객 유동인구의 중요도가 높다
            
            ⭐유명관광지 성산일출봉 관광객의 중요도
            
        - 20대 거주인구가 많다.
        - 코로나로 인해 제주도 관광객이 급증하였다.
            
            ⭐방문객이 크게 증가하였다.
            
            ⭐2020년을 기점으로 음식물 쓰레기 배출량이 크게 증가하였다.
            
    6. **Cluster5**
        - 인구가 가장 많은 상위 동들이 포함되있다.
            
            ⭐음식물 쓰레기 배출량이 가장 많은 군집에 해당된다.
            
    7. **Cluster6**
        - 중국인 관광객 관련 변수의 중요도가 높다.
            
            ⭐ 중국인 관광객들이 많이 찾는 관광지가 포함되어있다.
            
        - 구좌읍, 한경면은 음식물 쓰레기 배출량이 다소 적다.
- **음식물 쓰레기 배출량 감소 방안**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50f6f73b-6d77-42b1-97c6-2e38103bc3a1/Untitled.png)
