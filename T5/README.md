# Text2SQL using KoGPT2
KoGPT2를 활용하여 자연어 질의(한국어 질문)를 SQL 쿼리로 변환하는 Text2SQL 모델 구축

## Dataset
자연어 기반 질의(NL2SQL) 검색 생성 데이터 (AI HUB)

- 본 코드에서는 Healthcare dataset만 사용하였음


<img src="https://github.com/user-attachments/assets/d12b0810-80bb-46b8-bbbb-40431123d353" alt="이미지 설명" width="350" height="300">

https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=&topMenu=&aihubDataSe=data&dataSetSn=71351

## Dataset 구성
Training/라벨링데이터/TEXT_NL2SQL_label_seouldata_healthcare.json
```
"data": [
	{
		"db_id": "seouldata_healthcare_455",
		"utterance_id": "Whr_0001",
		"hardness": "easy",
		"utterance_type": "BR03",
		"query": "SELECT ADMDNG_NM FROM SEOUL_PUBLIC_HYGIENE_BIZ WHERE UPSO_NM = '태창위생지'",
		"utterance": "태창위생지의 행정동 이름이 뭐야",
		"values": [
			{
				"token": "태창위생지",
				"start": 0,
				"column_index": 7
			}
		],
		"cols": [
			{
				"token": "행정동 이름",
				"start": 7,
				"column_index": 12
			}
		]
	}
```

Training/원천데이터/seouldata_healthcare_db_annotation.json
```
"data": [
	{
		"source": "서울특별시",
		"db_id": "seouldata_healthcare_455",
		"table_names_original": "SEOUL_PUBLIC_HYGIENE_BIZ",
		"table_names": "서울시 기타 위생용품 제조업 현황",
		"column_names_original": "[0, "CGG_CODE], [0, SNT_COB_CODE], ... ",
		"column_names": "[0, "시군구코드], [0, 업종코드], ... ",
		"column_types": "[0, "number], [0, number], ... ",
		"foreign_keys": [],
		"primary_keys": [],
			}
]
```

## 주요 전략 - Customize Vocabulary
단어 사전을 DB에 맞게 커스텀하여 SQL문 생성 성능을 높이고자 함
1. **Query**: 쿼리 토큰 띄어쓰기 단위로 분할하여 SQL 쿼리 내의 주요 연산자와 키워드를 인식할 수 있도록 
```
query_tokens = row['query'].split()
final_vocab.update([token.replace("'", "") for token in query_tokens])
```
2. **Column name**: column_names_original에서 컬럼 이름을 추출하여 단어사전에 추가하여 모델이 데이터베이스의 특정 컬럼 명을 이해하고, 질의 시 사용하도록 
```
column_names = [col[1] for col in row['column_names_original'] if isinstance(col[1], str)]
final_vocab.update(column_names)
```
3. **Table name**: table_names_original에서 _를 기준으로 테이블 이름을 분할하여 단어사전에 추가
```
for table_name in row['table_names_original']:
    final_vocab.update(table_name.split('_'))
```
4. **Question**: 한국어 질문을 서브워드 단위로 토크나이징하고, 토크나이저에 없는 서브워드만 단어사전에 추가
```
tokens = tokenizer.tokenize(row['utterance'])
final_vocab.update([token for token in tokens if token not in tokenizer.vocab])
```
5. **SQL 키워드 및 특수 토큰**: 주요 SQL 키워드를 추가하여 모델이 다양한 SQL 쿼리 구성을 학습하도록 함
```
sql_keywords = [keyword.upper() for keyword in CLAUSE_KEYWORDS + JOIN_KEYWORDS + WHERE_OPS + UNIT_OPS + AGG_OPS]
final_vocab.update([keyword for keyword in sql_keywords if keyword not in tokenizer.vocab])

special_tokens = {
    "pad_token": "<pad>",
    "bos_token": "<sos>",
    "eos_token": "<eos>",
    "unk_token": "<unk>",
    "sep_token": "<sep>"
}
```

결과:

기존 단어 수: 51204

새로운 단어 수: 9003

최종 토크나이저 길이: 60207


## Model
```
from transformers import BertTokenizer, BertModel
from transformers import GPT2LMHeadModel

tokenizer = AutoTokenizer.from_pretrained('skt/kogpt2-base-v2')
model = GPT2LMHeadModel.from_pretrained('skt/kogpt2-base-v2')
```


## RUN
```
GPT2.ipynb
```

