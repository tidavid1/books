> Link: https://bravenewgeek.com/you-cannot-have-exactly-once-delivery/

### 분산 시스템의 한계
- 분산 시스템에서는 정확히 한 번 메시지를 전달하는 **Exactly-Once Delivery**는 불가능
- "At-most-once", "At-least-once", "Exactly-once" 중, 현실적으로 가능한 것은 전자의 두 가지
- 네트워크 분할과 장애 상황에서의 메시지 전달은 불확실성을 내포

### 불가능성의 이유
- **Two Generals Problem** 및 **Byzantine Generals Problem**: 메시지 전달에 대한 근본적인 불확실성을 설명
- **FLP 이론**: 장애 가능성이 있는 시스템에서는 프로세스 간 완전한 합의가 불가능하다는 것을 증명

### 전달 보장 방식
- **At-most-once**: 메시지 수신을 확인하기 전에 처리를 시작함
	- 수신자가 실패하면 데이터가 손실될 가능성
- **At-least-once**: 메시지 처리 후 확인 메시지를 보내지만, 이 경우 메시지가 중복될 가능성 존재

### Exactly-Once Delivery가 불가능한 이유
- 메시지와 확인 메시지의 손실 가능성
- 네트워크 지연, 장애, 성능 저하 등으로 인해 메시지의 순서와 신뢰성이 보장되지 않음
- Atomic broadcast 같은 높은 수준의 동기화가 필요하지만, 이는 성능(지연, 가용성)에 큰 영향

### Exactly-Once Delivery를 시뮬레이션하는 방법
- **Idempotency(멱등성)**: 같은 메시지가 여러 번 처리되어도 결과가 동일하도록 보장
- **Deduplication(중복 제거)**: 메시지 중복을 감지하고 제거하는 방식
- **상태 기반 접근**: 상태 변화를 직접 전파하거나 상태 변경 대신 사실을 기록하는 방식 활용

### 설계에서의 고려 사항
- 메시지의 순서 보장이 중요하지 않은 경우 설계 복잡도를 낮출 수 있음
- **CRDT(Commutative and Convergent Replicated Data Types)** 같은 구조 활용으로 데이터 동기화 문제 해결 가능
- 비동기적이고 장애에 강한 시스템 설계가 필요

### 결론
- 정확히 한 번 메시지를 전달하는 시스템은 이론적으로 불가능
- 현실적인 설계에서는 항상 실패를 고려해야 함
- 대부분의 시스템은 **At-least-once** 방식으로 동작 → 추가적인 설계(멱등성, 중복 제거 등)를 통해 이를 보완
- "Exactly-once"를 주장하는 시스템은 이를 멱등성이나 중복 제거를 통해 시뮬레이션하는 것에 불과