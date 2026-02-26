# Cache Server (Redis)

Board Service의 조회 트래픽을 분산하기 위한 Redis 캐시 서버입니다.

---

## 1. 역할

- 게시글 목록 조회 캐시
- DB 부하 감소
- 읽기 트래픽 분산

Board Service의 고트래픽 구간인 `GET /api/boards` 응답을 캐싱합니다.

---

## 2. 설계 의도

게시판 서비스는 읽기 트래픽이 집중되는 구조로 가정했습니다.

따라서:

- DB 직접 조회를 최소화
- 동일 요청에 대한 반복 쿼리 방지
- 트래픽 급증 시 RDS 보호

를 목표로 Redis 캐시를 도입했습니다.

---

## 3. 캐시 전략

### 적용 대상
`GET /api/boards`

### TTL 전략

- TTL: 1분
- 실시간 반영이 필수적이지 않은 목록 데이터
- Eventual Consistency 허용

게시글 생성 직후 1분 이내에는 목록에 반영되지 않을 수 있습니다.

정합성보다 트래픽 분산을 우선한 설계입니다.

---

## 4. 무효화 전략

현재는 TTL 기반 자동 만료 전략을 사용합니다.

운영 환경에서는 다음 전략 적용 가능:

- 게시글 생성 시 Cache Evict
- 이벤트 기반 캐시 무효화

---

## 5. Local 실행 방법

```bash
docker-compose up -d
```

### 기본 포트: 6379

## 6. 운영 환경 구성
- AWS ElastiCache (Redis)
- Private Subnet 배치
- Board Service와 동일 VPC 내 통신

---

## 7. 성능 개선 효과

Redis OFF → DB 직접 조회
Redis ON → 캐시 히트 시 DB 미접근

k6 부하 테스트 결과:
- 평균 응답 시간 감소
- RDS CPU 사용량 감소
- ASG 스케일링 지연 완화
