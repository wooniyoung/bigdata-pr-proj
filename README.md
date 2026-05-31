< Lambda Architecture 기반 OTT-극장 소비 트렌드 빅데이터 분석 파이프라인 >

Introduction
넷플릭스, 웨이브 등 OTT 서비스의 급성장과 팬데믹을 거치며 한국 극장 산업은 심각한 구조적 변화를 맞이했습니다. 본 프로젝트는 코로나 거품이 완전히 꺼진 **엔데믹 뉴노멀(2024~2026) 시대**를 기점으로, 대중의 콘텐츠 소비 중심이 어떻게 이동했는지 데이터 기반으로 증명합니다. 

대용량 과거 데이터를 Batch 분석하여 장기적 트렌드와 상관관계를 도출하는 동시에, 신작 개봉 시점의 대중 반응을 실시간으로 포착하는 **람다 아키텍처(Lambda Architecture)**를 설계했습니다. 불필요한 레거시 도구를 배제하고, Spark 중심의 단일화된 엔진을 사용하여 빠르고 확장성 있는 실무형 빅데이터 파이프라인을 구축하였습니다.

Problem Definition
1. OTT 흥행과 극장 수요의 갉아먹기(Cannibalization) 검증 - OTT 주간 TOP 10 콘텐츠 시청 시간 급증 시기와 극장 관객 수 감소 시기가 시계열상 일치하는가?
2. 동시/단기 출시작 박스오피스 타격도 분석 - 극장 개봉 후 1달 내 OTT에 단기 출시된 영화의 박스오피스 하락률은 독점 개봉작과 얼마나 차이나는가?
3. 장르별 미디어 전환 양상 - 스펙터클(액션/SF) 장르와 스토리 중심(드라마/로맨스) 장르 간 OTT 전환 속도 및 극장 관객 방어력의 차이는?
4. [ML] 박스오피스 낙폭 예측 - 영화 장르, 스크린 수, 배우 인지도, OTT 출시 여부를 바탕으로 개봉 2주 차 박스오피스 하락률을 예측할 수 있는가?
5. [Streaming] 실시간 화제성 지수 추적 - 신작 공개 직후 SNS/커뮤니티의 실시간 긍정/부정 버즈량이 초기 극장 예매율에 미치는 영향을 실시간 추적할 수 있는가?

Tech Stack
Infrastructure - Google Cloud Platform (GCP Compute Engine)
Platform - Hortonworks Data Platform (HDP 3.0.1 Sandbox via Docker)
Storage - HDFS, RDBMS (MySQL)
Data Ingestion - Apache Sqoop, Apache Kafka (Direct Python Producer)
Data Processing (Batch & Streaming) - Apache Spark (DataFrame/SQL, Structured Streaming)
Data Warehouse - Apache Hive
Machine Learning - Apache Spark MLlib
Visualization - Apache Zeppelin, Python (Matplotlib, Seaborn)

Data Pipeline Architecture
1. Data Ingestion (수집 및 이관)
   Batch: Python으로 KOBIS API(최신 박스오피스)를 호출해 로컬 MySQL에 1차 적재 후, `Apache Sqoop`을 통해 HDFS(Raw Zone)로 일괄 분산 이관합니다. Netflix 데이터는 HDFS CLI로 직접 적재합니다.
   Streaming - 영화에 대한 가상 실시간 리뷰/버즈 로그를 Python 생성기가 `Apache Kafka` 토픽으로 직접 실시간 발행(Publish)합니다.
2. Batch Processing (배치 병합 및 집계)
   Apache Spark를 활용해 HDFS의 박스오피스 및 OTT 데이터를 정제(결측치, 타입 변환)하고, 타임라인 기반 조인(Window Join) 기법을 통해 대규모 집계 연산을 수행합니다.
3. Speed Layer (실시간 스트리밍 분석)
   Kafka로 유입되는 실시간 버즈 데이터를 `Spark Structured Streaming`이 마이크로 배치 단위로 파싱하여 인메모리 상에서 실시간 화제성 지수를 갱신합니다.
4. Data Warehouse (데이터 적재)
   가공이 완료된 최종 데이터 셋은 분석 편의성을 위해 `Apache Hive` 테이블로 스키마를 부여(Schema-on-Read)하여 적재합니다.
5. Serving Layer & ML (시각화 및 예측)
   Spark MLlib을 활용해 박스오피스 하락률 예측 회귀 모델을 학습합니다. 
   Apache Zeppelin 대시보드와 연동하여 Hive의 통계치와 실시간 스트리밍 결과를 하나의 동적 웹 리포트로 시각화합니다.
