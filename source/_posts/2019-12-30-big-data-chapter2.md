---
layout: post
title:  "[빅데이터를 지탱하는 기술] 2장_빅데이터 탐색"
date:   2019-12-30
categories: Big Data
---

## 1. 크로스 집계

##### 크로스 집계

![](/image/bigdata_cross_table.png)

1. 크로스 테이블
   행과 열이 교차하는 부분에 숫자 데이터가 들어감

2. 트랜젝션 테이블
   행방향으로만 증가하고, 열방향으로는 데이터가 증가하지 않음

3. 크로스 집계
   트레젝션 테이블에서 크로스 테이블로 변환하는 과정.
   피봇 테이블을 이용해서 할 수 있다.

##### 룩업 테이블

트랜젝션 테이블의 '상품 ID' 를 사용해서 '상품명' 과 '상품 카테고리' 참고할 때 사용되는 것이, 룩업 테이블이다.
상품 정보를 하나의 테이블로 정리해두면 나중에 속성을 추가하거나 변경하는 것도 간단하다.

## 2. 열 지향 스토리지

##### 처리량과 지연 시간

1. 처리량 ( throughput )
   일정 시간에 처리할 수 있는 데이터의 양.
   배치 처리 등 대규모 데이터 처리에서 중요시.

2. 지연 ( latency )
   데이터 처리가 끝날 때 까지의 대기 시간.
   애드 혹 데이터 분석에서 중요시.

##### 데이터 처리의 지연

데이터 처리의 응답이 빠르면, '지연이 적다' 라고 한다. 

1. 메모리
   모든 데이터를 메모리에 올리는 것이다.

2. RDB
   만일, 한 레코드의 크기가 500 바이트라고 하면, 천만 레코드의 경우 5 GB 가 된다.
   이 정도면, MySQL 이나 PostgreSQL 같은 일반적인 RDB 가 데이터 마트로 적합하다.
   RDB는 원래 지연이 적고 많은 클라이언트가 접속해도 성능이 나빠지지 않으므로 많은 사용자가 사용하는 실제 운영 환경의 데이터 마트로 적합하다.
   하지만, RDB 는 메모리가 부족하면 급격히 성능이 저하된다.

##### '압축', '분산' 에 의해 지연 줄이기

데이터를 가능한 작게 '압축' 하고 여러 디스크에 '분산' 함으로써 데이터 로드에 따른 지연을 줄일 수 있다.
분산된 데이터를 읽기 위해서는 멀티 코어를 활용해 디스크 I/O 를 병렬 처리하는 것이 효과적이다.
이런 아키텍쳐를 MPP (Massive Parallel Processing) 라고 한다.
대량 데이터를 분석하기 위해 데이터베이스에 널리 사용되고 있다.
ex ) Amazon Redshift, Google BigQuery

##### 행 지향, 열 지향 데이터베이스 

1. 행 지향 데이터베이스
   레코드 단위로 읽고 쓰기에 최적화.
   테이블의 각 행을 하나의 덩어리로 디스크에 저장.
   새 레코드 추가할 때마다 파일의 끝에 데이터를 쓸 뿐이여서 빠르게 추가 가능.
   데이터 검색을 고속화하기 위해 인덱스 사용.
   데이터 분석에서는 어떤 칼럼 사용될지 미리 알 수 없으므로, 인덱스를 작성해도 도움이 되지 않음.
   ex ) Oracble, MySQL

2. 열 지향 데이터베이스
   칼럼 단위의 집계에 최적화.
   데이터를 미리 칼럼 단위로 정리해서 필요한 칼럼만을 로드하여 디스크 I/O 를 줄임.
   같은 칼럼에는 종종 유사한 데이터가 나열되어서 데이터 압축 효율도 우수.
   ex ) Teradata, Amazon Redshift

##### MPP 데이터베이스

1. 행 지향 데이터베이스
   하나의 쿼리는 하나의 스레드에서 실행.
   하나의 쿼리가 충분히 짧은 시간에 끝나는 것으로 가정하므로 하나의 쿼리를 분산 처리하는 상황은 가정하지 않음.

2. 열 지향 데이터베이스
   디스크에서 대량의 데이터를 읽기 때문에 한 번의 쿼리 실행 시간이 길다.
   멀티 코어를 활용해서 고속화 하는것이 좋음.

MPP 에서는 하나의 쿼리를 다수의 태스크로 분해하고, 이를 가능한 한 병렬로 수행한다.
예를 들어,

