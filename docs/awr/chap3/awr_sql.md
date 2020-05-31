---
layout: default
title: awr SQL 보고서 생성
parent: chapter3
nav_order: 3
---

### AWR SQL 보고서 생성
- 비효율적으로 수행되는 SQL에 대한 전체 구문 및 실행 계획과 실행 통계 정보를 분석해야할 경우에 사용.
- awrsqprt.sql 또는 awrsqprti.sql 활용.

```sql

[SYS@EDUALL]> @?/rdbms/admin/awrsqrpt.sql

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
Enter value for report_type: text

Type Specified:  text


Instances in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   DB Id     Inst Num DB Name      Instance     Host
------------ -------- ------------ ------------ ------------
* 236423250         1 EDUALL       EDUALL       cedu02.dbinf
                                                ra

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
                              20118 31 May 2020 14:00      1
                              20119 31 May 2020 15:00      1
                              20120 31 May 2020 16:00      1



Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 20104
Begin Snapshot Id specified: 20104

Enter value for end_snap: 20120
End   Snapshot Id specified: 20120




Specify the SQL Id
~~~~~~~~~~~~~~~~~~
Enter value for sql_id: 6gvch1xu9ca3g
SQL ID specified:  6gvch1xu9ca3g

Specify the Report Name
~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrsqlrpt_1_20104_20120.txt.  To use this name,
press <return> to continue, otherwise enter an alternative.

Enter value for report_name: sql_test 

Using the report name sql_test

```

**떨어진 파일 확인**

```sql

[oracle]$ cat sql_test.lst


WORKLOAD REPOSITORY SQL Report

Snapshot Period Summary

DB Name         DB Id    Instance     Inst Num Startup Time    Release     RAC
------------ ----------- ------------ -------- --------------- ----------- ---
EDUALL         236423250 EDUALL              1 17-Mar-20 10:26 11.2.0.4.0  NO

              Snap Id      Snap Time      Sessions Curs/Sess
            --------- ------------------- -------- ---------
Begin Snap:     20104 31-May-20 00:00:02        31       1.7
  End Snap:     20120 31-May-20 16:00:51        30       1.5
   Elapsed:              960.81 (mins)
   DB Time:                1.63 (mins)

SQL Summary                         DB/Inst: EDUALL/EDUALL  Snaps: 20104-20120

                Elapsed
   SQL Id      Time (ms)
------------- ----------
6gvch1xu9ca3g     15,597
Module: DBA
사용 SQL이 들어옴.

          -------------------------------------------------------------

SQL ID: 6gvch1xu9ca3g               DB/Inst: EDUALL/EDUALL  Snaps: 20104-20120
-> 1st Capture and Last Capture Snap IDs
   refer to Snapshot IDs witin the snapshot range
-> 사용 SQL이 들어옴. broken...

    Plan Hash           Total Elapsed                 1st Capture   Last Capture
#   Value                    Time(ms)    Executions       Snap ID        Snap ID
--- ---------------- ---------------- ------------- ------------- --------------
1   0                          15,597           960         20105          20120
          -------------------------------------------------------------


Plan 1(PHV: 0)
--------------

Plan Statistics 
DB/Inst: EDUALL/EDUALL  Snaps: 20104-20120
-> % Total DB Time is the Elapsed Time of the SQL statement divided
   into the Total Database Time multiplied by 100

Stat Name                                Statement   Per Execution % Snap
---------------------------------------- ---------- -------------- -------
Elapsed Time (ms)                            15,597           16.2    15.9
CPU Time (ms)                                15,335           16.0    13.5
Executions                                      960            N/A     N/A
Buffer Gets                                 708,842          738.4     6.8
Disk Reads                                   93,423           97.3    59.4
Parse Calls                                     960            1.0     0.7
Rows                                            960            1.0     N/A
User I/O Wait Time (ms)                         665            N/A     N/A
Cluster Wait Time (ms)                            0            N/A     N/A
Application Wait Time (ms)                        0            N/A     N/A
Concurrency Wait Time (ms)                        0            N/A     N/A
Invalidations                                     0            N/A     N/A
Version Count                                    16            N/A     N/A
Sharable Mem(KB)                                889            N/A     N/A
          -------------------------------------------------------------

Execution Plan
                  No data exists for this section of the report.


Full SQL Text

SQL ID       SQL Text
------------ -----------------------------------------------------------------
6gvch1xu9ca3 sql text가 들어옴.

```