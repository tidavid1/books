> Link: https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/

- **Ticket Server**는 분산 설정에서 PK 역할을 하는 unique integer를 전역 제공
## Why?
- Flickr는 샤딩을 활용해 데이터베이스 확장
	- 하나의 큰 DB에 데이터를 모두 저장하지 않고, 데이터 분산 저장을 통한 부하 분산
- 가끔 데이터베이스간 데이터 마이그래이션을 고려해야함 → PK는 전역적으로 고유해야함
- MySQL 샤드는 master-master 레플리카 페어로 구성 → 복원력을 위해서?
	- Flickr의 의도적인 구성
	- 시스템 안정성과 가용성을 높이기 위한 전략으로 추정
- MySQL의 `auto-incrementing`은 논리적 DB에서의 고유성 보장 불가

## GUIDs?
- 그렇다면 왜 GUID를 쓰지 않았을까?
- 크기와 index 문제
	- `UUID` 인덱싱을 쉽게 지원해주는 `UUID_TO_BIN()`의 경우 MySQL 8.0에 도입 → UUID를 CHAR(36)으로 저장해야 되네?
- 쿼리에 사용하는 모든 값들에 대한 인덱싱 → 오직 Index만 활용해 쿼리 성능을 향상시켜옴
- ID를 활용한 순차성 보장과 쉬운 디버깅 + 캐시 해킹이 가능함
	- 캐시 효율성을 높인 부분을 캐시 해킹으로 표현함

## Consistent Hashing?
- AWS Dynamo 등 일부 DB는 해시 링을 활용해 GUID/sharding 이슈를 해결함
	- 쓰기 비용이 저렴한 경우에 적합
- MySQL은 빠른 읽기에 최적화 되어 있어요..

## Centralizing Auto-Increments
- 하나의 DB에서 `auto-increments`를 활용해 ID를 만든 후 모든 DB가 이 값을 PK로 사용하는 방식
- 근데 부하가 갑자기 많이 들어가면 어떡해요?

## Replace Into
- MySQL 자체 기능중 하나인 `REPLACE INTO` 활용
	- `REPLACE`: 테이블의 이전 row가 PK 또는 Unique 인덱스의 새 row와 동일한 값을 갖는 경우 새 row가 삽입되기 전 이전 row 삭제
	- 나머지는 모두 `INSERT`와 동일하게 동작

## Putting It All Together
- Flickr Ticket Server는 단일 Database를 갖는 전용 서버
	- `Tickets32`, `Tickets64` 와 같은 ID 비트수에 따른 테이블 존재
- 테이블 스키마
	```sql
	CREATE TABLE `Tickets64` (
	  `id` bigint(20) unsigned NOT NULL auto_increment,
	  `stub` char(1) NOT NULL default '',
	  PRIMARY KEY  (`id`),
	  UNIQUE KEY `stub` (`stub`)
	) ENGINE=InnoDB
	```
- ID 발행 sql 구문
	```sql
	REPLACE INTO Tickets64 (stub) VALUES ('a');
	SELECT LAST_INSERT_ID();
	```

## SPOFs
- 그렇다면 티켓 서버는 SPOF가 되는거 아닌가요?
- 두 대의 티켓 서버를 실행해 고가용성 보장
	- ID 증가를 홀수, 짝수로 구분해 책임 분담
	- 두 서버에 대한 라운드 로빈을 활용한 로드 벨런싱