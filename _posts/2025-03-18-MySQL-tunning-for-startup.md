---
title: 스타트업을 위한 MySQL 튜닝 방법
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2025-03-18 22:55:00 +0900
categories: [DB, MySQL]
tags: [programming, db, mysql]
pin: false
math: false
---

## 들어가면서

근래에, 자주쓰던 NoSQL계열(mongodb, dynamodb, redis)이 아닌, RDB를 쓸일이 있었습니다.  
RDB를 production level로 올리고 나니, 개발환경과는 다르게, 몇가지 parameter를 세팅해야 ‘기대하는 기능’을 수행할 수 있었습니다.  
이 내용을 ‘기록해야겠다’는 생각이 들었습니다.  

### InnoDB의 특성 이해하기

> MySQL에서 일반적인 목적(General Purpose)로 많이 사용하는 Storage Engine인 ‘innoDB’를 기준으로 합니다.
{: .prompt-info }

**‘InnoDB’는**
: 트랜잭션을 실행할 때 **즉시 데이터를 디스크에 반영하지 않고, 먼저 로그(redo log)에 기록**합니다.
: **‘트랜잭션 수행 과정’**은 다음과 같습니다.
: 1. 변경된 데이터는 **Buffer Pool (메모리)에 먼저 저장**됩니다.
: 2. 트랜잭션을 COMMIT하면 **Redo Log에 기록**됩니다 (innodb_log_file).
: 3. 이후, **Lazy Flushing**을 통해 실제 데이터를 디스크의 데이터 파일(.ibd)에 기록.

> 이 과정 때문에, COMMIT이 되었더라도 데이터 파일에는 즉시 반영되지 않을 수 있습니다.
때문에, insert한 데이터를 바로 read했을때, 조회가 안되는 ‘phantom read’ 현상이 발생할 수 있습니다.
{: .prompt-info }

## 본문

### TCP Connection 재사용성 높이기

- `max_connections` 값 조정  
  MySQL에 최대로 동시접속할 수 있는 Connection 갯수를 조절합니다.  
  너무 낮으면, 새로운 TCP 커넥션을 만드는 일이 잦아지고, 너무 높으면 불필요한 리소스(TCP를 유지하는데에)를 계쏙 쓰게 됩니다.  
  [MySQL :: MySQL 8.4 Reference Manual :: 7.1.8 Server System Variables](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_max_connections)

- `wait_timeout` 과 `interactive_timeout` 값 조정  
  MySQL은 사용되지 않는 TCP connection을 자동으로 종료하는데, 이때 idle시간(사용하지 않는 시간)을 조정하여, Connection 재사용률을 조절합니다.  
    - `wait_timeout` : 일반 Client용 세팅값  
    - `interactive_timeout` : CLI 또는 DB 관리도구(GUI Client)용 세팅값  
  [MySQL :: MySQL 8.4 Reference Manual :: 7.1.8 Server System Variables](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_wait_timeout)

- `skip_name_resolve` 활성화  
  MySQL은 Client가 접속할 때, DNS조회를 수행하는데, IP기반으로 연결한다면 불필요 합니다.  
  [MySQL :: MySQL 8.4 Reference Manual :: 7.1.8 Server System Variables](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_skip_name_resolve)


### Transaction 반영 속도 높이기

- `innodb_flush_log_at_trx_commit = 2` 로 세팅해서, transaction이 빨리 반영하도록 합니다.  
  `2`로 설정하면, ‘강한 일관성’을 약간 포기하고(crash상황에서 transaction을 유실할 수 있습니다), transaction종료 후, log를 memory에 보관하고 사용합니다.(default인 1인 경우는 즉시 disk에 기록합니다.)  
  > Controls the balance between strict ACID(https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_acid)compliance for[commit](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_commit)operations and higher performance that is possible when commit-related I/O operations are rearranged and done in batches. You can achieve better performance by changing the default value but then you can lose transactions in a crash.

  [MySQL :: MySQL 8.4 Reference Manual :: 17.14 InnoDB Startup Options and System Variables](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit)

- `innodb_log_file_size` 값 조정  
  `innodb_log_file_size` 는 innodb의 ‘Redo log’의 크기를 결정합니다.  
  이 값을 더 크게 조정하면, transaction을 disk에 쓰기전에, ‘Redo log’에 저장되는 양이 많아지고, 메모리에 더 많은 내용을 저장하게 됩니다.  
  이 결과로, ‘[Checkpoint](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_checkpoint)’생성(memory에 저장된 변경사항을 disk에 쓰는 작업) 빈도가 줄어듭니다.

  [MySQL :: MySQL 8.4 Reference Manual :: 17.14 InnoDB Startup Options and System Variables](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_log_file_size)

    > 이 값을 반영하려면, DB재시작이 필요합니다.
    {: .prompt-info }

- `innodb_purge_threads` 값 조정  
  이 값은, innoDB의 MVCC(Multi-Version Concurrency Control, 하나의 record를 versioning하여 동시성을 제공하는 개념) 기능에서 따라오는 GC(Garbage Collection)성능을 개선하는 것과 관련된 parameter입니다.  
  ‘Undo Log(이전 버젼의 데이터를 저장하는 곳)’를 정리하는 Thread 수를 결정하며, 높을수록 빨리 정리됩니다.  
  ‘Undo Log’가 정리되지 않으면, ‘Buffer pool’에 데이터가 쌓이게 되어 transaction반영 속도를 느리게하는 원인이 됩니다.  
    > Transaction이 완료되면, 더 이상 필요 없는 ‘Undo Log’를 삭제하는 작업(purge)이 필요합니다.
    {: .prompt-info }

  [MySQL :: MySQL 8.4 Reference Manual :: 17.14 InnoDB Startup Options and System Variables](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html#sysvar_innodb_purge_threads)

    > 이 값을 반영하려면, DB재시작이 필요합니다.
    {: .prompt-info }


## References
Best practices for configuring parameters for Amazon RDS for MySQL  
: [Best practices for configuring parameters for Amazon RDS for MySQL, part 1: Parameters related to performance (Amazon Web Services)](https://aws.amazon.com/ko/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-1-parameters-related-to-performance/)

Designing Data Intensive Application(book)
: [Designing Data-Intensive Applications (DDIA) — an O’Reilly book by Martin Kleppmann (The Wild Boar Book)](https://dataintensive.net/)

MySQL의 storage engine인 ‘innodb’ introduction
: ‘[Key Advantages of InnoDB](https://dev.mysql.com/doc/refman/8.4/en/innodb-introduction.html)’부분을 봐두면 좋습니다.
: [MySQL :: MySQL 8.4 Reference Manual :: 17.1 Introduction to InnoDB](https://dev.mysql.com/doc/refman/8.4/en/innodb-introduction.html)

MySQL innodb Parameters
: [MySQL :: MySQL 8.4 Reference Manual :: 17.14 InnoDB Startup Options and System Variables](https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html)

MySQL Server System Parameters
: [MySQL :: MySQL 8.4 Reference Manual :: 7.1.8 Server System Variables](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html)

‘Phantom Rows’
: [MySQL :: MySQL 8.4 Reference Manual :: 17.7.4 Phantom Rows](https://dev.mysql.com/doc/refman/8.4/en/innodb-next-key-locking.html)
