---
layout: default
title: summary report
parent: chapter4
nav_order: 1
---

### 요약 보고서

- 스냅샷 구간의 인스턴스 정보 및 문제 발생 여부의 빠른 파악
- 문제 해결 방향 설정

| 항목        | 보고서                                                                                                                                                                                                                      |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DB 환경     | 처음부터 Cache Size 단위 보고서 전까지 부분                                                                                                                                                                                 |
| 메모리      | Cache Size, Instance Efficiency Percentages(Target 100%)                                                                                                                                                                    |
| 부하 발생   | Load PROFILE                                                                                                                                                                                                                |
| 대기 이벤트 | Top 5 Timed Events                                                                                                                                                                                                          |
| RAC         | Global Cache Load Profile,<br>Global Cache Efficiency Percentages(Target local + remote 100%),<br>Global Cache and Enqueue Services - Workload Characteristics,<br>Global Cache and Enqueue Services - messaging Statistics |


### 인스턴스 정보

![instance_info]()

| 항목     | 의미                          | 관련컬럼        |
|----------|-------------------------------|-----------------|
| DB Name  | 데이터베이스 이름             | DB_NAME         |
| DB ID    | 데이터베이스 ID               | DBID            |
| Instance | 인스턴스 이름                 | INSTANCE_NAME   |
| InstNum  | 인스턴스 번호                 | INSTANCE_NUMBER |
| Release  | 오라클 버전 정보              | VERSION         |
| RAC      | RAC 여부 확인                 | PARALLEL        |
| Host     | 인스턴스가 설치된 시스템 이름 | HOST_NAME       |

- dba_hist_database_instance
- v$instance 뷰 활용

```sql

select db_name, dbid, instance_name, startup_time, version, host_name, platform_name 
from dba_hist_database_instance;

select instance_name, startup_time, version, host_name from v$instance;

```

### AWR 스냅샷 정보

![awr_snapshot_info]()

| 항목      | 의미                                                     | 관련컬럼                                                                                                                                                                                                             |
|-----------|----------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Snap ID   | AWR 스냅샷 번호                                          | A.SNAP_ID                                                                                                                                                                                                            |
| Snap Time | AWR 스냅샷이 수행된 시간                                 | A.END_INTERVAL_TIME                                                                                                                                                                                                  |
| Sessions  | AWR 스냅샷 구간에서 세션 수(active + inactive)                               | SUM(C.VALUE) WHERE STAT_NAME = 'logons current'                                                                                                                                                                      |
| Cur/Sess  | AWR 스냅샷 구간에서 세션이 사용한 평균 커서의 수          | (SUM(C.VALUE) WHERE STAT_NAME 'opened cursors current') / (SUM(C.VALUE) WHERE STAT_NAME 'logons current')                                                                                                            |
| Elapsed   | AWR 스냅샷 구간에서 소요시간(단위:분)                      | (EXTRACT(DAY FROM(A.END_INTERVAL_TIME)) * 86400) +<br>(EXTRACT(HOUR FROM(A.END_INTERVAL_TIME)) * 3600) +<br>(EXTRACT(MINUTE FROM(A.END_INTERVAL_TIME)) * 60) +<br>(EXTRACT(SECOND FROM(A.END_INTERVAL_TIME)) / 60) + |
| DB Time   | AWR 스냅샷 구간에서 데이터베이스를 사용한 총 시간(단위:분) | (SUM(B.VALUE) WHERE STAT_NAME = 'DB Time') / 1000000 / 60                                                                                                                                                            |

- A : DBA_HIST_SNAPSHOT
- B : DBA_HIST_SYS_TIME_MODEL
- C : DBA_HIST_SYSSTAT

