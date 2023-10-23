# Cyber Security Project (Payload Analysis)


## 🏆 Project Introduction
 1. 대회: 사이버보안 AI/빅데이터 챌린지 2023 [A트랙] <https://aibigdatasec.kr/detail>
 2. 주제: payload 분석을 통한 네트워크 웹 공격 분류
 3. 최종결과: 0.89338 (22등 / 71팀)
 4. 데이터: A_Track_trainset.csv ('label_action' category → 총 9개)

    
    |Log_Number|payload|label_action|
    |------|---|---|
    |0|GET/forum1_professionnel.asp?n ...|System_Cmd_Execution|
    |1|POST/owa/auth/logon.aspx?replace ...|System_Cmd_Execution|
    |2|GET/goods/goods_search?display ...|SQL_Injection|
    |...|...|...|



## 📖 Research and Analysis

최근 소프트웨어 기술의 발전으로 Cyber Security에 대한 필요성이 증대되고 있다. 랜섬웨어의 공격, 공급망 공격, 인공지능을 이용한 공격 등 소프트웨어애 대한 해킹 시도 방법이 다각화됨에 따라 여러 공격 시도에 대처하는 AI기술의 필요성 또한 커지고있다. 따라서 이번 대회에서는 네트워크 웹 공격을 정확도 높게 분류하는 AI모델을 생성하고자 하였다.
웹 방화벽 로그를 분석하고 비정상 행위로 분류하는 모델을 생성할 때 가장 중요한 것은 무작위의 Payload data에서 공격 유형에 맞는 적절한 feature를 발견하는 것이다. 기존의 자연어 전처리 방식에서 사용하는 방법만으로는 규칙을 발견할 수 없었기 때문에 'BPE Tokenize' 방법을 사용하여 payload data의 feature를 추출하고 이를 공격유형(label_action)에 할당해서 정확도를 예측하는 모델을 고안하였다.


#### BPE Tokenizer
  - payload data는 HTTP 요청을 보낼 때 전달되는 데이터이다. 데이터를 전송할 때는 header와 metadata 등의 다양한 요소를 함께 전송하는데, 이를 제외한 전송하고자 하는 데이터 자체를 의미하는 것이 payload 데이터이다.
  - EDA를 통해서 raw data(ex. POST /member/login HTTP/1.1\r\nContent-Length ..)를 공격 유형 별로 나눠 특징을 미리 추출하여 그 기준으로 분류하는 방식 등을 시도했으나, 공격 유형별 패턴이 고정되어 있지 않아 적절한 feature를 선택하기 어렵다고 판단하였다.
  - payload sentence 자체에 laebl_action 별로 특수문자, /기호, 도메인, SQL쿼리 정보 유무 등이 나타날 것이라는 가정 하에, 빈도 기준으로 높은 빈도의 토큰을 합쳐나가는 Byte Pair Encoding(BPE) 기법을 선택하였다.
  - BPE 기법은 모르는 단어가 등장했을 때 해당 토큰을 UNK(Unknown Token)라고 표현하여 무의미한 정보로 처리하는 문제, 즉 OOV(Out-Of-Vocabulary)문제를 해결하기에 적합한 tokenizer이다.
  - 의미가 없다고 생각되는 특수문자와 단어들의 집합인 payload data 에서 공격 유형(label_action) 별로 자주 사용되는 문자들을 기준으로 분류하였다.
  - google sentencepiece 패키지에서 model_type을 bpe로 선택하여 구현했으며, sentencepiece는 사전 토큰화 작업없이 단어 분리 토큰화를 수행하므로 언어에 종속되지 않기 때문에 payload data 에도 적절하다. 



## 📝 Data Pre-processing
####  1. BPE Tokenizer


####  2. Vocab Dictionary 생성


####  3. CountVectorizer




## 🏆 Modeling


## 📜 Result & Discussion
