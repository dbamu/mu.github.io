---
layout: default
title: awr 생성
parent: chapter3
nav_order: 1
---

### AWR 보고서 생성

- V$view(Dynamic View)에 기록되는 수치들은 instance가 시작되면 수집되기 시작해서 종료되기 전까지 지속적으로 증가
  - 메모리에만 저장되므로 인스턴스 종료 시 다시 0으로 재설정.

- AWR snapshot : memory에 저장된 데이터가 디스크로 저장
  - 시작/종료 스냅샷 번호 설정
  - 두 구간의 wait event 발생 횟수 및 통게치의 차이를 계산
  - 주의점 : instance 종료할 때마다 대기 이벤트 및 통계 수치가 재설정되므로 종료 전 snapshot과 재시작 후의 snapshot 비교는 무의미.
  - DBMS_WORKLOAD_REPOSITORY 패키지 or 스크립트($ORACLE_HOME/rdbms/admin/)를 사용해서 생성 가능.
  - 소수의 보고서 생성 시 => 스크립트 활용 / 여러 개의 보고서를 생성 시 패키지를 이용.
  
  ![](http://wiki.gurubee.net/download/attachments/26742265/AWR07.JPG){: width="700" height="500" .center}

| 사용용도                | 스크립트                                 |
|-------------------------|------------------------------------------|
| AWR DB 보고서 생성      | awrrpt.sql, awrrpti.sql                  |
| AWR DB 비교 보고서 생성 | awrddrpt.sql, awrddrpi.sql               |
| AWR SQL 보고서 생성     | awrsqrpt.sql, awrsqrpi.sql               |
| AWR 정보 검색 보고서    | awrinfo.sql                              |
| ASH 보고서 생성         | ashrpt.sql, ashrpti.sql                  |
| ADDM 보고서 생성        | addmrpt.sql, addmrpti.sql                |
| 기타 스크립트           | awrddinp.sql, awrinput.sql, awrinpnmp.sq |

- AWR 보고서를 생성하기 위한 권한.
  - SELECT ANY DICTIONARY 권한
  - SYS.DBMS_WORKLOAD_REPOSITORY 패키지의 execute 권한.
  
| 스크립트    | 설명                                                            |
|-------------|-----------------------------------------------------------------|
| awrrpt.sql  | 스크립트를 수행한 인스턴스의 AWR DB 보고서를 생성               |
| awrrpti.sql | 데이터베이스 ID와 인스턴스 번호를 지정해서 AWR DB 보고서를 생성 |

**awrrpt.sql을 활용한 보고서 생성 예제**

```sql
[SYS@EDUALL]> @?/rdbms/admin/awrrpt.sql

Current Instance
~~~~~~~~~~~~~~~~

   DB Id    DB Name      Inst Num Instance
----------- ------------ -------- ------------
  236423250 EDUALL              1 EDUALL


Specify the Report Type
~~~~~~~~~~~~~~~~~~~~~~~
Would you like an HTML report, or a plain text report?
Enter 'html' for an HTML report, or 'text' for plain text
Defaults to 'html'
Enter value for report_type: html

Type Specified:  html


Instances in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   DB Id     Inst Num DB Name      Instance     Host
------------ -------- ------------ ------------ ------------
* 236423250         1 EDUALL       EDUALL       host명

Using  236423250 for database Id
Using          1 for instance number


Specify the number of days of snapshots to choose from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Entering the number of days (n) will result in the most recent
(n) days of snapshots being listed.  Pressing <return> without
specifying a number lists all completed snapshots.


Enter value for num_days: 1

Listing the last day's Completed Snapshots

                                                        Snap
Instance     DB Name        Snap Id    Snap Started    Level
------------ ------------ --------- ------------------ -----
EDUALL       EDUALL           20104 31 May 2020 00:00      1
                              20105 31 May 2020 01:00      1
                              20106 31 May 2020 02:00      1
                              20107 31 May 2020 03:00      1
                              20108 31 May 2020 04:00      1
                              20109 31 May 2020 05:00      1
                              20110 31 May 2020 06:00      1
                              20111 31 May 2020 07:00      1
                              20112 31 May 2020 08:00      1
                              20113 31 May 2020 09:00      1
                              20114 31 May 2020 10:00      1
                              20115 31 May 2020 11:00      1
                              20116 31 May 2020 12:00      1
                              20117 31 May 2020 13:00      1



Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 20104  
Begin Snapshot Id specified: 20104

Enter value for end_snap: 20105
End   Snapshot Id specified: 20105



Specify the Report Name
~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrrpt_1_20104_20105.html.  To use this name,
press <return> to continue, otherwise enter an alternative.

Enter value for report_name: test_awr.html

Using the report name test_awr.html

]$ ls | grep *.html                                                                                               
test_awr.html


```

### DBMS_WORKLOAD_REPOSITORY 패키지 사용
- AWR DB 보고서 대량으로 생성할 경우 사용

| 프로시저        | 설명                                 |
|-----------------|--------------------------------------|
| AWR_REPORT_TEXT | 텍스트 형식으로 AWR DB 보고서를 생성 |
| AWR_REPOST_HTML | 웹 문서 형식으로 DB 보고서를 생성    |

**패키지로 생성하는 방법**

```sql
[SYS@EDUALL]> select dbid, name from v$database;

      DBID NAME
---------- ---------
 236423250 EDUALL

col instance_number for 999
col instance_name for a20
col host_name for a30

select instance_number, instance_name, host_name from v$instance;

INSTANCE_NUMBER INSTANCE_NAME        HOST_NAME
--------------- -------------------- ------------------------------
              1 EDUALL               cedu02.dbinfra


SELECT OUTPUT
FROM  TABLE(DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_TEXT(
                      :DBID, :INST_ID, :BEGIN_SNAP, :END_SNAP));


SELECT OUTPUT
FROM  TABLE(DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_HTML(
                      :DBID, :INST_ID, :BEGIN_SNAP, :END_SNAP));
```

| 항목        | 설명            |
|-------------|-----------------|
| :DBID       | 데이터베이스 ID |
| :INST_ID    | 인스턴스 번호   |
| :BEGIN_SNAP | 시작 스냅 ID    |
| :END_SNAP   | 종료 스냅 ID    |

**awr_db_report_make.sh**
- 대량으로 만들 때 스크립트로 활용
- 아래 스크립트 수행 시 dbid와 inst_id 변경 필요

```bash

[oracle]$ cat awr_db_report_make.sh
# AWR_DB_REPORT_MAKE.sh

#!/bin/ksh
if [ "$#" -lt 5 ]
then
   echo "Usage : make_awr_rpt.sh  Name No Begin_snap  End_snap  text/html" 
   exit
fi

name=$1
no=$2
b_snap=$3
e_snap=$4
format=$5

sqlplus -s '/as sysdba' << EOF

set echo off
set heading on
set underline on

column inst_num  heading "Inst Num"  new_value inst_num format 99999;
column inst_name heading "Instance"  new_value inst_name format a12;
column db_name  heading "DB Name"   new_value db_name format a12;
column dbid  heading "DB Id"  new_value dbid  format 9999999999 just c;

set verify   off
set feedback off
set linesize 80
set termout  off
set heading  off
set pagesize 0

SPOOL ${name}-NO${no}-awr_report.${format}
SELECT OUTPUT 
FROM TABLE(DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_${format}( 
236423250, 1, ${b_snap}, ${e_snap} ));
spool off

undefine num_days;
undefine report_type;
undefine report_name;
undefine begin_snap;
undefine end_snap;

exit
EOF

```

**스크립트 수행**

```bash
sh awr_db_report_make.sh test_bulk 01 20108 20109 text 

[oracle@cedu02-EDUALL:/oracle/dba/mu/awr]$ ls | grep text
test_bulk-NO01-awr_report.text
```


