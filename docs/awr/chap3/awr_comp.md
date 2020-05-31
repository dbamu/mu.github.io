---
layout: default
title: awr 비교 보고서 생성
parent: chapter3
nav_order: 2
---


### AWR 비교 보고서 생성
- 기준 구간 수치와 비교 구간의 수치를 나란히 나열
- 차이를 한눈에 비교할 수 있게 해줌.
- 비교 보고서 생성 방법
    - DBMS_WORKLOAD_REPOSITORY 패키지 이용
    - awrddrpt.sql, awrddrpi.sql 스크립트 수행

```sql

[SYS@EDUALL]> @?/rdbms/admin/awrddrpt.sql

Current Instance
~~~~~~~~~~~~~~~~

   DB Id       DB Id    DB Name      Inst Num Inst Num Instance
----------- ----------- ------------ -------- -------- ------------
  236423250   236423250 EDUALL              1        1 EDUALL


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
* 236423250         1 EDUALL       EDUALL       cedu02.dbinf
                                                ra

Database Id and Instance Number for the First Pair of Snapshots
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Using  236423250 for Database Id for the first pair of snapshots
Using          1 for Instance Number for the first pair of snapshots


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
                              20118 31 May 2020 14:00      1
                              20119 31 May 2020 15:00      1



Specify the First Pair of Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 20104
First Begin Snapshot Id specified: 20104

Enter value for end_snap: 20105
First End   Snapshot Id specified: 20105




Instances in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   DB Id     Inst Num DB Name      Instance     Host
------------ -------- ------------ ------------ ------------
* 236423250         1 EDUALL       EDUALL       호스트명




Database Id and Instance Number for the Second Pair of Snapshots
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using  236423250 for Database Id for the second pair of snapshots
Using          1 for Instance Number for the second pair of snapshots


Specify the number of days of snapshots to choose from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Entering the number of days (n) will result in the most recent
(n) days of snapshots being listed.  Pressing <return> without
specifying a number lists all completed snapshots.


Enter value for num_days2: 1

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
                              20118 31 May 2020 14:00      1
                              20119 31 May 2020 15:00      1



Specify the Second Pair of Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap2: 20118
Second Begin Snapshot Id specified: 20118

Enter value for end_snap2: 20119
Second End   Snapshot Id specified: 20119



Specify the Report Name
~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrdiff_1_20104_1_20118.html  To use this name,
press <return> to continue, otherwise enter an alternative.

Enter value for report_name: awr_comp_test.html

Using the report name awr_comp_test.html


```

**비교 결과**

![](https://github.com/lght2000/mu.github.io/blob/master/assets/images/awr_comp.PNG?raw=true){: width="600" height="500"  .center}


**패지지로 비교 보고서 생성**
- 대량으로 생성할 때 사용

| 프로시저             | 설명                                      |
|----------------------|-------------------------------------------|
| AWR_DIFF_REPORT_TEXT | 텍스트 형식으로 AWR DB 비교 보고서를 생성 |
| AWR_DIFF_REPOST_HTML | 웹 문서 형식으로 DB 비교 보고서를 생성    |


```sql

SELECT OUTPUT
FROM  TABLE(DBMS_WORKLOAD_REPOSITORY.AWR_DIFF_REPOST_HTML(
                      :DBID1, :INST_ID1, :BEGIN_SNAP1, :END_SNAP1,
                      :DBID2, :INST_ID2, :BEGIN_SNAP2, :END_SNAP2));

SELECT OUTPUT
FROM  TABLE(DBMS_WORKLOAD_REPOSITORY.AWR_DIFF_REPOST_TEXT(
                      :DBID1, :INST_ID1, :BEGIN_SNAP1, :END_SNAP1,
                      :DBID2, :INST_ID2, :BEGIN_SNAP2, :END_SNAP2));

```

| 항목         | 설명                         |
|--------------|------------------------------|
| :DBID1       | 첫 번째 구간 데이터베이스 ID |
| :INST_ID1    | 첫 번째 구간 인스턴스 번호   |
| :BEGIN_SNAP1 | 첫 번째 구간 시작 스냅 ID    |
| :END_SNAP1   | 첫 번째 구간 종료 스냅 ID    |
| :DBID2       | 두 번째 구간 데이터베이스 ID |
| :INST_ID2    | 두 번째 구간 인스턴스 번호   |
| :BEGIN_SNAP2 | 두 번째 구간 시작 스냅 ID    |
| :END_SNAP2   | 두 번째 구간 종료 스냅 ID    |

