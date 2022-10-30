# KCC 2022: 멀티 도메인 대화 데이터 셋을 사용한 문서 검색 모델
[paper]()
### Conference: 
2022 한국컴퓨터종합학술대회 (KCC 2022.06.29~2022.07.01) [Link](https://www.kiise.or.kr/conference/kcc/2022/)
### Abstraction: 
오픈 도메인 질의 응답 모델에서 DPR(Dense Phrase Retrieval)은 두 개의 인코더를 사용해서 질문과 문단 간의 정확도 높은 문단을 검색하는 모델이다. 기존 DPR은 위키피디아 도메인
한 개를 사용해 학습을 진행했지만 본 연구에서는 네 개의 도메인과 대화 형식의 질의 응답으로 이루어진 데이터 셋을 사용해서 DPR 학습을 진행하고 기존 모델과 검색 성능을 비교하는 실험을 진행했다. 그 결과 기존 모델보다 검색 성능이 정확도 성능 지표 면에서 약 10~20% 정도 향상되는 결과가 나왔고, 이전 대화 내용이 포함된 질문을 제시했을 때 모델이 답변과 관련된 문단을 잘 검색하는 결과가 나왔다.

# Related Work
## 1. Open Domain Question Answering
방대한 정보들을 포함하고 있는 문서 집합들을 참조해 질문에 대한 답변을 생성하는 Task 

전통적인 접근 방법: Retriever-Reader Framework
- Retriever: 질문과 관련 있을 법한 passage를 찾아옴 (ex: TF-IDF, BM25)
- Reader: 주어진 문제에 대해 구체적인 답변을 찾아냄 (ex: Neural Network) 

Open domain question answering 에서는 후보 passages 를 고르는 Passage Retriever이 중요하다. 

## 2. Dense Phrase Retriever (DPR)
Retriever 부분을 기존에 사용하던 Sparse vector model(TF-IDF, BM25) 보다 dense vector representation을 사 용해 질문과 관련된 document passage 후보들을 결정하는 모델
Dual encoder architecture: 
1. question 과 passage 를 각각의 서로 다른 인코더에 통과시켜 d 차원으로 임배딩 
2. question 과 passage 벡터 간의 유사도를 Maximum Inner Product Search Algorithm (MIPS) 사용

추가적인 pretraining 없이 question, passage 쌍으로만 dense embedding model 학습 가능
![dpr image](https://github.com/BaeSuyoung/KCC2022/blob/main/image/dpr.png)

## 3. MultiDoc2Dial Dataset
- [dataset](https://doc2dial.github.io/multidoc2dial/)
- [paper](https://arxiv.org/abs/2109.12595)
- [github](https://github.com/IBM/multidoc2dial)
![enter image description here](https://github.com/BaeSuyoung/KCC2022/blob/main/image/multidoc2dial.png)


# 실험 방법
## 1. 아이디어
• Dense Phrase Retrieval 한계점 
1. Retrieval 학습 시 위키피디아 문서 도메인 하나만을 사용해서 학습 
	-> 실제 인터넷 상에 오픈 도메인은 수많은 도메인 존재 
	-> 여러 도메인을 반영해 학습을 해보자! 
2. 한 개 질문에 대한 답변을 생성하는 것을 목적으로 하는 모델 
	-> 대화 형식에서 문맥을 파악해 답변을 생성하는 능력 학습 부족 
	-> 대화 형식 데이터셋을 사용해보자! 

• Model : Dense Phrase Retrieval Dual encoder 구조, BERT 사전 학습 모델을 인코더 모델로 사용 
• Dataset : Multidoc2dial Dataset 
-> DPR question, passages 입력 형식으로 전처리 진행 후 입력으로 넣음


## 2. 실험
1) Domain Dataset 
• ‘\n’, ‘\t’, ‘\r’ 등 불필요한 부분 제거 
• 각 텍스트를 100 words 로 잘라서 passage 로 설정 
• 각 passage 가 포함된 문서 제목, 도메인 정보, 인덱스 함께 저장 
• 4282개 passages 구성 

2) Dialogue Dataset 
• MultiDoc2Dial 질문과 답변을 저장 
• 질문에 대해 참조하는 passage 에 대해 긍정 문단, 부정 문단을 만들어 저장함 (question, passage 쌍 생성) 
• 대화 기록 유무에 따른 성능 비교 평가를 위해 질문에 이전 대화 기록을 [sep] 토큰으로 구분해 함께 저장한 테스트셋도 별도로 만들어 저장

3) **최종 실험 모델: epoch 48 dpr fine-tuned model**

## 3. Task
1. 기존 DPR 모델과 멀티 도메인 데이터셋으로 학습한 검색 모델의 성능 비교 
• 정확도(Accuracy)
 • Top-k 에서 k 값 1, 20, 60, 100 으로 변경하면서 비교 

2. 멀티 도메인 데이터셋으로 학습한 모델을 사용해 질문에 대한 이전 대화 기록을 포함했는지 여부에 따라 정확 도가 얼마나 차이가 나는지 비교 
• 정확도(Accuracy) 
• Top-k 에서 k 값 1, 20, 60, 100 으로 변경하면서 비교

# 실험 결과
1. 기존 DPR 모델과 멀티 도메인 데이터셋으로 학습한 검색 모델의 성능 비교
- single

|K	|1 	|20	|60	|100	|
|----|----|----|----|----|
|Test	|52	|81	|	|87	|
- multi

|K	|1 	|20	|60	|100	|
|----|----|----|----|----|
|Test	|52	|81	|94	|96	|


2. 멀티 도메인 데이터셋으로 학습한 모델을 사용해 질문에 대한 이전 대화 기록을 포함했는지 여부에 따라 정확 도가 얼마나 차이가 나는지 비교 

|K	|1	|	|	|20	|	|	|60	|	|	|100	|	|	|
|----|----|----|----|----|----|----|----|----|----|----|----|----|
|	|Train|Dev|Test|Train|Dev|Test|Train|Dev|Test|Train|Dev|Test|
|o	|72	 |33	| 0	| 82	| 60	| 81	 |83	| 69	| 94 |83| 73| 96|
|x|40| 20| 0| 57| 39| 56| 62| 48| 59| 64| 52| 59|


# Conclusion
1. Retriever 모델 학습 시 2개 이상 도메인을 사용해 학습을 했을 때 검색 성능이 더 높아졌다는 것을 검증했다.
2. 질문 형식에 이전 대화 기록을 포함했을 때 질문과 관련된 문서 검색을 더 잘 한다는 결과가 나왔다.
