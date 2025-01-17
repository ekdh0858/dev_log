# DB 커넥션 스레드 풀

## DB 연결의 비용
- 어플리케이션에서 DB로의 연결은 큰 비용이 드는 작업.
- 네트워크를 통해 커넥션을 맺는 무거운 동작이며, 매 요청마다 연결을 생성하면 병목 현상을 유발.
- **DB Connection Pool**: 연결을 미리 생성하고 관리하여 요청마다 연결을 재사용.

---

## Spring 기준 DB Connection Pool
- **기본 커넥션 풀 크기**: 10개.
- Spring에서는 **`DataSource` 인터페이스**를 통해 커넥션을 관리.
- 과거에는 Tomcat JDBC 사용 → Spring Boot 2.0 이후 **HikariCP**가 기본 옵션.

---

## 커넥션 풀의 크기 설정
- 커넥션 풀이 많다고 해서 무조건 성능이 좋아지지 않음.
- **이유**: 잦은 컨텍스트 스위칭으로 인해 성능 저하.
- **DB 스레드 제한**:
  - 스레드 수는 **CPU 코어 개수 × 2** + **저장 매체의 특성**에 따라 제한.
- **최적의 커넥션 풀 크기**:
  - **NGrinder**와 같은 부하 테스트 도구를 사용하여 튜닝 필요.

---

## 커넥션 반환을 빠르게 하는 방법

### 트랜잭션 범위 최적화
- 트랜잭션을 작게 설정하여 DB 커넥션을 빠르게 반환.

### Open-in-View (OSIV) 설정
- **OSIV 기본 개념**:
  - 켜두면 Controller 반환 시까지 커넥션을 유지.
  - 끄면 커넥션의 영속성 컨텍스트 생존 범위를 서비스와 레포지토리로 제한.

- **OSIV 설정 효과**:
  - **켜진 경우**:
    - 뷰 템플릿 및 API 컨트롤러에서 지연 로딩 사용 가능.
    - 성능 영향을 덜 받는 **어드민 서비스**에서는 OSIV 설정이 유리할 수 있음.
  - **꺼진 경우**:
    - 커넥션이 빠르게 반환되어 효율적.
    - 영속성 컨텍스트 생존 범위가 짧아짐(서비스와 레포지토리 내부로 제한).

- **권장 사항**:
  - 성능 민감도가 높은 서비스에서는 OSIV를 꺼두고 최적화.
  - 성능에 큰 영향을 미치지 않는 환경에서는 켜둘 수 있음.
