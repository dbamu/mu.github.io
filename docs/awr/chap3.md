---
layout: default
title: chapter3
parent: AWR을 이용한 고성능 튜닝
nav_order: 3
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





