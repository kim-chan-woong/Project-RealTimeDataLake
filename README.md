# Real-Time-Data-Pipeline-DW-DM

# 실시간 데이터 수집 및 적재 (from gmarket)

# Requirements
1. 1시간 주기로 gmarket의 카테고리(12)별 베스트 100을 수집하고 유지할 수 있어야 한다.
2. 상품들의 정보(title, price, category, img)와 xpath(title_xpath, price_xpath, img_xpath)를 수집해야한다.   
3. 수집 시간에 따른 동적인 구조의 폴더로 저장되어 데이터의 가시성을 높인다.
4. 분석가 혹은 개발 인력들의 요구사항에 맞는 데이터를 언제, 어디서든 데이터 마트에 적재할 수 있는 구조여야 한다. (웨어하우징)
5. 과정 전체의 자동화와 모니터링 및 추적이 가능해야 한다.

# 주요 환경 및 스펙
virtual box - 6.1 (가상 서버 8대)   
centos7   
mobaXterm - 8.6 (원격 제어)   
hadoop - 3.1.0 (HA MODE, hdfs)   
zookeeper - 3.4.10 (모니터링, 작업 스케쥴링)   
hive - 3.1.2 (hdfs data warehousing)   
mysql - 5.7 (hive metastore)   
nifi - 1.9.2 (workflow tool)   
python - 3.8.5 (crawling code)   
postgre - 11 (data mart)    
anaconda3, juptyer notebook - python crawling code test   

# result
1. 2021년 03월 24일 19시 ~ 2021년 03월 25일 13시 까지의 19시간 수집 결과
2. 카테고리 12종류 x 각 베스트 100 = 1시간 주기 1,200 x 19시간  = 22,800건

# process

