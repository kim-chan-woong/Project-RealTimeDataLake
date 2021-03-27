# Real-Time-Data-Pipeline-DW-DM

# 실시간 데이터 수집 및 적재 (from gmarket)

# process
![Screenshot_24](https://user-images.githubusercontent.com/66659846/112711394-5ba6c380-8f0b-11eb-9a3a-d92790bd22fa.png)

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

# 서버별 역할
서버1(nn01): active namenode, zookeeper, journalnode, nifi and hive master 역할   
서버2(rm01): standby namenode, zookeeper, journalnode, ResourceManager 역할   
서버3(jn01): zookeeper, journalnode 역할   
서버4(dn01): NodeManager, Datanode 역할   
서버5(dn02): NodeManager, Datanode 역할   
서버6(dn03): NodeManager, Datanode 역할   
서버7(dbserver): Postgre, DataMart 역할   
서버8(getdataserver): Source Data 수집 역할   

