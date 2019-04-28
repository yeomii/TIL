# Launching into Machine Learning

* 2019 구글 머신러닝 스터디 잼 중급반 필수 강좌 중 하나인 coursera 강좌를 정리한 글입니다.
* [링크](https://www.coursera.org/learn/launching-machine-learning/home/welcome)

## Intro
* 강좌를 통해 배우게 될 것
    * 현재 딥러닝이 이렇게까지 인기있는 이유
    * 손실 함수와 성능 지표를 통해 모델을 평가하고 최적화 하는 방법
    * ML 에서 흔히 발생하는 문제를 해결하는 방법
    * 반복가능한 training, evaluation, test 데이터셋을 만드는 방법

## Practical ML

### Supervised Learning
* 머신러닝에서 가장 널리 알려진 두 유형의 알고리즘
    * 지도학습 (supervised learning)
    * 비지도학습 (unsupervised learning)
    * 가장 큰 차이는, 지도학습의 데이터에는 label (학습되기를 원하는 정답) 이 있다는 것
* 비지도학습
    
    ![](./images/Income-vs-Job-tenure.png)
    * 예시 : 수입과 근속년수 간의 관계를 보고 빠르게 승진하는 (fast-track) 그룹을 찾는 것
    * 이처럼, 정답이 없는 데이터를 보고 자연스럽게 데이터 포인트들을 그룹짓는 방법을 발견하는 것
* 지도학습
    
    ![](./images/Restraurant-tips-by-gender.png)
    * 예시 ( 총 지불 금액을 보고 팁 금액 또는 성별을 예측 )
        * 회귀문제 : 팁 금액을 label 로 할 경우, 연속적인 값을 예측하기 때문에 회귀 모델 (regression, continuous label)
        * 분류문제 : 성별을 label 로 할 경우, 이산적인 값을 예측하기 때문에 분류 모델 (classification, categorical label)

* Regression & Classification
    * classification 에서는 각 카테고리로 분류될지를 결정하는 decision boundary 가 존재
    * structured data 는 feature 들과 label 이 table 의 형태로 구조화된 데이터이고
    * unstructured data 는 위와 다르게 feature 들이 잘 정의되지 않은 이미지, 비디오, 오디오 등의 데이터이다.

* 예시 데이터셋

    ![](./images/Regression-Classification.png)
    * 위 예제는 linear regression 모델과 linear classification 모델 둘다에 적합한 데이터셋이다.

    ![](./images/loss-function-difference.png)
    * 선형회귀 문제로 푸는 경우, MSE (Mean Square Error, 오차 제곱의 평균) 를 손실함수로 하여 이 값이 최소가 되는 초록색 선을 그을 수 있고,
    * 선형분류 문제로 푸는 경우, logistic regression 을 손실함수로 하여 노란색 선을 그을 수 있다. (분류가 3개 이상인 경우 Cross entropy 가 많이 사용된다)
    * 두 모델이 그리는 선이 다른 이유는, 손실함수에 따라서 오차 (정답과 예측값의 차) 에 대한 페널티의 크기가 다르기 때문이다.

### ML History
#### ML History : Linear Regression

#### ML History : Perceptron

#### ML History : Neural Networks

#### ML History : Kernel Methods

#### ML History : Random Forest

#### ML History : Modern Neural Networks

## Optimization

* loss fuction
    * 학습과정에서 사용됨
    * 이해하기 어려움
    * 비즈니스적 목표와 간접적으로 연결됨

* Performance metric
    * 학습 이후에 사용됨
    * 이해하기 쉬움
    * 비즈니스적 목표와 직접적으로 연결됨

### Confusion Matrix (혼동 행렬)
* 분류 모델의 성능을 설명하기 위해 자주 사용되는 표를 말한다
* 아래 그림은 binary classifier 에 대한 confusion matrix 이다.
    * ![](./confusion-matrix.png)
* TP (True Positive)
    * 모델의 예측 결과가 true 이고, 실제로 정답 true 인 경우
* FP (False Positive)
    * 모델의 예측 결과가 true 이고, 실제 정답은 false 인 경우
* FN (False Negative)
    * 모델의 예측 결과가 false 이고, 실제 정답은 true 인 경우
* TN (True Negative)
    * 모델의 예측 결과가 false 이고, 실제 정답도 false 인 경우
* Precision
    * `TP/(TP+FP)`
    * true 라고 예측한 것 중 맞게 예측한 것의 비율
    * FP 가 증가하면 Precision 은 감소한다
* Recall
    * `TP/(TP+FN)`
    * 실제 정답이 true 인 것 중 맞게 예측한 것의 비율
    * sensitivity 라고도 한다
* Accuracy 
    * `(TP+TN)/Total`
    * 전체 데이터 중 모델이 맞게 예측한 것의 비율


