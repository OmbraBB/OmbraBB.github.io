---
title: Sysbench Test
author: B
date: 2025-09-07 09:53:00 +0900
categories: [MariaDB]
---

- Sysbench 란?
    - CPU, 메모리, 디스크 I/O, 네트워크, 운영체제 스케줄러, Database(Mysql/MariaDB)와 같은 시스템의 성능을 측정하는 오픈소스 벤치마크 도구.
    - LuaJIT 기반의 스크립트 엔진을 사용하며 멀티스레드 방식을 지원한다.

- 테스트 환경:
    - MariaDB 11.4.5
    - CPU : 2core (VMware)
    - Memory : 4GB (VMware)
    - /data 용량 : 13GiB

```shell
# 1. 테스트 Database 'sbtest' 생성
MariaDB> CREATE DATABASE sbtest

# 2.sysbench prepare (테이블 생성 및 데이터 생성)
sysbench --tables=5 --talbe-size=2500000 --mysql-user=root --mysql-password=root --mysql-socket=/tmp/mysql.sock /usr/share/sysbenct/oltp_read_write.lua prepare

# 3. sysbench run (싶 퍼포먼스 측정)
sysbench --tables=5 --table-size=2500000 --mysql-user=root --mysql-password=2dnjf1dlf! --mysql-socket=/tmp/mysql.sock /usr/local/share/sysbench/oltp_read_write.lua --report-interval=1 --threads=24 --time=300 run

# 4. sysbench cleanup (테스트 데이터 및 테이블 삭제)
sysbench --tables=5 --table-size=2500000 --mysql-user=root --mysql-password=2dnjf1dlf! --mysql-socket=/tmp/mysql.sock /usr/local/share/sysvench/oltp_read_write.lua cleanup

# 5. sysbench run 퍼포먼스 측정 결과
SQL statistics:    -- (SQL 통계)
    queries performed:    -- (수행된 쿼리)
        read:                            2769844    -- SELECT 쿼리 실행 횟수 (읽기 작업)
        write:                           791384    -- INSERT/UPDATE/DELETE 쿼리 실행 횟수 (쓰기 작업)
        other:                           395692    -- COMMIT, BEGIN 등 트랜잭션 관련 쿼리
        total:                           3956920    -- 전체 쿼리 실행 횟수
    transactions:                        197846 (659.12 per sec.)    -- 총 197,846개의 트랜잭션을 완료했으며, 초당 659.12개의 트랜잭션 처리 (TPS)
    queries:                             3956920 (13182.49 per sec.)    -- 초당 13,182.49 개의 쿼리 처리 (QPS)
    ignored errors:                      0      (0.00 per sec.)    -- 무시된 오류 없음 (안정적인 실행)
    reconnects:                          0      (0.00 per sec.)    -- 데이터베이스 재연결이 없음 (연결 안정성 양호)

Throughput:    -- (처리량)
    events/s (eps):                      659.1247    -- 초당 이벤트(트랜잭션) 처리량
    time elapsed:                        300.1647s    -- 실제 테스트 실행 시간 (약 5분)
    total number of events:              197846    -- 총 처리된 이벤트(트랜잭션) 수

Latency (ms):    -- (지연시간) - 밀리초 단위
         min:                                    1.40    -- 가장 빠른 트랜잭션 응답시간
         avg:                                   18.20    -- 평균 응답 시간 (상당히 양호한 수준)
         max:                                 3863.13    -- 가장 느린 트랜잭션 응답시간 (약 3.9초, 일시적 지연 발생)
         95th percentile:                       34.95    -- 95%의 트랜잭션이 34.95ms 이내에 완료(대부분의 요청이 빠르게 처리됨)
         sum:                              3601378.22    -- 모든 트랜잭션의 총 소요시간

Threads fairness:    -- (스레드 공정성)
    events (avg/stddev):           16487.1667/93.02    -- 스레드당 평균 16,487개 이벤트 처리, 표준편차 93.02 | 매우 낮은 표준편차로 12개 스레드 간 작업량이 균등하게 분배됨
    execution time (avg/stddev):   300.1149/0.02    -- 스레드별 평균 실행시간 300.11초, 표준편차 0.02초 | 모든 스레드가 거의 동일한 시간 동안 작업 수행
```

- 최종 성능 평가
1. 우수한 처리량 : TPS 659, QPS 13,182
2. 안정적인 응답시간 : 평균 18.2ms, 95%가 35ms 이내
3. 균등한 부하분산 : 12개 스레드가 고르게 작업 처리
4. 높은 안정성 : 오류나 재연결 없음
    - 다만 최대 응답시간이 3.8초로 긴 편인데, 이는 일시적인 I/O 대기나 잠금 경합으로 인한 것으로 보임.
    - 지연 부분의 min/max 의 차이가 클 수록 요청 처리 속도의 편차가 큼.