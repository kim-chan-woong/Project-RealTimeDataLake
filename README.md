# Real-Time-Data-Pipeline-DW-DM

# 실시간 데이터 수집 및 적재 (from gmarket)

# Requirements
1. 1시간 주기로 gmarket의 카테고리(12)별 베스트 100을 수집하고 유지할 수 있어야 한다.   
2. 수집 시간에 따른 동적인 구조의 폴더로 저장되어 데이터의 가시성을 높인다.
3. 분석가 혹은 개발 인력들의 요구사항에 맞는 데이터를 언제, 어디서든 데이터 마트에 적재할 수 있는 구조여야 한다. (웨어하우징)
4. 과정 전체의 자동화와 모니터링 및 추적이 가능해야 한다.

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

# result
1. 2021년 03월 24일 19시 ~ 2021년 03월 25일 13시 까지의 19시간 수집 결과
2. 카테고리 12종류 x 각 베스트 100 = 1시간 주기 1,200 x 19시간  = 22,800건

# process
![Screenshot_24](https://user-images.githubusercontent.com/66659846/112711394-5ba6c380-8f0b-11eb-9a3a-d92790bd22fa.png)

1. python 크롤링 코드 작성(anaconda3, jupyter notebook 테스트) 후 linux crontab 활용 1시간 주기 python script(.py) 실행   
2. 수집한 데이터를 HDFS에 분산 저장 (원본 데이터 유지, 수집 시간에 따른 동적 폴더 구조 / nifi 활용)   
3. hive 활용 HDFS 웨어 하우징(hiveQL문으로 원본 파일(.csv) 테이블화 및 조건 검색)   
4. 요구 사항 가정 및 hiveQL SELECT  결과를 DataMart(Postgre)에 적재 or 곧바로 csv파일로 추출
5. 위 과정들을 Apache Nifi(workflow tool)로 작업 자동화 및 실시간 추적, 모니터링

# servers
![Screenshot_14](https://user-images.githubusercontent.com/66659846/112711815-9100e080-8f0e-11eb-93ee-40c3c809374f.png)
서버1(nn01): active namenode, zookeeper, journalnode, nifi and hive master 역할   
서버2(rm01): standby namenode, zookeeper, journalnode, ResourceManager 역할   
서버3(jn01): zookeeper, journalnode 역할   
서버4(dn01): NodeManager, Datanode 역할   
서버5(dn02): NodeManager, Datanode 역할   
서버6(dn03): NodeManager, Datanode 역할   
서버7(dbserver): Postgre, DataMart 역할   
서버8(getdataserver): Source Data 수집 역할 

# mobaXterm remote work
![Screenshot_25](https://user-images.githubusercontent.com/66659846/112711853-de7d4d80-8f0e-11eb-8673-d2fa4d8219bd.png)

# 1hour cycle crontab
1시간 주기 데이터 수집 (get_data.py, getdataserver)
1. 1시간 주기로 get_data.py가 실행 되어 카테고리별 베스트 100 상품들의 정보를 수집, csv파일 형태로 원본 데이터가 생성   
2. /root/data에 저장되면 곧바로 active namenode인 nn01에 ssh통신을 통해 배포 후 원본 데이터 삭제(rank, title, price, img, title_xapth, price_xpath, img_xpath)   
3. 원본 csv파일이 nn01에 전달되면 nifi를 통해 후 작업 
![Screenshot_20](https://user-images.githubusercontent.com/66659846/112711786-6020ab80-8f0e-11eb-9542-831890e6e512.png)
![Screenshot_19](https://user-images.githubusercontent.com/66659846/112711787-61ea6f00-8f0e-11eb-8e06-3a0170ae6496.png)


