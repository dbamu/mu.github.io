---
layout: default
title: chapter2
parent: AWR을 이용한 고성능 튜닝
nav_order: 2
---

### AWR 스냅샷 관리

- AWR 스냅샷 : 메모리에서 디스크로 저장하는 시점의 AWR 데이터를 의미하며 디스크에 저장
- AWR 스냅샷 관리를 위해 DBMS_WORKLOAD_REPOSITRY 패키지 제공

| 항목                 | 사용법                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AWR 스냅샷 생성      | BEGIN DBMS_WORKLOAD.REPOSITORY.CREATE_SNAPSHOT([FLUSH_LEVEL=>'flush_level']); END; /  <br><br><br>dbms_workload_repository.create_snapshot(flush_level IN VARCHAR2 DEFAULT 'TYPICAL') RETURN NUMBER;                                                                                                                                                                                                                                                                                                                                          |
| AWR 스냅샷 삭제      | BEGIN DBMS_WORKLOAD.REPOSITORY.DROP_SNAPSHOT_RANGE(LOW_SNAP_ID=>snap_id, HIGH_SNAP_ID=>snap_id [, DBID->dbid]); END; /  <br><br>dbms_workload_repository.drop_snapshot_Range(low_snap_id  IN NUMBER, high_snap_id IN NUMBER, dbid IN NUMBER DEFAULT NULL);                                                                                                                                                                                                                                                                                    |
| AWR 스냅샷 설정 변겅 | BEGIN DBMS_WORKLOAD.REPOSITORY.MODIFY_SNAPSHOT_SETTINGS([RETENTION=>retention_time][, INTERVAL=>interval_time][, TOPNSQL=>topn_sql_number]); END; /  <br><br>dbms_workload_repository.modify_snapshot_settings(retention IN NUMBER DEFAULT NULL, interval IN NUMBER DEFAULT NULL, topnsql IN NUMBER DEFAULT NULL, dbid IN NUMBER DEFAULT NULL); <br> |
| 베이스라인 생성      | BEGIN DBMS_WORKLOAD.REPOSITORY.CREATE_BASELINE(START_SNAP_ID=>snap_id, END_SNAP_ID=>snap_id, BASELINE_NAME=>'baseline_name'[, DBID=>dbid]); END; /  <br><br>dbms_workload_repository.create_baseline(start_snap_id IN NUMBER, end_snap_id IN NUMBER, baseline_name IN VARCHAR2, dbid IN NUMBER DEFAULT NULL) RETURN NUMBER;                                                                                                                                                                                                                   |
| 베이스라인 삭제      | BEGIN DBMS_WORKLOAD.REPOSITORY.DROP_BASELINE(BASELINE_NAME=>'baseline_name'[, CASCADE=>true\|false][, DBID=>dbid]); END; /  <br><br>dbms_workload_repository.drop_baseline( baseline_name IN VARCHAR2, cascade IN BOOLEAN DEFAULT FALSE, dbid IN NUMBER  DEFAULT NULL);  <br><br>Cascade False : Drop baseline but not snapshots <br>True : Drops baseline and snapshots                                                                                                                                                                      |

<br>

### AWR 스냅샷 생성
- 원하는 시점에 AWR 스냅샷을 수행해야 할 경우

```sql
-- TYPICAL 옵션으로 AWR 데이터 수집
EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT();

-- ALL 옵션으로 AWR 데이터 수집
-- flush_level : AWR 스냅샷 수행 시 통계 레벨을 설정(ALL or TYPICAL) 
EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT(FLUSH_LEVEL=>'ALL');
```

- flush_level의 ALL과 TYPICAL의 의미는 statistics_level의 ALL과 TYPICAL과 같다.
- STATISTICS_LEVEL 초기화 파라미터
    - 오라클 10g부터 새로 추가.
    - AWR 데이터 수집 레벨을 설정
    - TYPICAL(default) : AWR 기본 통계 데이터 수집, AWR 데이터를 이용하는 대부분의 기능을 무리없이 사용.
    - ALL : AWR 기본 통계 데이터 + 추가 OS 통계 + 추가 실행 계획 통계 수집, 추가 테이터의 검색이 가능하나 저장 공간 및 수집 부하가 늘어난다.
    - BASIC : 통계 수집하지 않음
        - 10g의 새로운 기능 사용 불가.
            - AWR 스냅샷 자동 수행
            - 자가 진단(Automatic Database Diagnostic Monitor, ADDM)
            - 공유 메모리 자동 관리(Automatic Shared Memory Management, ASMM)
            - 자동 옵티마이저 통계 수집
            - MTTR 권고치(MTTR Advisor)
            
