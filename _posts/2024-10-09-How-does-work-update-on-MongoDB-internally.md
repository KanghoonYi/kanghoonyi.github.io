---
title: How does work 'update' on MongoDB internally?
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-10-06 02:29:00 +0900
categories: [Books, Designing Data-Intensive Applications]
tags: [Architecture, MongoDB, DocumentDB]
pin: false
math: false
---

책 ‘Designing Data-Intensive Applications’을 보면서, 궁금한것들이 많아집니다.  
이 책이 2017년 초판이 인쇄 되었는데, 제가 쓰는 기술들에 실제로 적용 되는지 궁금해집니다.  
  
이 글은 그중 하나인, **‘Mongodb(Document db)에서 update는 disk의 rewrite를 발생시키는가?’**에 대한 글 입니다.

## Overview

Mongodb와 같은 Document DB는 별도의 schema가 없습니다(정확하게는, write할때 schema가 없습니다). 이는 ‘Relational Database(이하 RDB)’와 같이 정해진 schema가 있는 형태와 가장 큰 차이점 입니다  
때문에, Document를 update할 때, 고려해야할 문제가 한가지 더 생깁니다.  

**‘Update한 document의 size가 original보다 크다면?’**  
기존에 차지하던 disk공간에 update된 document를 넣을 순 없습니다. 크기가 다르니까요.  
여기서는 ‘Mongodb의 경우 어떻게 처리되는지’ 확인한 내용을 정리하여 다루고자 합니다.

## Default StorageEngine인 Wiredtiger 이해하기

[WiredTiger](https://www.mongodb.com/ko-kr/docs/manual/core/wiredtiger/)는 Mongodb의 default storage engine입니다.  
Mongodb에서 ‘write’는 ‘**Checkpoint**’와 ‘Journaling’을 기반으로 작동합니다.

### [Snapshot and Checkpoint](https://www.mongodb.com/docs/manual/core/wiredtiger/#snapshots-and-checkpoints)

WiredTiger는 특정 record에 대한 write작업 시작시, 데이터에 대한 스냅샷(snapshot)을 만듭니다.  
이 ‘snapshot’에 대하여 write작업을 적용하고, Disk에 쓰기작업까지 모두 완료되면, 이 변경사항을 기반으로하는 ‘Checkpoint’를 생성합니다.

[WiredTiger: Snapshot](https://source.wiredtiger.com/develop/arch-snapshot.html#snapshot_transaction)(WiredTiger에서 snapshot을 어떤 개념으로 사용하고 있는지 알 수 있는 페이지입니다.)

### [Journaling](https://www.mongodb.com/docs/manual/core/journaling/)

‘Journaling’은 ‘기록을 남기는것’을 의미합니다.  
DB에서의 ‘Journaling’은 Data에 변화(delta값)를 기록하는것을 말합니다.  
‘Checkpoint’사이의 데이터 변화를 기록합니다.  
이때 장애가 발생하면, 이 ‘journal record’를 기준으로 복구합니다.

> 이러한 system을 ‘[Journaling file system](https://en.wikipedia.org/wiki/Journaling_file_system)’이라고 합니다
{: .prompt-tip }

[Journaling Process](https://www.mongodb.com/ko-kr/docs/manual/core/journaling/#journaling-process)에 따라 작동합니다.  
  
**Client가 DB에 대한 ‘write’를 발생시키면,**

1. 가장 최신의 ‘Checkpoint’를 기준으로 data의 ‘snapshot(in-memory)’을 생성합니다.
2. 해당 작업에 대한 ‘journal record’를 생성합니다.

   이후 추가되는 write작업이 ‘journal record’에 buffering 됩니다(쌓입니다).

3. 이후 특정 record에 대한 update가 발생하면, snapshot에 반영합니다.
   이를 ‘journal record’에 기록합니다.

   > 특정 주기나 특정 조건을 만족하면, 이 ‘journal record’를 Disk에 씁니다(’journal commit’).  
   > 이 data는 장애시 복구를 위해 사용합니다.
   {: .prompt-tip }

4. 특정 주기나 조건을 만족하면, snapshot을 Disk에 작성합니다.
5. snapshot이 반영된 새로운 ‘Checkpoint’를 생성합니다.
6. 완료.

이런 과정을 반복하면서, Disk에 여러 ‘Checkpoint’가 생기게 됩니다.

### Compaction

‘Checkpoint’는 여러 disk block을 참조합니다.  
> Each checkpoint references a certain set of disk blocks for a table.

‘Checkpoint’가 쌓이면서, disk block또한 늘어나게 됩니다. 이에따라, 이제 사용하지 않는 old block을 정리하고, table을 탐색하는데 사용하는, [B-Tree](https://source.wiredtiger.com/11.0.0/arch-btree.html)를 새롭게 구축하게 됩니다.  
이 과정을 ‘**Compaction**’이라고 합니다.

## Conclusion(결론)

Data를 Update하는 경우, 항상 **disk에 Document전체 데이터가 새로 써집니다**. 기존의 block을 copy하여, update를 적용한 새로운 Block을 생성합니다.  
이렇게 생성된 새로운 Block은 Table을 구성하는 ‘B-Tree’에 있던 기존의 Block Reference를 대체하게 됩니다.  
이 ‘B-Tree’를 기반으로 새로운 ‘Checkpoint’를 생성하게 됩니다.

## References

WiredTiger Storage Engine
: [WiredTiger Storage Engine](https://www.mongodb.com/docs/manual/core/wiredtiger/)

Mongodb에서의 Write Operation
: [Write Operation Performance](https://www.mongodb.com/docs/manual/core/write-performance/#journaling)

Journaling and the WiredTiger Storage Engine
: [Journaling](https://www.mongodb.com/docs/manual/core/journaling/#journaling-and-the-wiredtiger-storage-engine)

journal
: [Glossary](https://www.mongodb.com/docs/manual/reference/glossary/#std-term-journal)

[Issue] Add interface allowing partial updates to existing values
: [](https://jira.mongodb.org/browse/WT-2972)

Amazon DocumentDB vs MongoDB 의 내부 아키텍쳐 와 장단점 비교
: [Amazon DocumentDB vs MongoDB 의 내부 아키텍쳐 와 장단점 비교](https://www.slideshare.net/awskorea/t1s2pdf#11)

WiredTiger의 Transaction using Snapshots
: [WiredTiger: Snapshot](https://source.wiredtiger.com/develop/arch-snapshot.html#snapshot_transaction)

WiredTiger의 B-Tree
: [WiredTiger: B-Trees](https://source.wiredtiger.com/11.0.0/arch-btree.html)

WiredTiger의 Compaction
: [WiredTiger: Compaction](https://source.wiredtiger.com/11.0.0/arch-compact.html)