1. python 크롤링 코드 작성(anaconda3, jupyter notebook 테스트) 후 linux crontab 활용 1시간 주기 python script(.py) 실행   
2. 수집한 데이터를 HDFS에 분산 저장 (원본 데이터 유지, 수집 시간에 따른 동적 폴더 구조 / nifi 활용)   
3. hive 활용 HDFS 웨어 하우징(hiveQL문으로 원본 파일(.csv) 테이블화 및 조건 검색)   
4. 요구 사항 가정 및 hiveQL SELECT  결과를 DataMart(Postgre)에 적재 or 곧바로 csv파일로 추출
5. 위 과정들을 Apache Nifi(workflow tool)로 작업 자동화 및 실시간 추적, 모니터링   
![Screenshot_24](https://user-images.githubusercontent.com/66659846/112711394-5ba6c380-8f0b-11eb-9a3a-d92790bd22fa.png)

# servers
서버1(nn01): active namenode, zookeeper, journalnode, nifi and hive master 역할   
서버2(rm01): standby namenode, zookeeper, journalnode, ResourceManager 역할   
서버3(jn01): zookeeper, journalnode 역할   
서버4(dn01): NodeManager, Datanode 역할   
서버5(dn02): NodeManager, Datanode 역할   
서버6(dn03): NodeManager, Datanode 역할   
서버7(dbserver): Postgre, DataMart 역할   
서버8(getdataserver): Source Data 수집 역할   
![Screenshot_40](https://user-images.githubusercontent.com/66659846/112713011-281d6680-8f16-11eb-8be4-77e45eda58ab.png)   

# mobaXterm remote work
![Screenshot_25](https://user-images.githubusercontent.com/66659846/112711853-de7d4d80-8f0e-11eb-8673-d2fa4d8219bd.png)

# 1hour cycle crontab
1시간 주기 데이터 수집 (get_data.py, getdataserver)
1. 1시간 주기로 get_data.py가 실행 되어 카테고리별 베스트 100 상품들의 정보를 수집, csv파일 형태로 원본 데이터가 생성   
2. /root/data에 저장되면 곧바로 active namenode인 nn01에 ssh통신을 통해 배포 후 원본 데이터 삭제(rank, title, price, category, img, title_xapth, price_xpath, img_xpath)   
3. 원본 csv파일이 nn01에 전달되면 nifi를 통해 후 작업 
![Screenshot_20](https://user-images.githubusercontent.com/66659846/112711786-6020ab80-8f0e-11eb-9542-831890e6e512.png)
![Screenshot_19](https://user-images.githubusercontent.com/66659846/112711787-61ea6f00-8f0e-11eb-8e06-3a0170ae6496.png)
![Screenshot_28](https://user-images.githubusercontent.com/66659846/112712052-2fda0c80-8f10-11eb-9877-09f384625b12.png)

# nifi processots flow
1. GetFile: nn01의 지정할 경로에 파일 발생 시 nifi 상으로 get 후 경로의 로컬 데이터는 삭제   
2. PutHDFS: get한 데이터를 hdfs에 저장, now() format 활용 동적인 폴더 생성   
3. ReplaceText: hdfs를 hive 테이블화 하기 위한 hiveQL문 작성, csv 설정 및 수집 시간에 따른 테이블명 동적 생성   
4. PutHiveQL: 위 작성한 hiveQL문 실행   
5. SelectHiveQL: hive테이블을 요구 사항에 맞게 SELECT, 이 때 csv파일로 바로 추출도 가능   
6. ConvertRecord: HiveQL SELECT 결과(.csv)를 json 형식으로 변경   
7. ConverJSONToSQL: json 형식의 데이터를 데이터 마트(postgre)의 테이블에 적재하기 위한 설정   
8. PutSQL: 위 설정한 SQL문 실행   
9. LogAttribute: 과정 별 failure 혹은 전체 과정 success 모니터링
10. Queued: 각 과정별 진행 상황 확인 및 파일 추출(본 flow에선 .csv, .json 형태로 곧바로 추출 가능)
![Screenshot_37](https://user-images.githubusercontent.com/66659846/112712416-dcb58900-8f12-11eb-9f10-161534b1a606.png)   

# nifi controllers   
1. CSVReader: csv 파일 핸들링 설정   
2. DBCPConnectionPool: datamart(postgre) jdbc driver 및 DB 설정   
3. HiveConnectionPool: hive jdbc driver 및 DB 설정   
4. JsonRecordSetWriter: json 파일 핸들링 설정   
![Screenshot_38](https://user-images.githubusercontent.com/66659846/112712724-703b8980-8f14-11eb-8aa6-c57fe3174974.png)

# Save HDFS   
수집 시간에 따른 동적인 폴더 구조 구축   
hdfs folder:/user/source_data/YYYY/MM/DD/HH/gmarket.csv
![Screenshot_27](https://user-images.githubusercontent.com/66659846/112712090-6ca60380-8f10-11eb-8e23-6844c59ba256.png)
![Screenshot_29](https://user-images.githubusercontent.com/66659846/112712094-77f92f00-8f10-11eb-8ab3-e0f771d2e4dd.png)

# hive hdfs warehousing   
수집 시간에 따른 동적인 테이블 명 생성   
![Screenshot_22](https://user-images.githubusercontent.com/66659846/112712130-c4446f00-8f10-11eb-9dde-d99f5ba2a746.png)
![Screenshot_30](https://user-images.githubusercontent.com/66659846/112712160-02da2980-8f11-11eb-9c4e-33a28cad977d.png)
![Screenshot_31](https://user-images.githubusercontent.com/66659846/112712161-040b5680-8f11-11eb-88f1-f41b1b3d0ecc.png)
![Screenshot_32](https://user-images.githubusercontent.com/66659846/112712183-2c935080-8f11-11eb-8c92-c580617e2bf1.png)

# save datamart(postgre)
1. 2021년 03월 24일 19시, 2021년 03월 25일 13시의 카테고리별 베스트 100의 데이터 필요 요구 사항 가정   
 
요구 사항에 맞는 SELECT 쿼리문 설정(nifi flow no.05 SelectHiveQL)   
![Screenshot_39](https://user-images.githubusercontent.com/66659846/112712839-42a31000-8f15-11eb-86b9-b4627a601414.png)   

category_kind table: 새로 들어올 데이터의 카테고리 이상값 보완 목적
![Screenshot_33](https://user-images.githubusercontent.com/66659846/112712240-78de9080-8f11-11eb-968a-ec29aa860919.png)   

데이터 적재   
![Screenshot_34](https://user-images.githubusercontent.com/66659846/112712243-7a0fbd80-8f11-11eb-8f56-c7e0300507f7.png)

idx: primary key, auto increment   
rank: not null   
title: not null   
price: not null   
url: not null   
img: not null   
category: foreign key, category_kind(ca_name)   
title_xpath: not_null   
price_xpath: not_null   
img_xpath: not_null   
![Screenshot_35](https://user-images.githubusercontent.com/66659846/112712244-7a0fbd80-8f11-11eb-8990-206b8aa7305f.png)
![Screenshot_36](https://user-images.githubusercontent.com/66659846/112712245-7aa85400-8f11-11eb-8665-2477583a4faf.png)
