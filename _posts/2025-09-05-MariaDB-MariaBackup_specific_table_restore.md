---
title: MariaBackup_specific_table_restore
author: B
date: 2025-09-05 17:50:00 +0900
categories: [MariaDB]
---
- 테스트 개요 :
    - mariabackup으로 백업 한 테이블에서 특정 테이블 하나만 복구가 가능할까?

- 테스트 환경 :
    - MariaDB 11.4.5 / VMware 사용

```sql
-- 테스트 테이블 생성
MariaDB > CREATE DATABASE test;
MariaDB > use test;
MariaDB > show tables;
Empty set (0.001 sec)

MariaDB > CREATE TABEL restore_test (
    -> i1 int not null primary key,
    -> i2 int not null );

MariaDB > insert into restore_test values (1,1),(2,2),(3,3),(4,4),(5,5);

-- 백업 진행
# mariabackup --decomporess --remove-original --target-dir=./2025-08-19_17-30-37/

-- 테스트 데이터 지우기
MariaDB > truncate table test.restore_test;

# ll /data/db/test
total 72
-rw-rw---- 1 mysql mysql    67 Aug 19 17:10 db.opt
-rw-rw---- 1 mysql mysql   949 Aug 19 17:11 restore_test.frm
-rw-rw---- 1 mysql mysql 65536 Aug 19 17:34 restore_test.ibd

# yum install qpress
# mariabackup --decompress --remove-original --target-dir=./2025-08-19_17-30-37/
# mariabackup --prepare --export --target-dir=./2025-08-19_17-30-37/

MariaDB [test]> ALTER TABLE test.restore_test DISCARD TABLESPACE;
# ll /data/db/test
total 8
-rw-rw---- 1 mysql mysql  67 Aug 19 17:10 db.opt
-rw-rw---- 1 mysql mysql 949 Aug 19 17:11 restore_test.frm

# cp ./2025-08-19_17-30-37/test/restore_test.cfg /data/db/test
# cp ./2025-08-19_17-30-37/test/restore_test.ibd /data/db/test
# chown mysql:mysql /data/db/test/restore_test.*
# ll /data/db/test
total 76
-rw-rw---- 1 mysql mysql    67 Aug 19 17:10 db.opt
-rw-r----- 1 mysql mysql   402 Aug 19 17:36 restore_test.cfg
-rw-rw---- 1 mysql mysql   949 Aug 19 17:11 restore_test.frm
-rw-r--r-- 1 mysql mysql 65536 Aug 19 17:36 restore_test.ibd

MariaDB [test]> ALTER TABLE test.restore_test IMPORT TABLESPACE;
Query OK, 0 rows affected (0.051 sec)

MariaDB [test]> select * from test.restore_test;
+----+----+
| i1 | i2 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
5 rows in set (0.037 sec)
```
- 테스트 결론 :
    - Mariabackup 사용 시, 특정 테이블만 복구를 하기 위해서는 전체적으로 복원과 prepare를 진행해야하는건 마찬가지이다.
    - prepare 진행 시, export 옵션을 사용하여 생성된 .cfg 파일을 사용하여 부분 백업을 복원이 가능하다.
        - .cfg 파일은 각 테이블의 메타데이터를 담고있는 파일이며, 테이블 스페이스(.ibd) 파일과 MySQL/MariaDB 데이터 딕셔너리 (InnoDB 내부 메타데이터)를 연결해주기 위한 매개체이다.
        - 즉, "이 .ibd 파일이 어떤 테이블의 어느 구조와 매칭되어야 하는지"를 알려주는 역할을 하는 파일.
        - .cfg 파일 안에 들어있는 내용은 테이블의 스키마 정의, 인덱스 정보, 필드 구성 같은 메타데이터를 담고있으며, `ALTER TABLE ... DISCARD TABLE SPACE` / `IMPORT TABLESPACE` 절차에서 .cfg + .ibd 를 같이 써야 정상적으로 테이블을 복구할 수 있다.