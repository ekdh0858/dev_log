# 도메인 객체와 엔티티 객체 분리

## 도메인 객체와 엔티티 객체를 동일하게 가져가는 경우
- **도메인 객체**와 **엔티티 객체**를 통합하여 하나로 관리.
- 장점:
  - 관리 대상 객체 수가 줄어들어 **코드 작성 및 유지보수가 간단**.
- 단점:
  - 엔티티 클래스에 비즈니스 로직과 관련 없는 코드가 추가될 가능성.
  - 데이터베이스에 종속된 설계로 인해, 데이터베이스 변경 시 영향이 큼.

### 필요 어노테이션
엔티티 객체로 사용하려면 다음과 같은 어노테이션을 추가해야 합니다:
- `@Entity`: JPA 엔티티로 선언.
- `@Id`: 기본 키(Primary Key) 설정.
- `@GeneratedValue`: 기본 키 생성 전략 설정.
- `@Table`: 테이블 이름과 매핑 설정 (필요시).

---

## 도메인 객체와 엔티티 객체를 분리하는 경우
- **엔티티 객체**: 데이터베이스와의 매핑과 관련된 기능만 담당.
- **도메인 객체**: 비즈니스 로직을 처리하며, 데이터베이스 의존성을 제거.

### 장점
1. **데이터베이스와 비즈니스 로직의 완전한 분리**:
   - 엔티티 클래스는 데이터베이스 매핑만 담당.
   - 비즈니스 로직은 도메인 객체에서 관리.

2. **유연성 증가**:
   - 데이터베이스 변경에도 비즈니스 로직에 영향이 적음.
   - 특정 요구사항에 따라 도메인 객체를 확장하거나 수정 가능.

3. **코드의 복잡도 감소**:
   - 비즈니스 로직과 상관없는 필드(칼럼) 추가를 엔티티 클래스에 제한.

### 단점
- 관리해야 할 객체가 늘어나면서 개발 시간이 증가.

---

## 예시: 생성 순서를 보장하는 요구사항

### 통합 방식
- 엔티티 객체와 도메인 객체를 통합한 경우:
  - 생성 순서를 보장하기 위해 `order`와 같은 칼럼을 엔티티 클래스에 추가.
  - 문제점: 해당 칼럼은 비즈니스 로직과 무관하므로 엔티티 클래스의 역할이 혼재됨.

```java
@Entity
public class ExampleEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private LocalDateTime createdAt;

    private Integer order; // 비즈니스 로직과 무관하지만 순서를 보장하기 위해 추가

    // Getters and Setters
}
```

### 분리 방식
- **도메인 객체**와 **엔티티 객체**를 분리:
  - `order`와 같은 필드는 도메인 객체에만 존재.
  - 엔티티 클래스는 데이터베이스와의 매핑에 필요한 정보만 포함.
  - 데이터베이스 변경에도 비즈니스 로직이 유지 가능.

#### 엔티티 클래스
```java
@Entity
@Table(name = "example_entity")
public class ExampleEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private LocalDateTime createdAt;

    // Getters and Setters
}
```

#### 도메인 클래스 
```java
public class Example {
    private Long id;
    private String name;
    private LocalDateTime createdAt;
    private Integer order;

    // Business Logic
    public Example(ExampleEntity entity, Integer order) {
        this.id = entity.getId();
        this.name = entity.getName();
        this.createdAt = entity.getCreatedAt();
        this.order = order;
    }
}
```

#### 결론
도메인 객체와 엔티티 객체를 분리하면 데이터베이스 의존성을 제거하고 코드 유연성을 확보할 수 있습니다.
데이터베이스에 종속되지 않고 비즈니스 로직을 명확히 분리하는 설계를 위해, 가능한 한 엔티티 클래스를 따로 관리하는 것이 바람직합니다.
시간이 부족하거나 프로젝트 규모가 작다면 통합 방식도 선택할 수 있지만, 장기적인 유지보수와 확장성을 고려한다면 객체를 분리하는 설계가 더 권장됩니다.