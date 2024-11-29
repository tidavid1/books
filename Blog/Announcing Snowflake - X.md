> Link: https://blog.x.com/engineering/en_us/a/2010/announcing-snowflake

## The Problem
- 데이터 저장을 위해 MySQL 사용 → 데이터 증가에 따라 Cassandra 혹은 수평 샤딩된 MySQL로 대체중
- Cassandra는 고유 ID 생성 기능이 없음 → ID에 대한 획일적인 솔루션 필요
	- 고가용성을 보장하면서 초당 수만 개의 ID를 생성할 수 있는 솔루션 필요
	- 트윗을 정렬하기 위한 대략적인 정렬이 가능한 ID
	- 64비트 ID
## Options
- MySQL 기반 티켓 서버
	- 재동기화 루틴을 구현하지 않으면 순서 보장이 불가능함
- UUID
	- 128비트로 구성되어 있어 64비트 ID로 사용할 수 없음
- Zookeeper 순차 노드
	- 가용성 문제
## Solution
- Timestamp, Worker Number, Sequence Number로 구성된 64 bit ID 구성
	- Sequence Number는 스레드 단위로 zookeeper에 의해 선택