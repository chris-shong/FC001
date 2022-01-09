# Part 2. Apache Spark와 데이터 병렬-분산 처리 Review

## Apache Spark
```local:4040/jobs```에서 Spark job 확인 가능

---
## Chapter 02. 병렬처리에서 분산처리까지
- Resilient Distributed Dataset(RDD)
  - 데이터는 클러스터에 흩어져있지만 하나의 파일인것 처럼 사용이 가능하다.
  - Resilient & Immutable, 탄력적이고 불변하는 성질이 있다: RDD에 대한 변환이 이루어질 때마다 새로운 RDD가 생성되고 그 과정을 하나의 비순환 그래프(Acyclic Graph)로 그릴 수 있게 된다. RDD 파일에 문제가 생길 경우 다른 RDD를 사용하여 복원이 가능하다. 새로운 RDD는 action 시에 만들어진다.
  - Type-safe: 컴파일시 Type을 판별할 수 있어 문제를 일찍 발견할 수 있다.
  - Unstructured / Structured Data 둘 다를 다룰 수 있다.
      - ex. Unstructured: log, 자연어 / Structured: RDD, DataFrame
  - Lazy Evaluation: 결과가 필요할 때(Action)까지 연산을 하지 않는다.


- Spark Operation = Transform + Action
  - Transformations: 결과값으로 새로운 RDD를 반환, Lazy Execution
  - Actions: 결과값을 연산하여 출력하거나 저장, Eager Execution


- Transformation = Narrow + Wide
  - Narrow: 1:1 변환, 1열을 조작하기 위해 다른 열/파티션의 데이터를 사용할 필요가 없다. 정렬이 필요하지 않은 경우
  - Wide: Shuffling, Output RDD의 파티션에 다른 파티션의 데이터가 들어갈 수 있음


- cache()와 persist()로 데이터를 메모리에 저장해두고 사용이 가능
  - Cache: default Storage level 사용.
      - RDD: MEMORY_ONLY
      - DF: MEMORY_AND_DISK
  - Persist: Storage Level을 사용자가 원하는대로 지정 가능


- 스파크를 쓰면서 잊지 말아야 할 점
  - 항상 데이터가 여러 곳에 분산되어 있다는 점
  - 같은 연산이어도 여러 노드에 걸쳐서 실행된다는 점


### Partitioning 을 이용한 성능 최적화
- Shuffle을 최소화 하려면
  - 미리 파티션을 만들어 두고 캐싱 후 reduceByKey 실행
  - 미리 파티션을 만들어 두고 캐싱 후 join 실행
  - 둘다 파티션과 캐싱을 조합해서 최대한 로컬 환경에서 연산이 실행되도록 하는 방식
  - 셔플을 최소화 해서 10배의 성능 향상이 가능하다.


- 파티션의 특징
  - RDD는 쪼개져서 여러 파티션에 저장됨
  - 하나의 파티션은 하나의 노드(서버)에
  - 하나의 노드는 여러개의 파티션을 가질 수 있음
  - 파티션의 크기와 배치는 자유롭게 설정 가능하며 성능에 큰 영향을 미침
  - Key-Value RDD를 사용할 때만 의미가 있다.


- 스파크의 파티셔닝 == 일반 프로그래밍에서 자료 구조를 선택하는 것


- 디스크에서 파티션하기
  - partitionBy(): 보통 많이 사용, Transformation


- 메모리에서 파티션하기: 파티션의 개수를 조절하는데 사용하며 둘다 shuffling을 동반하여 매우 비싼 작업이다.
  - repartition(): 파티션의 크기를 줄이거나 늘리는데 사용됨
  - coalesce(): 파티션의 크기를 줄이는데 사용됨


- 파티션을 만든 후에 persist()하지 않으면 다음 연산에 불를 때마다 반복하게 된다.(셔플링이 반복적으로 일어난다.)

---

## Chapter 03. Structured vs Unstructured Data

- Unstructured: free form
  - log 파일, 이미지
- Semi Structured: 행과 열
  - CSV, JSON, XML
- Structured:행과 열 + 데이터 타입(스키마)
  - 데이터 베이스


- RDD에선
  - 데이터의 구조를 모르기 때문에 데이터를 다루는 것을 개발자에게 의존
  - Map, flatMap, filter등을 통해 유저가 만든 function을 수행

- Structured Data에선
  - 데이터의 구조를 이미 알고 있으므로 어떤 테스크를 수행할 것인지 정의만 하면 됨
  - 최적화도 자동으로 할 수 있음


- Spark SQL 소개
  - 스파크 위에 구현된 하나의 패키지
  - 3개의 주요 API
      - SQL
      - DataFrame
      - Datasets
  - 2개의 백엔드 컴포턴트
      - Catalyst - 쿼리 최적화 엔진
      - Tungsten - 시리얼라이저
  - Spark Core에 RDD가 있다면 Spark SQL엔 DataFrame이 있다. 개념적으론 RDD에 스키마가 적용된 것으로 보면 된다.
  - Spark Core에 SparkContext가 있다면 Spark SQL엔 SparkSession이 있다.


