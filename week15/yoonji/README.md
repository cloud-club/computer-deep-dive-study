### Q. 컨텍스트 스위칭(Context Switching)이란 무엇이며, 왜 비용이 발생하는가?

- **정의**: CPU가 하나의 프로세스/스레드에서 다른 것으로 전환할 때, 이전 상태를 저장하고 새 상태를 복원하는 과정
- **발생 비용**:
  - 레지스터, 프로그램 카운터, 스택 등의 상태를 커널이 저장/복원
  - 캐시 로컬리티 손실 → 캐시 미스 증가 → 성능 저하
  - 커널 모드 전환 시 오버헤드
- **예시**:
  - 컨텍스트 스위칭이 잦으면 서버 TPS 감소
  - Go는 M:N 스케줄링으로 OS 스레드 수를 줄이고 협력적 스케줄링으로 성능 확보

---

### Q. TCP의 신뢰성은 어떤 메커니즘으로 보장되는가?

- **핵심 요소**:
  - `3-Way Handshake`: 연결 수립 및 초기 시퀀스 동기화
  - `ACK & Seq Number`: 순서 보장 및 패킷 유실 감지
  - `Retransmission`: 타임아웃 또는 중복 ACK 감지 시 재전송
  - `Congestion Control`: 네트워크 혼잡 시 윈도우 크기 조절
    - 알고리즘: Slow Start, Congestion Avoidance, Fast Retransmit/Recovery
- **예시**:
  - 고RTT 환경에서는 BBR 알고리즘이 Cubic보다 효율적
  - HTTP/3는 TCP의 HOL blocking 문제를 피하기 위해 UDP 기반 QUIC 사용

---

### Q. 트랜잭션의 ACID란 무엇이며, 각각을 보장하기 위한 메커니즘은?

- **Atomicity (원자성)**:
  - 모두 성공 or 모두 실패 → Undo 로그, 트랜잭션 롤백
- **Consistency (일관성)**:
  - 무결성 제약조건을 항상 만족 → 제약조건 검사, 트리거
- **Isolation (고립성)**:
  - 동시에 실행되는 트랜잭션 간 간섭 없음 → Lock, MVCC
- **Durability (지속성)**:
  - 커밋된 변경 사항은 영구 반영 → WAL(Write-Ahead Logging), 디스크 동기화

 ---

 ### Q. 무중단 배포(Zero Downtime Deployment)는 어떻게 설계할 수 있는가?

  - 서비스를 사용하는 사용자에게 장애나 중단 없이 새로운 버전을 배포하는 방법
  - 배포 중에도 기존 요청은 정상적으로 처리되어야 함

1. **Blue-Green Deployment**
   - 두 개의 환경(Blue, Green)을 운영
   - Green에서 새 버전 배포 후 테스트 → 트래픽 전환
   - 롤백이 용이함 (트래픽 다시 Blue로 전환)

2. **Canary Deployment**
   - 전체 트래픽 중 일부만 새 버전으로 유도 (5% → 20% → 100%)
   - 오류 발생 시 피해를 최소화하며 롤백 가능

3. **Rolling Update**
   - 전체 인스턴스를 순차적으로 교체
   - Pod 또는 서버 단위로 점진적 배포
   - Kubernetes에서 기본 전략
