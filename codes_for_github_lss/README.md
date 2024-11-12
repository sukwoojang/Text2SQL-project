# text2sql

1. RAG_test1_* : 한국어 임베딩 모델에 따른 자연어 쿼리에 맞는 테이블 찾는 성능 비교

2. RAG_test2_* : DB Schema 정보로 벡터스토어를 만들떄 각 DB의 value값을 추가해서 만들었을 경우 성능 비교

3. SQL_value_sampler : sql파일들로 부터 랜덤으로 10줄씩 DB의 value 값 추출

4-1. text2sql_google_mt5_small_finetuning_test1 : google_mt5_small과 데이터를 이용해서 파인튜닝

4-2. text2sql_google_mt5_small_finetuning_test2 : 파인튜닝한 모델로 output 확인
