---
layout: default
title: awr 정보 검색 및 기타 보고서 
parent: chapter3
nav_order: 4
---

### AWR 정보 검색 보고서 생성
- awrinfo.sql을 실행하면 AWR에 저장된 정보를 확인할 수 있음.

| 구분            | 항목                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AWR 스냅샷 정보 | DB 유저별 SYSAUX 테이블스페이스 사용 정보<br>기능별 SYSAUX 테이블스페이스 사용 정보<br>일반 DB 유저의 SYSAUX 테이블스페이스 사용 정보<br>일별과 주별 AWR 공간 사용 정보<br>AWR의 구성요소별 SYSAUX 테이블스페이스 사용 정보<br>비AWR의 구성요소별 SYSAUX 테이블스페이스 사용 정보<br>최근 50개의 AWR 스냅샷 수행 정보<br>AWR 수행 시 에러 발생 정보<br>AWR 설정 정보(수행 주기, 보존 기간)<br>AWR 구성요소별 각 스냅샷에 저장된 행의 수<br>AWR 구성요소별 스냅샷에 저장된 평균 행의 수<br>AWR 스냅샷에 의해 수집되고 사용된 DB 구성요소의 개수 |
| 권고자 구성정보 | 권고자 관련 작업 수행 결과 (최근 50개)<br>권고자 관련 작업 수행 결과 (가장 오래된 5개)<br>에러가 발생한 권고자 관련 작업의 상세 정보 (최근 50개)                                                                                                                                                                                                                                                                                                                                                                                               |
| ASH 사용정보    | ASH 통계 정보<br>ASH 기록 세션 중 서버 프로세스와 백그라운드 프로세스 비율                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |


```sql

@?/rdbms/admin/awrinfo.sql


Specify the Report File Name
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrinfo.txt.  To use this name,
press <return> to continue, otherwise enter an alternative.

Enter value for report_name: awrinfo.txt

Using the report name awrinfo.txt
No errors.
No errors.
~~~~~~~~~~~~~~~
AWR INFO Report
~~~~~~~~~~~~~~~


```

**아래와 같은 정보를 보여줌**