1. 1억 래코드로 이루어진 테이블의 합계를 계산하기 위해
2. 10만 레코드로 구분하여 1000 개의 테스크로 나눈다.
3. 각 테스크는 각각 독립적으로 10 만 레코드의 합계를 집계해서 
4. 마지막 모든 결과를 모아 총 합계를 계산한다.

MPP 를 사용한 데이터 집계는 CPU 코어수에 비례해서 고속화된다.
단, 디스크로부터의 로드가 병목 현상이 발생하지 않도록 데이터가 고르게 분산되어 있어야한다. 

## 3. 애드 혹 분석과 시각화 도구

1. Jupyter Notebook
2. Redash
3. Superset
4. Kibana

## 4. 데이터 마트의 기본 구조

##### 다차원 모델과 OLAP 큐브

1. RDB
   표 형식으로 모델링된 데이터를 SQL 로 집계한다.

2. OLAP
   다차원 모델의 데이터 구조를, MDX (Multidimensional Exrepssions) 등의 쿼리 언어로 집계한다.

데이터 분석을 위해 만들어진 다차원의 데이터를 OLAP 큐브라고 한다. OLAP 큐브를 크로스 집계하는 구조가 OALP 이다.
컴퓨터 성능이 높지 않을 때는, 크로스 집계의 모든 조합을 데이터 베이스 안에 캐쉬해두고, 쿼리가 실행되면 이미 집계된 결과를 반환하는 구조였다.
최근에는 MPP 데이터베이스와 인 메모리 데이터베이스로, 사전에 계산해 둘 필요가 없다. MPP 데이터베이스에 다차원 모델의 개념이 없기 때문에, 이를 대신해 '비정규화 테이블' 을 준비한다. 비정규화 테이블을 BI 도구에서 열어서 기존의 OLAP 과 동등한 시각화를 실현할 수 있다.

##### 트렌젝션, 마스터 테이블

![](/image/bigdata_rdb_table.png)

1. 트랜젝션 테이블
   시간과 함께 생성되는 데이터들을 기록

2. 마스터 테이블
   트랜젝션에서 참고되는 각종 정보를 기록

##### 팩트, 디맨젼 테이블

1. 팩트 테이블
   트랜젝션 테이블 처럼, 사실이 기록된 것.
   집계의 기반이 되는 숫자 데이터 등.

2. 디멘젼 테이블
   마스터 테이블 처럼, 참고되는 마스터 데이터.
   데이터를 분류하기 위한 속성 값.

##### 스타 스키마

![](/image/bigdata_star_scheme.png)

데이터 마트를 만들때는, 팩트 테이블을 중심으로 여러 디멘젼 테이블을 결합하는 것이 좋다.
스타 스키마를 사용하는 이유는,

1. 단순하고 이해하기 쉬워 데이터 분석이 쉽다.
   
2. 성능 상의 이유
   데이터 양이 증가하면 팩트 테이블은 디멘젼 테이블보다 훨씬 커져, 팩트 테이브이 메모리 용량을 초과한 시점에 디스크 I/O 가 발생하고 그 대기 시간으로 쿼리의 지연으로 이어진다.
   그래서, 팩트 테이블에는 id 와 같은 키만을 남겨두고 그 이외의 나머지는 디멘젼 테이블로 옮긴다.

##### 비정규화 테이블

스타 스키마는 과거의 이야기이다. 성능 상의 문제는 열 지향 스토리지에 의해 해결된다.
MPP 데이터베이스와 같은 열 지향 스토리지의 보급으로, 칼럼의 수가 아무리 늘어나도 성능에 영향이 없다.
그래서, 처음부터 팩트 테이블에 모든 칼럼을 포함하고 쿼리의 실행 시에는 테이블 결합을 하지 않는 것이 간단하다.
스타 스키마에 비정규화를 진행해 모든 테이블을 결합한 팩트 테이블을 비정규화 테이블이라고 한다.

##### 테이블 추상화

![](/image/bigdata_dimension.png)

비정규화 테이블을 준비했으면, 다차원 모델에 의해 추상화한다.
다차원 모델은 칼럼을 디멘젼과 측정값으로 분류한다. 

1. 디멘젼
   주로 날짜 및 문자열.
   크로스 집계의 행이나 열로 사용

2. 측정값
   주로 숫자값.

---

빅데이터를 지탱하는 기술 <니시다 케이스케>