```sql

-- 스냅샷 정보
select snap_id, instance_number, startup_time, begin_interval_time, end_interval_time  from dba_hist_snapshot a
where (a.begin_interval_time >= to_date('2020/06/07 09:30','YYYY/MM/DD HH24:mi') 
and a.END_INTERVAL_TIME < to_date('2020/06/07 10:01','YYYY/MM/DD HH24:mi')) 
and instance_number = 1;

-- 스냅샷 구간의 세션 수
select sum(c.value)  
from dba_hist_snapshot a, dba_hist_sysstat c
where (a.begin_interval_time >= to_date('2020/06/07 09:30','YYYY/MM/DD HH24:mi') 
and a.END_INTERVAL_TIME < to_date('2020/06/07 10:01','YYYY/MM/DD HH24:mi')) 
and a.instance_number = 1
and a.snap_id = c.snap_id
and a.instance_number = c.instance_number
and c.stat_name= 'logons current';


-- 스냅샷 구간에서 열렸던 커서 갯수
select sum(c.value)  
from dba_hist_snapshot a, dba_hist_sysstat c
where (a.begin_interval_time >= to_date('2020/06/07 09:30','YYYY/MM/DD HH24:mi') 
and a.END_INTERVAL_TIME < to_date('2020/06/07 10:01','YYYY/MM/DD HH24:mi')) 
and a.instance_number = 1
and a.snap_id = c.snap_id
and a.instance_number = c.instance_number
and c.stat_name= 'opened cursors current';


-- 스냅샷 구간의 소요 시간(Elapsed) => 단위 분 
select 
(A.END_INTERVAL_TIME - A.begin_interval_time)  
from dba_hist_snapshot a 
where a.begin_interval_time >= to_date('2020/06/07 09:30','YYYY/MM/DD HH24:mi') 
and a.END_INTERVAL_TIME <= to_date('2020/06/07 10:01','YYYY/MM/DD HH24:mi') 
and a.instance_number = 1;

-- 스냅샷 구간 DB time 확인 => 단위 분
select SUM(B.VALUE) / 1000000 / 60
from dba_hist_snapshot a, DBA_HIST_SYS_TIME_MODEL b
where (a.begin_interval_time >= to_date('2020/06/07 09:30','YYYY/MM/DD HH24:mi') 
and a.END_INTERVAL_TIME < to_date('2020/06/07 10:01','YYYY/MM/DD HH24:mi')) 
and a.instance_number = 1
and a.snap_id = b.snap_id
and a.instance_number = b.instance_number
and upper(b.stat_name)= upper('DB time');

```

### Cache size

| 항목           | 의미                                                         | 관련컬럼                                           |
|----------------|--------------------------------------------------------------|----------------------------------------------------|
| Buffer Cache   | 버퍼 캐시 크기                                               | A.VALUE WHERE PARAMETER_NAME = '_db_cache_size'    |
| Shared Pool    | 공유 풀 크기                                                 | A.VALUE WHERE PARAMETER_NAME = '_shared_pool_size' |
| Std Block Size | DB_bloCK_SIZE 초기화 파라미터로 설정한 기본 데이터 블록 크기 | A.VALUE WHERE PARAMETER_NAME = 'db_block_size'     |
| Log Buffer     | 리두 로그 버퍼 크기                                          | ROUND(B.bytes / 1024, 0) WHERE NAME = 'log_buffer' |

- A : DBA_HIST_PARAMETER => buffer cache / shared pool / std block size 확인 / 히든 파라미터 조회 가능
- B : DBA_HIST_SGASTAT => log buffer 확인
   - log buffer : ASMM(Automatic Shared Memory Management) 사용 시 로그 버퍼의 크기 변경 X , 고정 크기
   - ASMM 사용 시 buffer cache, shared pool, large pool, java pool, stream pool 크기 자동 조절
   

### Load Profile

- DB 튜닝 필요성 유무를 판단하기 보다는 해당 구간에 발생한 부하량 파악 및 비정상적인 구간과의 비교를 위해 사용

| 항목                       | 의미                                                                  | 관련컬럼                                                 |
|----------------------------|-----------------------------------------------------------------------|----------------------------------------------------------|
| Redo size                  | 발생한 리두 로그의 크기(단위 : 바이트)                                | redo size                                                |
| Logical reads              | 메모리 읽기 I/O 발생 횟수                                             | session logical reads                                    |
| Block changes              | SGA 메모리 블록을 변경한 횟수                                         | db block changes                                         |
| Physical reads             | 디스크에서 메모리로 읽어온 횟수(SGA + PGA)                                       | physical reads                                           |
| Physical writes            | 메모리에서 디스크로 내려쓴 횟수                                       | physical writes                                          |
| User calls                 | 로그인, parse, fetch, sql execute 등 사용자가 요청한 모든 호출의 합        | user calls                                               |
| Parses                     | 소프트 파스와 하드 파스가 발생한 횟수                                 | parse count(total)                                       |
| Hard parses                | 하드 파스 횟수                                                        | parse count(hard)                                        |
| Sorts                      | SQL 수행 시 디스크에 내려쓰지 않고 메모리에서만 발생한 정렬 작업 횟수 | sorts(memory)                                            |
| Logons                     | 스냅샷 구간 중 로그온한 총 횟수                                       | logons cumulative                                        |
| Executes                   | 사용자가 수행한 SQL과 DB 내부에서 실행된 recursive SQL 횟수                  | execute count                                            |
| Transactions               | 스냅샷 구간 중 발생한 트랜잭션 횟수(user commit + rollback)                                   | user commits + user rollbacks                            |
| % Blocks changed per Read  | 메모리 읽기 I/O당 블록 변경 비율                                      | 100 * (db bock changes / session logical reads)          |
| Rollback per transaction % | 전체 트랜잭션 횟수 중 롤백 비율                                       | 100 * (user rollbacks / (user commits + user rollbacks)) |
| Recursive Call %           | 전체 SQL 중 recursive SQL 비율                                             | 100 * (recursive calls / (user calls + recursive calls)) |
| Rows per Sort              | 정렬 작업 시 평균 행의 수                                             | (sort(rows) / (sorts(memory) + sorts(disk))              |


- DBA_HIST_SYSSTAT 참조 ex) where upper(stat_name) = upper('parse count (total)')
- 관련 동적 뷰 v$sysstat

