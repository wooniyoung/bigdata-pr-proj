< OTT 소비 트렌드와 극장 관객 수 상관관계 분석을 위함 람다 아키텍처 기반 빅데이터 파이프라인 구축 >
1. Introduction
넷플릭스, 웨이브, 티빙 등 OTT 서비스의 급성장과 코로나를 거치며 한국 극장 산업은 심각한 구조적 변화를 맞이했다. 최근 들어 극장가가 일부 회복세를 보이고 있으나, 대중의 콘텐츠 소비 관점이 OTT 중심으로 이동하면서 여전히 과거와는 다른 양상을 보인다. 본 프로젝트는 이러한 미디어 패러다임의 변화를 데이터 기반으로 증명하고자 한다. 대용량 과거 데이터를 Batch 분석하여 장기적 트렌드와 상관관계를 도출하는 동시에, 신작 개봉 및 공개 시점의 대중 반응을 실시간으로 포착하는 Lambda Architecutre 설계를 목표로 한다. Hadoop의 다양한 분산 처리 도구들을 유기적으로 연동하여 안정적이고 확장성 있는 실무형 빅데이터 파이프라인을 구축하였다.

2. Problem Definition
ㄴ OTT 흥행과 극장 수요의 관계 검증
OTT 주간 TOP 10에 오른 한국 영화 및 드라마의 시청 시간이 급증한 시기와 극장 관객 수의 감소 시기가 시계열상으로 일치하는가?

ㄴ 동시/단기 출시 영화의 박스오피스 분석
극장 개봉작이 OTT에 동시 혹은 한 달 이내 단기 출시된 경우, 극장 독점 개봉작과 비교하여 박스오피스 관객 수 하락에 얼마나 큰 차이가 발생하는가?

ㄴ 장르별 미디어 전환 분석
스펙터클이 중요한 액션/SF 장르와 스토리 중심의 드라마/로맨스 장르 간에 OTT 전환에 따른 극장 관객 감소 영향력이 다르게 나타나는가?

ㄴ 펜데믹 시기별 대중 소비 행동 변화 분석
코로나19 이전(2018 ~ 2019), 팬데믹 기간(2020 ~ 2022), 엔데믹 이후(2023 ~) 시기별로 OTT 화제성과 극장 관객 수 간의 상관계가 어떻게 변화했는가?

ㄴ 머신러닝 기반 박스오피스 예측
영화의 장르, 상영관 수, 주연 배우 인지도, 그리고 OTT 동시 개봉 여부를 기반으로 개봉 2주 차 이후의 박스오피스 하락률을 예측할 수 있는가?

ㄴ 실시간 반응 기반 화제성 지수 측정
신작 개봉 및 OTT 공개 직후, 실시간 대중 리뷰와 언급량을 기반으로 초기 화제성 지수가 극장 예매율 및 OTT 순위에 미치는 영향을 실시간으로 추적할 수 있는가?

3. Tech Stack
Infrastructure: Google Cloud Platform (GCP Compute Engine)
Platform: Hortonworks Data Platform (HDP 3.0.1 Sandbox via Docker)
Storage: HDFS, RDBMS (MySQL)
Data Ingestion: Apache Sqoop, Apache Flume, Apache Kafka
Data Processing (Batch): Apache Pig, Apache Spark (PySpark DataFrame/SQL)
Data Processing (Streaming): Apache Spark Structured Streaming / Apache Flink
Data Warehouse: Apache Hive
Machine Learning: Apache Spark MLlib
Visualization: Apache Zeppelin, Python (Matplotlib, Seaborn)
Version Control: Git / GitHub

4. Data Pipeline
ㄴ 데이터 수집 및 이관 (Data Ingestion)
정형 데이터 (Sqoop 파이프라인) - Python requests 라이브러리로 KOBIS(일별 박스오피스) 및 KMDB(영화 상세 메타데이터) API를 호출하여 정형 데이터를 수집한 후, 레거시 환경을 모사한 로컬 MySQL에 1차 적재한다. 이후 Apache Sqoop을 통해 MySQL의 데이터를 분산 저장소인 HDFS로 안전하고 빠르게 일괄 이관한다.