```sql

[oracle]$ cat awrinfo.txt
~~~~~~~~~~~~~~~
AWR INFO Report
~~~~~~~~~~~~~~~

Report generated at                                                                                           
16:26:43 on May 31, 2020 ( Sunday ) in Timezone +09:00                                                        
                                                                                                              

Warning: Non Default AWR Setting!                                                                             
--------------------------------------------------------------------------------                              
Snapshot interval is 60 minutes and Retention is 8 days                                                       


       DB_ID DB_NAME   HOST_PLATFORM                             INST STARTUP_TIME      LAST_ASH_SID PAR      
------------ --------- ---------------------------------------- ----- ----------------- ------------ ---      
* 236423250  EDUALL    cedu02.dbinfra - Linux x86 64-bit            1 10:26:55 (03/17)      72375198 NO       

########################################################
(I) AWR Snapshots Information
########################################################

*****************************************************
(1a) SYSAUX usage - Schema breakdown (dba_segments)
*****************************************************
|                                                                                                             
| Total SYSAUX size                        830.0 MB ( 3% of 32,768.0 MB MAX with AUTOEXTEND ON )              
|                                                                                                             
| Schema  SYS          occupies            459.8 MB (  55.4% )                                                
                                                                                                       
********************************************************                                                      
(1b) SYSAUX occupants space usage (v$sysaux_occupants)                                                        
********************************************************                                                      
|                                                                                                             
| Occupant Name        Schema Name               Space Usage                                                  
| -------------------- -------------------- ----------------                                                  
| SM/AWR               SYS                          216.5 MB                                                  
| SM/OPTSTAT           SYS                          130.1 MB                                                  
| XDB                  XDB                          126.9 MB                                                  
| SDO                  MDSYS                         66.6 MB                                                  
| EM                   SYSMAN                        48.8 MB                                                  
                                                                                                      

******************************************
(1c) SYSAUX usage - Unregistered Schemas
******************************************

| This section displays schemas that are not registered                                                       
| in V$SYSAUX_OCCUPANTS                                                                                       
|                                                                                                             
| Schema  APEX_030200  occupies             82.4 MB                                                           
|                                                                                                             
| Total space                               82.4 MB                                                           
|                                                                                                             
                                                                                                              
*************************************************************
(1d) SYSAUX usage - Unaccounted space in registered schemas
*************************************************************
|
| This section displays unaccounted space in the registered
| schemas of V$SYSAUX_OCCUPANTS.
|                                                                                                             
| Unaccounted space in SYS/SYSTEM          -16.4 MB                                                           
|                                                                                                             
| Total space                              -16.4 MB                                                           
|                                                                                                             
*************************************                                                                         
(2) Size estimates for AWR snapshots                                                                          
*************************************                                                                         
|                                                                                                             
| Estimates based on 60 mins snapshot INTERVAL:                                                               
|    AWR size/day                           25.0 MB (1,066 K/snap * 24 snaps/day)                             
|    AWR size/wk                           174.9 MB (size_per_day * 7) per instance                           
|                                                                                                             
| Estimates based on 24 snaps in past 24 hours:                                                               
|    AWR size/day                           25.0 MB (1,066 K/snap and 24 snaps in past 24 hours)              
|    AWR size/wk                           174.9 MB (size_per_day * 7) per instance                           
|                                                                                                             

**********************************
(3a) Space usage by AWR components (per database)
**********************************

COMPONENT        MB  % AWR  KB_PER_SNAP MB_PER_DAY MB_PER_WEEK TABLE% : INDEX%                                
--------- --------- ------ ------------ ---------- ----------- ----------------                               
FIXED         119.1   55.0          586       13.7        96.2    56% : 44%                                   
SQLPLAN        26.0   12.0          128        3.0        21.0    65% : 35%                                   
EVENTS         21.5    9.9          106        2.5        17.4    59% : 41%                                   
SPACE          10.9    5.0           54        1.3         8.8    59% : 41%                                   
SQL            10.8    5.0           53        1.2         8.7    61% : 39%                                   
ASH             1.9    0.9           10        0.2         1.6    58% : 42%                                   
SQLTEXT         0.9    0.4            4        0.1         0.7    93% : 7%                                    
RAC             0.6    0.3            3        0.1         0.5    50% : 50%                                   
SQLBIND         0.6    0.3            3        0.1         0.5    56% : 44%                                   

**********************************
(3b) Space usage within AWR Components (> 500K)
**********************************

COMPONENT        MB SEGMENT_NAME - % SPACE_USED                                           SEGMENT_TYPE        
--------- --------- --------------------------------------------------------------------- ---------------     
FIXED          17.0 WRH$_SYSMETRIC_HISTORY                                        -  80%  TABLE               
FIXED          12.0 WRH$_SYSMETRIC_HISTORY_INDEX                                  -  85%  INDEX               
FIXED           7.0 WRH$_SYSMETRIC_SUMMARY                                        -  40%  TABLE               

COMPONENT        MB SEGMENT_NAME - % SPACE_USED                                           SEGMENT_TYPE        
--------- --------- --------------------------------------------------------------------- ---------------     
EVENTS          0.5 WRH$_EVENT_HISTOGRAM.WRH$_EVENT__236423250_19915              -  84%  TABLE PARTITION     
EVENTS          0.5 WRH$_EVENT_HISTOGRAM.WRH$_EVENT__236423250_19891              -  84%  TABLE PARTITION     

**********************************
(4) Space usage by non-AWR components (> 500K)
**********************************

COMPONENT        MB SEGMENT_NAME                                                          SEGMENT_TYPE        
--------- --------- --------------------------------------------------------------------- ---------------     
NON_AWR        54.2 XDB.SYS_LOB0000069716C00025$$                                         LOBSEGMENT          
NON_AWR        49.0 SYS.I_WRI$_OPTSTAT_H_OBJ#_ICOL#_ST                                    INDEX               
NON_AWR        31.0 SYS.WRI$_OPTSTAT_HISTGRM_HISTORY                                      TABLE               
NON_AWR        23.0 SYS.I_WRI$_OPTSTAT_H_ST                                               INDEX               
                                         TABLE               

**********************************
(5a) AWR snapshots - last 50
**********************************

Total snapshots in DB 236423250 Instance 1 = 208                                                              

      DBID    SNAP_ID  INST FLUSH_ELAPSED        ENDTM             STARTUP_TIME      STATUS ERRCNT            
---------- ---------- ----- -------------------- ----------------- ----------------- ------ ------            
 236423250      20071     1 +00000 00:00:00.5    15:00:54 (05/29)  10:26:55 (03/17)       0      0            
 236423250      20072     1 +00000 00:00:01.0    16:00:24 (05/29)  10:26:55 (03/17)       0      0            
 236423250      20073     1 +00000 00:00:00.6    17:00:43 (05/29)  10:26:55 (03/17)       0      0            
 236423250      20074     1 +00000 00:00:01.1    18:00:01 (05/29)  10:26:55 (03/17)       0      0            

**********************************
(5b) AWR snapshots with errors or invalid
**********************************

no rows selected


**********************************
(5c) AWR snapshots -- OLDEST Non-Baselined snapshots
**********************************

      DBID  INST    SNAP_ID ENDTM             STATUS ERROR_COUNT                                              
---------- ----- ---------- ----------------- ------ -----------                                              
 236423250     1      19913 01:00:53 (05/23)       0           0                                              

**********************************
(6) AWR Control Settings - interval, retention
**********************************

       DBID  LSNAPID LSPLITID LSNAPTIME      LPURGETIME      FLAG INTERVAL          RETENTION         VRSN    
----------- -------- -------- -------------- -------------- ----- ----------------- ----------------- ----    
  236423250    20120    20105 05/31 16:00:52 05/31 00:07:26     2 +00000 01:00:00.0 +00008 00:00:00.0    5    

**********************************
(7a) AWR Contents - row counts for each snapshots
**********************************

   SNAP_ID  INST        ASH        SQL      SQBND      FILES      SEGST     SYSEVT                            
---------- ----- ---------- ---------- ---------- ---------- ---------- ----------                            
     20071     1          3         71        225         29         90        118                            
     20072     1          5         70        203         29         89        118                            
     20073     1          5         71        214         29         89        118                            
     20074     1          3         70        204         29         88        118                            
     20075     1          1         71        216         29         89        118                            
     20076     1          2         73        207         29         90        118                            

**********************************
(7b) AWR Contents - average row counts per snapshot
**********************************

SNAP_COUNT  INST        ASH    SQLSTAT    SQLBIND      FILES    SEGSTAT   SYSEVENT                            
---------- ----- ---------- ---------- ---------- ---------- ---------- ----------                            
       208     1       8.52      81.37     284.39      27.79      88.72     117.38                            

**********************************
(7c) AWR total item counts - names, text, plans
**********************************

   SQLTEXT    SQLPLAN   SQLBMETA     SEGOBJ   DATAFILE   TEMPFILE                                             
---------- ---------- ---------- ---------- ---------- ----------                                             
       340       7874       1673        958         29          1                                             


########################################################
(II) Advisor Framework Info
########################################################

**********************************
(1) Advisor Tasks - Last 50
**********************************

OWNER/ADVISOR  TASK_ID/NAME                     CREATED          EXE_DURATN EXE_CREATN HOW_C STATUS           
-------------- -------------------------------- ---------------- ---------- ---------- ----- ------------     
SYS/SQL Tuning 1/SYS_AUTO_SQL_TUNING_TASK       15:07:18 (02/12)          1 ########## AUTO  COMPLETED        
SYS/ADDM       20211/ADDM:236423250_1_20072     16:00:26 (05/29)          0          0 AUTO  COMPLETED        
SYS/ADDM       20212/ADDM:236423250_1_20073     17:00:44 (05/29)          0          0 AUTO  COMPLETED        

**********************************
(2) Advisor Task - Oldest 5
**********************************

OWNER/ADVISOR  TASK_ID/NAME                     CREATED          EXE_DURATN EXE_CREATN HOW_C STATUS           
-------------- -------------------------------- ---------------- ---------- ---------- ----- ------------     
SYS/ADDM       19514/ADDM:236423250_1_19385     01:00:45 (05/01)          0          0 AUTO  COMPLETED        

**********************************
(3) Advisor Tasks With Errors - Last 50
**********************************

no rows selected



########################################################
(III) ASH Usage Info
########################################################

**********************************
(1a) ASH histogram (past 3 days)
**********************************

NUM_ACTIVE_SESSIONS   NUM_SAMPLES                                                                             
-------------------- ------------                                                                             
0000 - 0004                   249                                                                             

**********************************
(1b) ASH histogram (past 1 day)
**********************************

NUM_ACTIVE_SESSIONS   NUM_SAMPLES                                                                             
-------------------- ------------                                                                             
0000 - 0004                    85                                                                             

**********************************
(2a) ASH details (past 3 days)
**********************************

INST MIN_TIME         MAX_TIME          NUM_SAMPLES     NUM_ROWS AVG_ACTIVE                                   
---- ---------------- ---------------- ------------ ------------ ----------                                   
   1 17:01:06 (05/28) 15:55:45 (05/31)       25,523          260       0.01                                   

**********************************
(2b) ASH details (past 1 day)
**********************************

INST MIN_TIME         MAX_TIME          NUM_SAMPLES     NUM_ROWS AVG_ACTIVE                                   
---- ---------------- ---------------- ------------ ------------ ----------                                   
   1 17:15:49 (05/30) 15:55:45 (05/31)        8,158           89       0.01                                   

**********************************
(2c) ASH sessions (Fg Vs Bg) (past 1 day across all instances in RAC)
**********************************

Foreground %           14.6                                                                                   
Background %           85.4                                                                                   
MMNL %                  0.0                                                                                   
                                                                                                              
End of Report

```

### 기타 AWR 보고서 생성 관련 스크립트
- 다른 스크립트에 의해 호출되거나 동일한 기능이 다른 스크립트에 포함되어 있어서 사용하지 않는 스크립트


| 스크립트     | 설명                                                                                                                                                        |
|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| awrddinp.sql | awrddrpti.sql 스크립트에서 호출되어서 사용되며, AWR DB 비교 보고서 생성에서 실제 이 스크립트가 수행.<br>awrddrpti.sql은 AWR에서 정보를 수집하는 기능만 수행 |
| awrinpnm.sql | awrrpti.sql, awrsqrpi.sql에 의해 호출되어서 사용.<br>awrrpti.sql, awrsqrpi.sql은 AWR에서 정보를 수집하는 기능만 수행                                        |
| awrinput.sql | awrrpti.sql, awrsqrpi.sql에 의해 호출되어서 사용.                                                                                                           |