- physical reads는 SGA영역과 PGA영역에서 발생한 I/O의 합이기 때문에
    - Physical reads > SGA physical reads cache +  SGA physical reads direct 

- Physical writes = Physical writes direct + Physical writes from cache

- 정렬 작업 시 한번이라도 disk에 내려쓸 경우 sorts(disk) 수치에 포함됨.

- transactions = user commits + user rollbacks
    - user commit => 변경된 데이터 없이 commit할 경우 수치 증가 X
    - user rollbacks => 변경된 데이터가 없더라도 명시적으로 롤백을 수행하면 수치가 증가
    - Load Profile의 transaction은 정확한 transaction 양을 나타내지 않음.
    - 정확한 확인을 위해서는 Undo Segment Summary 단위 보고서의 Number of Transactions 칼럼 값을 확인.
        - Number of Transaction = 트랜잭션 발생 시 undo segment에 기록된 트랜잭션 양을 기준으로 산출.
        - 따라서 Load Profile 단위 보고서의 Tranaction 값 보다 정확.
     
     ![tran_diff]()
     
     
- DB에 발생한 부하량을 나타내는 지표( User calls, Execute, Transactions) 
- 데이터 변경 발생량을 나타내는 지표 ( Redo Size, Block Changes )
   - 소수의 SQL로 수행되는 대형 배치 작업으로 인한 문제가 발생할 경우 부하량 지표는 증가 X
   - 전체적인 부하 발생 증가 시 부하량 + 데이터 변경 발생량 모두 증가.
   - 예제 참고 페이지 70쪽

### Instance Efficency Percentages(Target 100%)

sult (click "Generate" to refresh) Copy to clipboard  Preview
| 항목                        | 의미                                                                                                                                                                      | 관련컬럼 |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| Buffer Nowait %             | 버퍼 캐시 요청 시 대기 없이 원하는 버퍼 캐시를 획득한 비율                                                                                                                |          |
| Buffer Hit %                | 필요한 블록을 디스크에서 읽어 오지 않고 버퍼 캐시에서 획득한 비율.<br>RAC 환경의 경우, 캐시 퓨전을 통해 다른 인스턴스의 메모리에서 읽어온 비율도 버퍼 캐시 적중률에 포함. |          |
| Library Hit %               | 공유 풀의 라이브러리 캐시 적중률                                                                                                                                          |          |
| Execute to Parse %          | 파스 없이 SQL이 수행된 비율                                                                                                                                               |          |
| Parse CPU to Parse Elapsd % | 파스 수행 시간 중 CPU 사용 시간 비율.<br>100%에 가까울수록 대기 시간 없이 파스를 수행한 것                                                                                |          |
| Redo NoWait %               | 리두 로그 생성 시 로그 파일 공간 부족으로 log switch complete를 대기하지 않은 비율                                                                                           |          |
| In-memory Sort %            | 전체 정렬 작업 중 메모리에서만 발생한 정렬 작업 비율                                                                                                                      |          |
| Soft Parse %                | 소프트 파스 비율                                                                                                                                                          |          |
| Latch Hit %                 | 래치 요청 시 대기 없이 획득한 비율                                                                                                                                        |          |
| % Non-Parse CPU             | 파스에 사용된 CPU 시간을 제외한 CPU 사용 시간 비율                                                                                                                        |          |
| Memory Usage %              | 시작 스냅샷과 종료 스냅샷 수행 시 공유 풀 사용 비율                                                                                                                       |          |
| % SQL with executions>1     | 전체 SQL 중 1회 이상 수행된 SQL 비율                                                                                                                                      |          |
| % Memory for SQL w/exec>1   | 전체 SQL 중 1회 이상 사용된 SQL이 차지하는 메모리 공간 비율      

 - A : DBA_HIST_WAITSTAT
 - B : DBA_HIST_LIBRARYCACHE
 - C : DBA_HIST_SYSSTAT
 - D : DBA_HIST_LATCH
 - E : DBA_HIST_SYS_TIME_MODEL
 - F : DBA_HIST_SGASTAT
 - G : DBA_HIST_PARAMETERS
 - H : DBA_HIST_SQL_SUMMARY
     

 





