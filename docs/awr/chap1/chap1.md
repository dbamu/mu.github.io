---
layout: default
title: chapter1
parent: AWR을 이용한 고성능 튜닝
nav_order: 1
---



### 데이터베이스 성능 분석 및 튜닝

- 데이스베이스 성능 문제는 **한 두가지 핵심 원인** 때문에 발생
- DB 성능 분석은 **정확한 원인**을 빠르게 찾는 것이 큰 목적
    - 인과 관계를 파악
    - 성능 분석에 필요한 데이터의 선별, 수집, 분석 능력 중요

### 데이터베이스 성능 분석 및 튜닝을 위한 데이터 수집 방법
- AWR 데이터 이용
- 오라클 metaData, Dynamic View, Dictionary 검색
- SQL Trace File 생성


### AWR 개념 
- 위의 목적을 위해 AWR(Automatic Workload Repository) 활용
- oracle 8i version부터 제공된 STATSPACK과 유사하나 수집 범위가 확장
- AWR 동작
    - 데이터베이스 생성 후 설치 없이 자동 수행
    - 메모리 모니터링 **background process(MMON, MMNL)**에 의해 데이터 수집 후 **SYSAUX** 테이블스페이스에 저장
    - snapshot interval은 default 1시간, 보관기간은 7일이고 수집 및 보관 주기의 변경 가능.
    - AWR 데이터 조회는 WRH$_*, WRM$_*, DBA_HIST_*를 통해 조회 가능.
    
    ![](https://github.com/lght2000/mu.github.io/blob/master/assets/images/Oracle-AWR-Report.png?raw=true){: width="700" height="500"  .center}

- AWR 수집 데이터 
    - 데이터베이스 대기 이벤트 정보 및 통계 정보
    - 그룹화된 DB 사용 부하의 통계 수치(DBA_HIST_SYSTEMETRIC_SUMMARY, DBA_HIST_SYS_TIME_MODEL)
    - SQL 수행 통계 정보 및 실행 계획
    
- AWR 데이터의 활용
    - DB 상태 파악
        - DB의 문제 발생 여부 및 운영 상태 파악
        - dynamic view, dictionary 정보로 실시간 DB 상태 파악 가능하지만 특정 구간 사이의 DB 상태 파악 불가
        - 시점 데이터를 저장하는 AWR을 이용해서 원하는 구간의 DB 상태 파악 가능.
    - 성능 문제 해결
        - AWR report 및 ASH 생성 스크립트 활용
    - 성능 추세 분석
        - 과거 데이터의 추세 분석을 통해 미래 예측
    
- statspack
    - Oracle 8.1.6이상에서 사용 가능
    - AWR이 나오기 전인 oracle 10g 이전에 사용됨.
    - 수집 정보 : DB 대기 이벤트 및 통계 정보, 시스템 통계 정보, 데이터베이스 부하 정보, SQL 수행 정보, active session\
    - 좀 더 자세한 차이는  [링크](http://wiki.gurubee.net/pages/viewpage.action?pageId=30965836) 참조.





