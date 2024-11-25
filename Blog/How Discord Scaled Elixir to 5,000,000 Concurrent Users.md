> Link: https://discord.com/blog/how-discord-scaled-elixir-to-5-000-000-concurrent-users

- Discord는 실시간 통신 시스템을 구축하기 위해 Elixir를 초기부터 도입
- Erlang VM의 높은 동시성 처리 능력과 Elixir의 현대적인 언어 특성 활용, Discord는 500만 명의 동시 사용자를 지원하는 인프라를 구축
## Message Fanout
- 디스코드의 다양한 기능은 `pub/sub`에 집중됨
- 사용자가 `WebSocket`을 통해 세션 프로세스 생성 → 원격 Erlang 노드의 길드(서버)프로세스와 통신
- 사용자 증가에 따라 메시지 전송 지연 발생
	- Erlang 프로세스 간 메시지 전송 비용이 예상보다 저렴하지 않음
	- 프로세스 스케줄링에 사용되는 Erlang 작업 단위인 감소 비용도 상당히 높음
- Erlang 프로세스는 사실상 단일 스레드로 동작 → 작업을 병렬화를 위한 샤딩만이 답인지 고민
- 프로세스 분산이 답이다?
	- → 이벤트에 선형화 기능에 의존하고, pub schedule이 서로 다른 시간에 진행될 가능성
	- → 작업량 증가에 따른 확장성 감소
- `Manifold` 라이브러리 개발
	- 메시지 전송 작업을 PID의 원격 노드에 분산 및 PID 그루핑
## Fast Access Shared Data
- 일관된 해싱을 통한 분산 시스템을 구현 및 노드 조회를 위한 **링 데이터 구조**를 사용
- 초기에는 Erlang의 C 포트를 활용 → 사용자 재접속 시 병목 현상이 발생
	- 개선을 위해 ETS 활용 데이터 공유, FastGlobal 라이브러리를 개발하여 읽기 전용 공유 힙에 데이터 저장

## Limited Concurrency
- 세션 프로세스가 길드 레지스트리에 과도한 요청을 보내면서 시스템 과부하가 발생
- 이를 방지하기 위해 ETS의 원자적 카운터 기능을 활용하여 동시 요청 수를 제한 및, 세션 프로세스가 실패 가능성이 높은 요청을 사전 차단