파일 데이터 - Netflix 공식 웹사이트에서 제공하는 글로벌 및 한국 주간 TOP 10 CSV 파일은 하둡 명령어(hdfs dfs -put)를 통해 HDFS 원시 데이터 공간(Raw Zone)에 직접 업로드한다.

실시간 로그 데이터 (Flume & Kafka 파이프라인) - 영화 커뮤니티 평점 및 SNS 실시간 언급량을 모사한 가상의 스트리밍 로그(버즈 데이터)를 리눅스 파일 시스템에 발생시킵니다. Apache Flume이 해당 로그 파일의 변경 사항을 실시간 감지(Spooling Directory)하여 수집하고, 분산 메시징 시스템인 Apache Kafka의 특정 토픽(Topic)으로 이벤트를 실시간 발행(Publish)한다.

ㄴ 데이터 정제 및 초기 ETL (Data Cleansing)
Apache Pig 환경 - HDFS에 로드된 결측치가 많고 스키마가 불명확한 CSV 및 텍스트 데이터를 처리하기 위해 데이터 흐름 언어인 Pig Latin 스크립트를 가동한다. FILTER, FOREACH 연산자를 활용하여 불필요한 컬럼 제거, 한국 영화 데이터 필터링, 날짜 포맷 통일 등 1차 정제 및 변환(ETL) 과정을 수행한다.

ㄴ 핵심 데이터 병합 및 집계 (Batch Processing)
Apache Spark (DataFrame API) - 대규모 분산 메모리 처리 엔진인 Spark를 활용하여 정제된 박스오피스 데이터와 넷플릭스 주간 데이터를 결합한다. 영화 제목 형태소 매칭 및 날짜 윈도우 조인(Join) 기법을 통해 특정 콘텐츠의 극장 상영기-OTT 공개기 데이터를 타임라인 기반으로 정밀 매칭하고 대규모 집계연산을 수행한다.

ㄴ 예측 모델링 (Advanced Analytics)
Apache Spark MLlib - 단순 통계 분석을 넘어, Spark 내장 머신러닝 라이브러리를 사용한다. 영화 장르, 상영 스크린 수, 팬데믹 시기 구분, OTT 단기 출시 여부 등을 피처(Feature)로 가공하여, 향후 개봉할 영화의 '박스오피스 관객 수 낙폭'을 예측하는 다중 선형 회귀(Multiple Linear Regression) 모델을 학습시키고 평가한다.

ㄴ 실시간 스트리밍 분석 (Speed Layer)
Spark Structured Streaming / Flink - Kafka 토픽으로 끊임없이 흘러 들어오는 실시간 버즈 데이터를 스트리밍 엔진이 마이크로 배치(Micro-batch) 단위로 파싱한다. "현재 시간 기준 OTT 화제성 탑 10 영화의 실시간 긍/부정 반응 빈도" 등을 인메모리 상에서 실시간으로 계산하고 갱신한다.

ㄴ 데이터 웨어하우스 적재 (Data Warehouse)
Apache Hive - Spark 및 전처리 레이어에서 가공이 완료된 최종 데이터 셋 구조에 맞추어 Hive 테이블을 설계한다. SQL 쿼리를 활용해 언제든 통계 지표를 뽑아낼 수 있도록 스키마를 부여(Schema-on-Read)하고, 분석용 데이터 웨어하우스를 완성한다.

ㄴ 대시보드 시각화 (Serving Layer)
Apache Zeppelin - 하둡 및 스파크 생태계와 완벽히 연동되는 대시보드 노트북인 Zeppelin을 웹 인터페이스(포트 9995)로 구성한다.
%hive 인터프리터를 통해 Hive DW에 SQL 질의를 던져 장르별 비교 차트, 코로나 전/중/후 구간 비교 그래프를 시각화한다.
%spark 인터프리터를 사용하여 실시간 스트리밍 결과 데이터와 머신러닝 예측 모델의 예측치 서빙 결과를 하나의 동적 대시보드 리포트 화면으로 표출한다.
