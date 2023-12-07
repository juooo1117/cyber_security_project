# Cyber Security Project (Payload Analysis & Classification)


## 🏆 Project Introduction
 1. 대회: 사이버보안 AI/빅데이터 챌린지 2023 [A트랙] <https://aibigdatasec.kr/detail>
 2. 주제: payload 분석을 통한 네트워크 웹 공격 분류
 3. 팀원: 박주현, 이신철, 김이정, 최형규
 4. **최종결과: 0.89338 (22등 / 71팀)**
 5. 데이터: A_Track_trainset.csv ('label_action' category → 총 9개)
 
    
    |Log_Number|payload|label_action|
    |------|---|---|
    |0|GET/forum1_professionnel.asp?n ...|System_Cmd_Execution|
    |1|POST/owa/auth/logon.aspx?replace ...|System_Cmd_Execution|
    |2|GET/goods/goods_search?display ...|SQL_Injection|
    |...|...|...|



## 📖 Research and Analysis

최근 소프트웨어 기술의 발전으로 Cyber Security에 대한 필요성이 증대되고 있다. 랜섬웨어의 공격, 공급망 공격, 인공지능을 이용한 공격 등 소프트웨어애 대한 해킹 시도 방법이 다각화됨에 따라 여러 공격 시도에 대처하는 AI기술의 필요성 또한 커지고있다. 따라서 이번 대회에서는 네트워크 웹 공격을 정확도 높게 분류하는 AI모델을 생성하고자 하였다.


웹 방화벽 로그를 분석하고 비정상 행위로 분류하는 모델을 생성할 때 가장 중요한 것은 무작위의 Payload data에서 공격 유형에 맞는 적절한 feature를 발견하는 것이다. 기존의 자연어 전처리 방식에서 사용하는 방법만으로는 규칙을 발견할 수 없었기 때문에 'BPE Tokenize' 방법을 사용하여 payload data의 feature를 추출하고 이를 공격유형(label_action)에 할당해서 정확도를 예측하는 모델을 고안하였다.


### BPE Tokenizer
  - payload data는 HTTP 요청을 보낼 때 전달되는 데이터이다. 데이터를 전송할 때는 header와 metadata 등의 다양한 요소를 함께 전송하는데, 이를 제외한 전송하고자 하는 데이터 자체를 의미하는 것이 payload 데이터이다.
  - EDA를 통해서 raw data(ex. POST /member/login HTTP/1.1\r\nContent-Length ..)를 공격 유형 별로 나눠 특징을 미리 추출하여 그 기준으로 분류하는 방식 등을 시도했으나, 공격 유형별 패턴이 고정되어 있지 않아 적절한 feature를 선택하기 어렵다고 판단하였다.
  - payload sentence 자체에 laebl_action 별로 특수문자, /기호, 도메인, SQL쿼리 정보 유무 등이 나타날 것이라는 가정 하에, 빈도 기준으로 높은 빈도의 토큰을 합쳐나가는 Byte Pair Encoding(BPE) 기법을 선택하였다.
  - BPE 기법은 모르는 단어가 등장했을 때 해당 토큰을 UNK(Unknown Token)라고 표현하여 무의미한 정보로 처리하는 문제, 즉 OOV(Out-Of-Vocabulary)문제를 해결하기에 적합한 tokenizer이다.
  - 의미가 없다고 생각되는 특수문자와 단어들의 집합인 payload data 에서 공격 유형(label_action) 별로 자주 사용되는 문자들을 기준으로 분류하였다.
  - google sentencepiece 패키지에서 model_type=bpe로 구현하였고, sentencepiece는 단어 분리 토큰화를 수행하므로 언어에 종속되지 않기 때문에 payload data 에도 적절하다. 



## 📝 Data Pre-processing

모델에 넣기 위해서 Data pre-processing은 아래의 3단계의 순서로 진행하였다.

<p align="center">
  <img src="https://github.com/juooo1117/cyber_security_project/assets/95035134/f9191047-e711-465b-873e-8ac5d7bdedb9">
</p>

###  1. BPE Tokenizer
   - google sentencepiece 패키지에서 model_type=bpe로 구현하였고, sentencepiece는 단어 분리 토큰화를 수행하므로 언어에 종속되지 않기 때문에 payload data 에도 적절하다. 
   - Payload데이터를 각 행마다 토큰화한 새로운 컬럼(payload_token)에 저장해 공격유형을 분류할 수 있는 Feature를 추출

###  2. Vocab Dictionary 생성
   - sentencepiece(BPE) tokenizer를 이용한 단어사전(vocab dictionary)생성
   - 'payload' column의 데이터들만 .txt 형태의 파일로 먼저 합치고 sentencepiece - BPE type을 활용하여 tokenize한다. 이 때 vocab dictionary에 담길 수 있는 최대 단어 수는 32,000개로 제한하였다.
   - 실제로 토큰화한 뒤에 vocab에 담긴 단어를 살펴보니 지정해 준 바와같이 32,000개의 단어가 생성된 것을 확인할 수 있었다.

###  3. CountVectorizer
   - 추출된 payload_token의 값들을 벡터화를 통해 임베딩 벡터로 변환 임베딩 벡터를 학습시켜 공격 유형을 분류
   - CountVectorizer로 'payload_token' column의 각 행별 값들을 공백으로 나누고 빈도수를 확인하여 벡터화 → 각 단어들은 빈도정보를 반영한 숫자 값을 가진다.
   - 토큰화과정에서 단어 사전의 생성 기준이 빈도였기 때문에, TF-IDF Vectorizer 보다는 단어의 빈도(frequency)로 벡터화하는 CountVectorizer를 사용하는 것이 적절하다고 판단했다. 



## 🏆 Modeling

### RandomforestClassfier
   - 각 행별 'payload_token' 값들의 label_action 분류를 위해서 랜덤포레스트(RandomForestClassifier)을 분류모델로 사용하였다.
   - DecisionTreeClassifier는 정확도가 낮아서 분류모델로 사용하지 않았고, boosting은 100번만 반복(n_estimators=100)하도록 해서 손실함수의 가중치를 100번만 조정하였다. boosting값이 클수록 모델의 정확도는 증가하지만 학습시간, overfitting의 가능성 또한 증가하므로 100 이상으로 늘리지 않았다.
   - 학습 후에는 cosstab(Confusion Matrix)으로 'label_action' category별 'pred'&'real'값을 교차 확인해서 피팅시킨 모델로 예측한 값이 얼마나 맞았는지를 확인해 보았다. (diagonal value: 맞춘 대상을 의미)

**[참고] Confusion Matrix**
<p align="center">
  <img src="https://github.com/juooo1117/cyber_security_project/assets/95035134/dcd7f2f4-1744-4db5-9240-b61b91256629" width="700" height="300">
</p>



## 📜 Result & Discussion
   - 주어진 train set(45000개)을 2배 늘려서(총 90000개), test data(0.7) & validation data(0.3) 비율로 나누어 학습했을 때 **validation data로 나온 model accuracy는 0.93**이었다.
   - 이 model에 실제 test data(대회문제)를 넣었을 때의 **accuracy(f1 score)는 0.89338** 이다.

