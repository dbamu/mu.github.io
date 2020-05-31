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


