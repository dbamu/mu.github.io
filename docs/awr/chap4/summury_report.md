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

![instance_info](https://github.com/lght2000/mu.github.io/blob/master/assets/images/instance_info.PNG?raw=true)

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

![awr_snapshot_info](https://github.com/lght2000/mu.github.io/blob/master/assets/images/awr_snapshot_info.PNG?raw=true)

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
     
     ![tran_diff](https://github.com/lght2000/mu.github.io/blob/master/assets/images/tran_diff.PNG?raw=true)
     
     
- DB에 발생한 부하량을 나타내는 지표( User calls, Execute, Transactions) 
- 데이터 변경 발생량을 나타내는 지표 ( Redo Size, Block Changes )
   - 소수의 SQL로 수행되는 대형 배치 작업으로 인한 문제가 발생할 경우 부하량 지표는 증가 X
   - 전체적인 부하 발생 증가 시 부하량 + 데이터 변경 발생량 모두 증가.
   - 예제 참고 페이지 70쪽

### Instance Efficency Percentages(Target 100%)

| 항목                        | 의미                                                                                                                                                                      | 관련컬럼 |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| Buffer Nowait %             | 버퍼 캐시 요청 시 대기 없이 원하는 버퍼 캐시를 획득한 비율                                                                                                                |    page 72      |
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
     
- Buffer Nowait % => 아래의 경우 %가 떨어짐.
    - 다른 세션이 읽고 있는 버퍼 블록 읽기를 시도하는 횟수가 증가
    - 서로 다른 세션이 호환되지 않는 lock을 같은 buffer block에 적용하려고 하는 횟수가 증가
    - hot block일 경우 낮은 수치.
    - 일반적으로 95% 이상 값을 유지하는 것을 권장.

- Buffer Hit % => 전체 I/O 중 buffer cache I/O가 차지하는 비율
    - 배치 업무가 주로 수행되는 DW 시스템에서는 디스크 I/O 비율이 높기 때문에 낮게 나옴.
    - OLTP에서는 일반적으로 80% 이상을 유지
    - 수치가 낮을 경우 
        - buffer cache size가 적절한지 확인
        - I/O 발생량이 높은 SQL들을 확인 후 튜닝 진행
    - 정확한 캐시 적중률을 확인하기 위해서는 Buffer Pool Statistics 단위보고서의 Pool Hit% 항목을 참조해야 함
        
    
 - Library Hit %
    - SQL 수행에 필요한 정보를 라이브러리 캐시 영역에서 읽은 비율을 나타냄.
    - 일반적으로 적중률이 95% 이상
    - 수치가 낮을 경우
        - library cache 영역 크기 체크
        - 과도한 hard parse가 발생하는지 확인
        - 상세 내용은 Library Cache Activity 참조

- Execute to Parse %
    - soft parse와 hard parse를 거치치 않고 SQL이 수행된 비율
    - bind 변수 사용 및 library cache 적중률을 높게 유지하면 hard parse 발생을 최대한 피할 수 있음.
    - SESSION_CACHED_CURSORS 값을 높게 설정하면 parse 정보를 세션의 PGA 영역에서 찾기 때문에 sofr parse도 피할 수 있음.
        - SQL 수행 빈도가 아주 높은 OLTP 시스템에서는 과도한 soft parse로 library cache latch에 대한 경합이 높아질 수 있음.
            - SESSION_CACHED_CURSORS 값을 높게 설정하여 soft parse 비율을 낮춘다.
    - 일반적으로 SQL 수행 빈도가 OLTP 시스템에서는 80% 이상 유지
    
- Parse CPU to Parse Elapsed %
    - parse를 수행하는 시간 중에 CPU를 사용하는 시간을 비율로 표현.
    - 파스 수행시간 = DB CPU 사용 시간 + 대기 시간 
    - 일반적으로 20% 이하일 경우 문제로 판단.
        - 대기 시간이 높을 경우 Library Hit(%) 값과 Soft Parse(%) 값을 점검
        - library cache 크기가 적당한지 hard parse가 과도하게 발생하는지에 대해 점검 필요.
        - Library Cache와 관련된 대기 이벤트의 발생 수치가 증가했는지도 점검.
    - SQL 수행 빈도가 높은 시스템
        - soft parse에 대한 부하도 더 낮추기 위해 SESSION_CACHED_CURSORS 파라미터 값을 높게 설정해서 
        - PGA영역에서 cursor(SQL hash value)를 캐싱하는 것도 좋은 방법.
    
- Redo Nowait %
    - 변경 사항을 redo log buffer에 기록하기 위해 기다리지 않는 비율
    - 99% 이상 유지
        - 낮을 경우 redo log buffer나 redo log file의 크기가 부족하거나
        - redo log file에 대한 disk I/O가 느리기 때문.
        - 불필요하게 자주 수행되는 commit도 문제가 될 수 있음.

- In-memory Sort %
    - 정렬 작업이 수행될 때 디스크가 아닌 메모리에서 정렬이 이루어진 비율
    - OLTP 업무에서 90% 이상을 권장.
        - 값이 낮다면 memory sort 크기 지정 파라미터 설정 값을 점검.
        - PGA 자동 관리 시 => PGA_AGGREGATE_TARGET 파라미터 확인
        - PGA 수동 관리 시 => SORT_AREA_SIZE 값을 확인.

- Soft Parse %
    - 전체 parse 중에서 soft parse 비율 
        - literal SQL이 많이 수행된다면 bind 변수를 사용하도록 변경해서 soft parse 비율을 높일 수 있음.
    - DW와 배치 전용 시스템에서 필요에 의해 hard parse가 아니면 90% 이상을 유지.

- Latch Hit % 
    - latch 획득 시도 시 대기 없이 바로 획득한 비율.
    - 값이 낮다면 Latch Sleep Breakdown 단위 보고서 참고.
        - Sleeps 칼럼과 Misses 칼럼의 수치가 높은 latch에 대해 튜닝을 수행해야 함.
        

- % Non-Parse CPU
    - parse에 소요된 CPU 시간을 제외한 나머지 시간의 비율
    - Non-Parse 값이 98% 이하면 과도한 parse로 인한 경합 의심
        - 경합 발생 시 Library Cache 관련 대기 이벤트 발생 수치도 함께 높아짐.
        - 이를 해결하려면 hard Parse를 수행하는 SQL을 찾아서 soft Parse를 수행할 수 있도록 변경.
        - 과도한 soft parse를 피하기 위해서 SESSION_CACHED_CURSORS 파라미터를 점검.

- Memory Usage %
    - shared pool 내에서 FREE공간을 제외한 실제 사용 중인 메모리 공간의 비율
    - 100%에 가까울수록 문제가 됨.
        - shared pool의 크기가 너무 작게 설정되거나 과도한 hard parse 수행으로 증가하게 됨.
    - 0%에 가까워도 문제가 됨.
        - shared pool 공간이 너무 과도하게 크게 설정된거라 효율성이 떨어진다고 볼 수 있음.
    - 75% ~ 80% 정도의 사용률을 유지

- % SQL with executions > 1
    - 1회만 수행된 SQL 수를 제외한 나머지 SQL의 수행 빈도.
    - 1회만 수행되는 literal SQL은 hard parsing의 주 원인.
    - 90% 이상 유지해야 shared pool을 효율적으로 사용하고 최적의 성능을 유지

- % Memory for SQL w/exec > 1 
    - 1회 이상 수행된 SQL들이 사용하는 shared pool 공간의 메모리 비율
    - % SQL with executions > 1과 연관관계가 높고 높을수록 좋다.

### Top 5 timed Events(현재 Top 10 Foreground Events by Total Wait Time)

- CPU 사용 시간을 포함한 값으로 각 이벤트별 소요 시간 기준으로 상위 10개를 보여줌.
- end_snapshot - begin_snapshot
- 오라클 성능 문제 시 가장 먼저 체크해야할 보고서

![top10](https://github.com/lght2000/mu.github.io/blob/master/assets/images/top10.PNG?raw=true)

| 항목                  | 의미                            | 관련컬럼                                      |
|-----------------------|---------------------------------|-----------------------------------------------|
| Event                 | 대기 이벤트 이름                | EVENT_NAME                                    |
| Waits                 | 대기 발생 횟수                  | TOTAL_WAITS                                   |
| Total Wait Time (sec) | 대기 발생 총 시간(단위:초)      | TIME_WAITED_MICRO / 1,000,000                 |
| Wait Avg(ms)          | 평균 대기 시간(단위 : 1/1000초) | (TIME_WAITED_MICRO / 1,000,000) / TOTAL_WAITS |
| % DB Time             | 이벤트별 대기 발생 총 시간 비율 | —                                             |
| Wait Class            | 대기 이벤트 클래스              | WAIT_CLASS                                    |

- DBA_HIST_SYSTEM_EVENT 딕셔너리 참조
- 관련 뷰 v$system_event

- DB 성능 표현 공식
    - 수행 시간 = CPU 사용 시간 + 대기 시간
    - 성능 저하 원인
        - CPU 경합으로 인한 처리 시간 지연
            - 대기 시간 < CPU 사용 시간 => 안정적인 사용으로 판단할 수 있음.
            - CPU 사용 시간이 문제가 되는 경우는 정상적인 경우보다 지연될 때
            - CPU 경합은 CPU 사용량과 밀접한 연관
                - CPU 사용량이 증가하면 CPU 실행 대기열 길이(CPU Run Queue Length)가 길어짐.
                - CPU 사용 경합이 발생하게 되고 CPU 처리 시간이 증가.
                - CPU 대기열 길이는 UNIX 명령으로 확인 가능.
                    - vmstat 1 10
                    - r이 CPU 대기열 수를 나타냄
                    - r 값을 CPU 개수로 나누면 평균 CPU 실행 대기열 수를 구할 수 있음.
                    - CPU가 4개라고 했을 때 r값/ 4 로 했을 때 0 or 0.25인 것을 알 수 있음.
                    - 보통 CPU 실행 대기열의 길이가 1.5 ~1.8 이상일 때부터 문제가 됨.(시스템에 따라 상이)
        
                 ```bash
                    vmstat 1 10
                    procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
                     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
                     0  1 1895544 3132444 157756 46029996    0    0  3618   420    0    0  7  1 91  2  0    
                     0  1 1895544 3127692 157764 46030044    0    0 22257    61 46605 13542  0  0 97  2  0  
                     0  1 1895544 3123228 157768 46030044    0    0 10257   155 46703 13966  0  0 97  3  0  
                     0  1 1895544 3122152 157776 46030048    0    0  8225    90 46734 14020  0  0 97  3  0  
                     0  1 1895544 3125624 157784 46030044    0    0 20313    39 47775 15009  0  0 97  3  0  
                     0  1 1895544 3123084 157784 46030056    0    0 11185    29 46733 13962  0  0 97  2  0  
                     1  1 1895544 3121184 157784 46030056    0    0 10161   151 47125 15025  1  1 95  3  0  
                     0  1 1895544 3121528 157796 46030044    0    0 10161   129 47093 14958  0  0 97  3  0  
                     0  2 1895544 3119188 157804 46030056    0    0  7057   150 47361 14973  0  0 97  3  0  
                     0  1 1895544 3117072 157804 46030056    0    0 11017   140 48114 15964  0  0 96  3  0  
        
                 ```
        
            - CPU 사용량 증가 시 CPU 경합이 발생하는 경우 해결 방법
                - OS 튜닝 => OS kernal이나 기타 파라미터 설정, 하드웨어 설정이 잘못된 경우 튜닝 필요
                - DB 튜닝 => 즉각적인 latch 획득 실패로 과도하게 spin이 발생하는 경우 CPU 급증(대기 시간을 줄여야 함.)
                - SQL 튜닝 => 악성 SQL 수행 시 CPU를 과도하게 사용
                - CPU 증설 => 위의 세 가지로 불가능할 경우 시스템 자원에 비해 발생하는 부하 자체가 높은 것으로 CPU 증설 필요

        - CPU가 아닌 다른 자원의 경합으로 대기 시간 증가
            - 대기 시간 증가로 성능이 저하될 경우 전체 이벤트 중 특정 소수의 대기 이벤트의 발생 비중이 매우 높게 나타남.
            - 그러므로 문제 대기 이벤트는 Top 10 Foreground Events by Total Wait Time 단위 보고서에 보여짐.
            - 위 보고서에서 CPU Time은 실제 DB가 CPU를 사용한 시간을 의미
                - CPU 경합이 발생하지 않는 이상 CPU Time이 높은 비중을 차지하는 것은 우려할 사항이 아님.
                - CPU 경합이 발생하지 않았으나, CPU Time 발생 비율이 대부분을 차지하면서 성능이 낮게 나타난다면
                    - 악성 SQL 수행이나 latch 경합 등과 같이 CPU를 비효율적으로 많이 사용하는 원인을 점검해야 함.
                    
     
    - db file sequential read
        - 디스크 I/O는 피할 수 없음. disk I/O 성능이 보장된다면 문제가 안됨.
        - 단일 block I/O의 평균 속도 5 ~ 20ms로 권장.
    
    - latch free => latch가 sleep 상태로 빠질 때 수치가 증가.
        - latch 경합이 발생할 때 나타남.
        - latch free 대기 이벤트 비중이 높은 경우 latch spin 현상으로 CPU를 불필요하게 많이 사용.
            - 비중이 높은 경우, latch 경합으로 성능에 문제가 있을 수 있으므로 튜닝해야 함.
        - AWR 보고서 참고(latch activity, latch sleep breakdown, latch miss sources, parent latch statistics 등)
    
    - db file scattered read
        - full table scan or index fast full scan 시 발생.
        - OLTP 시스템이라면 불필요한 full table scan이 발생하는지 체크
        
    - log file sync
        - redo log buffer의 내용을 redo log file에 내려쓸 때 발생하는 대기 이벤트
        - 평균 대기 시간이 높다면 disk I/O 속도 및 redo log buffer 크기를 점검
        - 평균 대기 시간은 낮은데 발생 횟수가 많다면 redo log buffer 크기 설정 및 commit 발생 횟수를 점검.
        
  
  
           
                
    





 





