# Learning to Represent Entities by Ranking Them

## abstract
```
Word2vec이나 matrix factorization과 같은 representation learning 알고리즘들은 현대의 기계학습 및 추천 시스템에 핵심적인 feature extractor로 사용되고 있습니다. 이러한 알고리즘들은 대개 부가적으로 정의된 분류 분석 문제를 대상으로 학습되는데 (예: 단어 queen과 woman이 같은 맥락에서 사용되었는지 분류), 과연 분류 분석이 이상적인 학습 대상인지는 분명치 않은 측면이 있습니다. 본 강의에서는 그 대안으로써 대상을 정렬하는 ranking문제가 자연어 처리와 추천 시스템의 맥락에서 분류 분석 문제와 대비해 어떤 장점들이 있고, 어떻게 이러한 ranking 문제의 학습을 분산 처리 환경에서 병렬화해 대용량 데이터에 적용할 수 있는지를 다루고자 합니다.
```

* https://papers.nips.cc/paper/5363-ranking-via-robust-binary-classification.pdf
* https://arxiv.org/pdf/1506.02761.pdf

## contents
* robust classification
* learning to rank
* latent collaborative retrieval
* word embeddings

## Summary ??
* 학습 목적에 따른 loss function 선택이 중요
    * 검색 결과의 경우 첫 페이지 첫 번째 항목과 다음 페이지의 항목의 중요성이 매우 차이남
* robust statistics 와 ranking 은 큰 연관성이 있음

## keyword
* learning to rank
    * 검색 엔진에서 많이 쓰임
    * 가장 중요한 컨텐츠들을 검색 결과 상위에 위치시키는 역할

* representation learning
    * http://hugman.re.kr/blog/repr_learning/

### binary classification
y_i * (x_i * w) < 0 => prediction 이 틀림
이걸 목적함수로 쓰면 미분 불가하기 때문에 GD 로 학습할 수 없음
대신 logistic loss 로 optimize 학습
hinge loss 를 쓰기도 함
e 대신 log_2 를 쓰는 이유가 따로 있음
convex loss function
    * 데이터를 잘못 labeling 했을 때 잘못된 영향을 크게 미침

### robust binary classification
* loss 가 inf 로 천천히 감
* convex loss function 의 한계를 커버할 수 있음
* type I loss & type II loss function
    * type I - non convex 이지만 좋은 성질들을 가지고 있음
* asymptotic

### Learning to rank
* ranking 을 할 때 모든 position 이 중요하지 않음
* 검색 결과에서도 첫 페이지 첫 번째 항목과 다음 페이지의 항목의 중요성이 매우 차이남
* notations
    * x : user
    * y : items
    * r_{xy} : rating user x assigned to item y
* 목적 함수를 logistic loss 로 치환

### Discounted Cumulative Gain
* DCG 
* 검색 시스템을 평가하는 metric

### RoBiRank
* Type I loss function 을 사용해서 optimize 하는 것이 더 쉬움

* 위 방법들은 feature 를 사람이 직접 뽑는다

================= classical ML ^

* feature vector design 을 알아서 하게 하는 것이 최근 연구

### Latent Collaborative Retrieval
* 실제 데이터에서는 
    * feature vector design 이 어려움
    * 직접적인 라벨링보다는 click 과 같은 implicit feedback 이 많음
* low-dimensonal embedding 사용
* 클릭한 아이템에 대해서 아이템의 포지션이 상위 포지션에 위치하도록 object function design

* bayesian personalized ranking
    * 계산이 다 summation
    * 효율적

### stochastic optimization
* upperbound 활용

### parallelization
* SGD 를 쓰면 분산 optimization 을 할 수 있어 유리

### Word Embeddings
* NLP 의 최근 연구들도 Word2vec 영향을 많이 받음
* word2vec 
* Named Entity Recognition
* WordRank
    * 특정 단어들의 시퀀스가(context) 있을 때 같이 나타날 단어들을 ranking
* WordRank 방법으로 Word Representation 을 학습

## Summary

* 학습 목적에 따른 loss function 선택이 중요
* robust 통계와 ranking 은 큰 연관성이 있음