- RDD로부터 DataFrame 만들기
  - Schema를 자동으로 유추하여 DataFrame 만들기
  - Schema를 사용자가 정의하기
  - 파일 읽어오기


- createOrReplaceTempView
  - DataFrame을 하나의 데이터베이스 테이블처럼 사용하려면 createOrReplacetempView() 함수로 temporary view를 만들어줘야 한다.


- SparkSession으로 불러오는 데이터는 DataFrame


- Datasets
  - Type이 있는 DataFrame
  - PySpark에선 크게 신경쓰지 않아도 된다.


- DataFrame은 관계형 데이터
  - 한마디로: 관계형 데이터셋: RDD + Relation, RDD의 확장판
      - 지연 실행 (Lazy Execution)
      - 분산 저장
      - Immutable
      - 열(Row) 객체가 있다
      - SQL 쿼리를 실행할 수 있다.
      - 스키마를 가질 수 있고 이를 통해 성능을 더욱 최적화 할 수 있다.
      - CSV, JSON, Hive 등으로 읽어오거나 변환이 가능하다.
  - RDD가 함수형 API를 가졌다면 DataFrame은 선언형 API
  - 자동으로 최적화가 가능
  - 타입이 없다.


### Two Engines of Spark
- 스파크는 쿼리를 돌리기 위해 두가지 엔진을 사용한다.

- Catalyst: Logical Plan을 Physical Plan으로 바꾸는 일.

  - Logical Plan은 수행해야하는 모든 transformation단계에 대한 추상화로 데이터가 어떻게 변해야 하는지 정의하지만 실제 어디서 어떻게 동작하는지는 정의하지 않는다.
  - Physical Plan은 Logical Plan이 어떻게 클러스터 위에서 실행될지 정의하며 실행 전략을 만들고 Cost Model에 따라 최적화한다.
  - Logical Plan → Physical Plan
      1. 분석: DataFrame 객체의 relation을 계산, 칼럼의 타입과 이름 확인
      2. Logical Plan 최적화
          1. 상수로 표현된 표현식을 Compile Time에 계산
          2. Predicate Pushdown: join&filter → filter&join
          3. Projection Pruning: 연산에 필요한 칼럼만 가져오기
      3. Physical Plan 만들기: Spark에서 실행 가능한 Plan으로 변환
      4. 코드 제너레이션: 최적화된 Physical Plan을 Java Bytecode로
- Tungsten: 스파크 엔진의 성능 향상이 목적
  - 메모리 관리 최적화
  - 캐시 활용 연산
  - 코드 생성


- ```spark.sql(query).explain(True)```: True가 없을 경우 Physical plan만 출력
  - Parsed Logical Plan
  - Analyzed Logical Plan
  - Optimized Logical Plan
  - Physical Plan
  
- ```spark.udf.register(alias, func_name, type)```: udf를 spark sql에서 사용할 수 있도록 함수를 등록

---
## Chapter 04. Spark ML과 머신러닝 엔지니어링

- MLlib: Machine Learning Library
  - ML을 쉽고 확장성 있게 적용하기 위해
  - 머신러닝 파이프라인 개발을 쉽게 하기 위해


- 머신러닝 파이프라인 구성: 데이터 로딩 → 전처리 → 학습 → 모델 평가

- MLlib으로 할 수 있는 것들
  - feature engineering
  - 통계적 연산
  - ML 알고리즘: Regression, SVM, Naive Bayes, Decision Tree, K-Means clustering
  - 추천 (ALS)


- DataFrame을 쓰는 MLlib API를 Spark ML이라고도 부름


- MLlib의 주요 Components
  - Transformer:
      - feature 변환과 학습된 모델을 추상화
      - 모든 Transformer는 transform() 함수를 가지고 있다.
      - 데이터를 학습이 가능한 포맷으로 바꾼다.
      - DF를 받아 새로운 DF를 만드는데, 보통 이상의 Column을 더하게 된다.
      - ex) Data Normalization, Tokenization, One-hot encoding
  - Estimator
      - 모델의 학습 과정을 추상화
      - 모든 Estimator는 fit() 함수를 가지고 있다.
      - fit()은 DataFrame을 받아 Model을 반환
      - 모델은 하나의 Transformer
      - ex) LinearRegression()
  - Evaluator
      - metric을 기반으로 모델의 성능을 평가
      - 모델을 여러개 만들어서, 성능을 평가 후 가장 좋은 모델을 뽑는 방식으로 모델 튜닝을 자동화 할 수 있다.
      - ex) BinaryClassificationEvaluator, CrossValidator