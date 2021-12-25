# 가스공급량 수요예측 모델개발
한국가스공사 | 스타트업 | 정형데이터 | 수요예측

## 대회 개요
- 목적: 한국가스공사의 시간단위 공급량 데이터를 활용하여 90일 한도 일간 공급량을 예측하는 인공지능 모델 개발
- 주최: 한국가스공사
- 참가대상: 스타트업 및 예비창업가
- 평가지표: NMAE(Normalized Mean Absolute Error)
- [대회 홈페이지](https://dacon.io/competitions/official/235830/overview/description)


## 데이터
- 변수: 연월일, 시간, 공급사, 공급량
- 학습: 2013년 1월 1일 ~ 2018년 12월 31일
- 예측: 2019년 1월 1일 ~ 2019년 3월 31일
- [데이터 소스](https://dacon.io/competitions/official/235830/data)

## 외부데이터 검토
가스 공급량과 관련이 높은 가스 가격, 기상정보, 공휴일 관련 외부데이터의 모델 적용 시도:
| 데이터 | 출처 | 적용방법 |
|:-|:-|:-|
| 도시가스 월별 상대가격 지수 데이터 | 한국가스공사 | 민수용, 산업용 가스 상대가격 변수 추가 |
| 기상정보 데이터 | 기상자료개방포털 | 온도, 강수 등 월평균 값 추출후 3개월 Lagging 변수 추가 |
| 지역별 특수일 효과 | 한국가스공사 | 평일 대비 특수일 가스 수요 감소 비율 적용 |
| 공휴일 데이터 | 한국천문연구원 특일정보 API | 명절, 국경일 등 공휴일 변수 추가 |

대회 규정상 예측기간 이전(~'18.12.31)의 데이터만 사용가능함. Data Leakage를 피하기 위해 1)월평균 값 적용, 2)'18년 12월 값 적용, 3)예측기간의 외부 데이터 값 예측 등으로 외부데이터를 적용해 보았으나 성능이 개선되지 않았음. 또한 공급사가 위치한 지역을 알 수 없어 외부데이터를 활용하기가 더욱 어려웠음 
=> 최종적으로 외부데이터는 사용하지 않음


## Results on Competition Dataset
| Model | PubChem | PubChemQC | Competition LB (Public/Private) |
|:-|:-:|:-:|:-:|
| ELECTRA | 0.0493 | − | 0.1508/− |
| BERT Regression | 0.0074 | 0.0497 | 0.1227/− |
| MoT-Base (w/o PubChem) | − | 0.0188 | 0.0877/−|
| MoT-Base (PubChemQC 150k) | **0.0086** | 0.0151 | 0.0666/− |
| &nbsp;&nbsp;&nbsp;&nbsp;+ PubChemQC 300k | " | **0.0917** | 0.0526/− |
| &nbsp;&nbsp;&nbsp;&nbsp;+ 5Fold CV | " | " | 0.0507/− |
| &nbsp;&nbsp;&nbsp;&nbsp;+ Ensemble | " | " | 0.0503/− |
| &nbsp;&nbsp;&nbsp;&nbsp;+ Increase Maximum Atoms | " | " | **0.0497/0.04931** |

**Description**: Comparison results of various models. ELECTRA and BERT Regression are SMILES-based models which are trained with PubChem-100M (and PubChemQC-3M for BERT Regression only). ELECTRA is trained to distinguish fake SMILES tokens (i.e., ELECTRA approach) and BERT Regression is trained to predict the labels, without unsupervised learning. PubChemQC 150k and 300k denote that the model is trained for 150k and 300k steps in PubChemQC stage.

## Utilities
This repository provides some useful utility scripts.

* `create_dataset_index.py`: As mentioned above, it creates seeking positions of samples in the dataset for random accessing.
* `download_pubchem.py` and `download_pubchemqc.py`: Download PubChem3D and PubChemQC datasets.
* `find_test_compound_cids.py`: Find CIDs of the compounds in test dataset to prevent from training the compounds. It may occur data-leakage. 
* `simple_ensemble.py`: It performs simple ensemble by averaging all predictions from various models.

## License
This repository is released under the Apache License 2.0. License can be found in [LICENSE](LICENSE) file.
