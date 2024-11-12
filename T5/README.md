# RAG and Text-To-SQL with T5
RAG 기반 DB 검색 시스템 구축
T5모델을 fine-tuning하여 자연어 질의(한국어) 입력을 SQL 구문으로 바꿔 주는 모델 개발 

## Dataset
자연어 기반 질의(NL2SQL) 검색 생성 데이터 (AI HUB)


<img src="https://github.com/user-attachments/assets/d12b0810-80bb-46b8-bbbb-40431123d353" alt="이미지 설명" width="350" height="300">

https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=&topMenu=&aihubDataSe=data&dataSetSn=71351

## Dataset 구성 예시
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

## 주요 전략 1 - RAG
효과적인 DB linking을 위해 RAG 활용
기존의 Sentence-transformer는 범용적인 텍스트 임베딩 모델을 Text-To-SQL task에 특화된 임베딩 모델로 fine-tuning
양성샘플, 음성샘플을 활용한 대조학습 진행

## 주요 전략 2 - 특수 토큰 추
테이블을 구분하는 T토큰, 컬럼을 구분하는 C토큰을 추가함으로써 각 요소의 경계를 명확히 인식하고, 특정 부분에 집중하는 효과를 기대

## 주요 전략 - Customize Vocabulary
단어 사전을 DB에 맞게 커스텀하여 SQL문 생성 성능을 높이고자 함
학습 데이터 기반으로 단어 빈도수 측정 후, 일정 이상의 단어들을 단어사전에 포함 시킴



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