- 제세한 내용은 [오라클docs_statistics_level](https://docs.oracle.com/cd/B28359_01/server.111/b28320/initparams240.htm#REFRN10214) 를 참고


- **AWR 스냅샷 정보 검색 스크립트**
    - [DBA_HIST_SNAPSHOT](https://docs.oracle.com/cd/E11882_01/server.112/e40402/statviews_4045.htm#REFRN23442) 활용
    - 에러 발생 시 [DBA_HIST_SNAP_ERROR](https://docs.oracle.com/cd/B28359_01/server.111/b28320/statviews_4040.htm#REFRN23478) 활용

```sql
SELECT SNAP_ID, INSTANCE_NUMBER,
       TO_CHAR(STARTUP_TIME, 'YYYYMMDD HH24:MI') INST_START,
       TO_CHAR(BEGIN_INTERVAL_TIME, 'YYYYMMDD HH24:MI') BEGIN,
       TO_CHAR(END_INTERVAL_TIME, 'YYYYMMDD HH24:MI') END,
       SNAP_LEVEL
FROM   DBA_HIST_SNAPSHOT
ORDER BY INSTANCE_NUMBER, SNAP_ID;
```


| 항목                | 사용법                                                                                        |
|---------------------|-----------------------------------------------------------------------------------------------|
| SNAP_ID             | AWR 스냅샷 번호                                                                               |
| INSTANCE_NUMBER     | AWR 스냅샷이 수행된 인스턴스 번호                                                             |
| STARTUP_TIME        | 인스턴스가 시작된 시간                                                                        |
| BEGIN_INTERVAL_TIME | 이전 스냅샷이 수행된 시간. 인스턴스가 시작된 후 처음 스냅샷이 수행되었다면 인스턴스 시작 시간 |
| END_INTERVAL_TIME   | AWR 스냅샷 수행 시간                                                                          |
| SNAP_LEVEL          | AWR 스냅샷 수행시 설정된 FLUSH_LEVEL(TYPICLA:1, ALL:2)                                        |

<br>

### AWR 스냅샷 삭제
- AWR 스냅샷 데이터는 SYSAUX 테이블스페이스에 저장
- RAC 환경의 경우, drop_snapshot_range 프로시서 수행 시 모든 instance에서 지정한 구간의 스냅샷이 삭제되므로 주의!

```sql

BEGIN
    DBMS_WORKLOAD.REPOSITORY.DROP_SNAPSHOT_RANGE
    (LOW_SNAP_ID=>1, HIGH_SNAP_ID=>10);
END;
/

```

| 옵션         | 설명                                                                                   |
|--------------|----------------------------------------------------------------------------------------|
| LOW_SNAP_ID  | 삭제하려는 AWR 스냅샷의 시작 번호                                                      |
| HIGH_SNAP_ID | 삭제하려는 AWR 스냅샷의 마지막 번호                                                    |
| DBID         | 데이터베이스 ID 지정. 기본값은 NULL, 이 경우 프로시저를 수행한 데이터베이스가 선택된다 |

<br>

### AWR 스냅샷 설정 변경
- AWR 설정 기본 값
  - AWR 스냅샷 수행 주기 - 60분
  - AWR 스냅샷 보관 주기 - 7일
  - 항목별 수집 SQL 수 - 상위 30개

- 기본 값 설정 변경

```sql

BEGIN
    DBMS_WORKLOAD.REPOSITORY.MODIFY_SNAPSHOT_SETTINGS
    (INTERVAL=>30, RETENTION=>60*24*60, TOPNSQL=>'100');
END;
/
-- 수행 주기 30분, 보관 기간 60일, 수집되는 상위 SQL수 100개로 변경
SELECT * FROM DBA_HIST_WR_CONTROL;

```

| 옵션      | 설명                                                                                                                                                                                                                                                                                    |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| INTERVAL  | 스냅샷 수행 주기를 분 단위로 지정. <br>default => NULL, NULL인 경우 기존 설정 값이 유지<br>NUMBER 타입 변수<br>최소10분 최대 1년까지 지정 가능<br>0으로 설정 => AWR 스냅샷이 자동으로 수행되지 않음.<br>0으로 설정 => CREATE_SNAPSHOT() 프로시저를 이용한 수동 스냅샷도 수행할 수 없음. |
| RETENTION | AWR 스냅샷 보관 기간을 분단위로 지정. <br>default => NULL, 이 경우 기존 설정 값이 유지 <br>NUMBER 타입 변수<br>최소 1일 ~ 최대 100년까지 지정 가능.<br>0으로 설정 => 영구적으로 보존                                                                                                    |
| TOPNSQL   | Elapse 시간, CPU 사용 시간, Parse Call 횟수, Sharable Memory, Version Count의 항목에 대해 각각 수집되는 SQL문의 수를 지정.<br>각 항목의 크기를 내림차순 정렬해서 설정한 개수만큼 저장.<br>NUMBER타입 및 VARCHAR2 타입 변수<br>default => NULL. NULL인 경우 기존 설정값 유지             |
| TOOPNSQL 변수타입  | 설정 값                                                                                                                                                                                                                                                                                 |
| NUMBER    | 최소 30개에서 최대 100,000,000개까지 지정 가능하다.                                                                                                                                                                                                                                     |
| VARCHAR2  | DEFAULT:기본 값으로 설정 <br>AWR 통계 수집 레벨이 TYPICAL => 30개, ALL => 100개의 SQL이 저장된다.<br>MAXIMUM:커서 캐시에 저장된 모든 SQL을 저장한다.<br>숫자:NUMBER 타입으로 지정한 것과 동일하게 사용된다.                                                                             |
| DBID      | 데이터베이스 ID. <br>default => NULL. <br>NULL인 경우 프로시저를 수행한 데이터베이스가 선택                                                                                                                                                                                             |

<br>

### AWR 베이스라인 생성과 삭제

- baseline이란?
    - 튜닝 전후 비교나 정상 구간과 문제 구간 비교 등 여러 가지 이유로 설정하는데 이 기준 구간을 baseline이라고 함.

```sql

BEGIN
    DBMS_WORKLOAD.REPOSITORY.CREATE_BASELINE
    (START_SNAP_ID=>2004, END_SNAP_ID=>2006, BASELINE_NAME=>'BAD_BATCH', DBID=>1134139816);
END;

-- 생성한 baseline 정보 확인
SELECT * FROM DBA_HIST_BASELINE
/

```
- snapshot 2004 ~ 2006 사이에 문제가 발생했고 향후 해당 구간의 문제를 튜닝 후 문제가 되었던 시점과 비교하기 위해 베이스 라인을 생성.

| 옵션          | 설명                                                                      |
|---------------|---------------------------------------------------------------------------|
| START_SNAP_ID | 기준 값 구간으로 지정할 시작 스냅샷 ID. NUMBER type |
| END_SNAP_ID   | 기준 값 구간으로 지정할 종료 스냅샷 ID. NUMBER type |
| BASELINE_NAME | 베이스라인 이름 지정. VARCHAR2 type |
| DBID          | 데이터베이스 ID. default => 프로시저를 수행한 데이터베이스 ID  |

- baseline(기준 구간)과 튜닝 후 성능(비교 구간) 비교는 **awrddrpt.sql**, **awrdrpti.sql** 를 사용.
- baseline으로 설정된 스냅샷은 AWR RETENTION 기간이 지나도 삭제되지 않음.

<br>

### baseline 삭제

```sql

BEGIN
    DBMS_WORKLOAD.REPOSITORY.DROP_BASELINE
    (BASELINE_NAME=>'BAD_BATCH', CASCADE=>true);
END;
/

```


| 옵션          | 설명                                                                                                                                          |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| BASELINE_NAME | 베이스라인 이름 지정. VARCHAR2 type.                                                                                                          |
| CASCADE       | 삭제할 baseline의 snapshot 구간 삭제 여부 지정. <br>BOOLEAN type<br>TRUE 설정 => 베이스라인에 해당하는 스냅샷도 같이 삭제<br>default => false |
| DBID          | 데이터베이스 ID. default => 프로시저를 수행한 데이터베이스 ID                                                                                 |

<br>

### AWR 데이터 백업 및 복구
- 백업 관련 스크립트 사용 내용은 [링크](http://wiki.gurubee.net/pages/viewpage.action?pageId=26742263) 참조
- 오라클에서 제공하는 스크립트를 이용하면 data pump를 사용해서 AWR 데이터의 백업 및 복구 수행 가능.

- awrextr.sql : 데이터 펌프를 이용해서 AWR 데이터를 덤프 파일로 저장할 수 있음.(@?/rdbms/admin/awrextr.sql)
    - 스크립트 수행 전 data pump를 사용하기 위한 디렉토리 및 권한 필요!!
        - data pump 사용 => oracle directory 지정
        - OS 파일 시스템 디렉토리와 오라클을 연결시키는 역할
        - directory를 만들기 위해서 **CREATE ANY DIRECTORY** 권한, 삭제를 위해서는 **DROP ANY DIRECTORY** 권한 필요
        - 오라클 OS 유저가 지정된 OS 디렉토리에 **read write** 권한 필요
        
        ```sql
        
        create or replace directory 생성할디렉토리명 as 'OS의 디렉토리경로';
        
        -- 디렉토리를 생성한 계정에서만 사용 가능.
        grant read, write on directory 디렉토리명 
        
        -- 생성된 오라클 디렉토리를 공용 디렉토리로 사용하기 위해 public 유저에게 권한을 부여
        grant read, write on directory 디렉토리명 to public;
        
        drop directory 디렉토리명;
        
        ```
        
- awrload.sql : 백업 받은 데이터 dump file을 적재하는 스크립트(@?/rdbms/admin/awrload.sql)
    - 백업된 AWR 데이터가 저장되어 있던 DB ID와 동일한 DB ID를 가지는 DB에 적재할 수 없음.
    ![](https://github.com/lght2000/mu.github.io/blob/master/assets/images/awr%EB%B0%B1%EC%97%85.PNG?raw=true){: width="600" height="500"  .center}




