# 3가지 데이터 저장 및 조회 방식

## 1. JPA 사용 (Query Method)

- 가장 기본적이고 간단한 CRUD 기능 구현에 편리
- 예시:
  ```java
  public interface UserRepository extends JpaRepository<User, Long> {
      User findByUsername(String username);
      List<User> findByAgeGreaterThan(int age);
  }
  ```

## 2. JPA의 JPQL 사용

- 복잡한 쿼리나 특정 데이터베이스 함수 사용 시 유용
- 페이징 처리나 대량의 데이터 조회 시 성능 최적화에 필요
- 예시:
  ```java
  @Query("SELECT u FROM User u WHERE u.age > :age")
  List<User> findUsersOlderThan(@Param("age") int age);
  
  @Query(value = "SELECT * FROM users WHERE FUNCTION('DATE_FORMAT', created_at, '%Y') = :year", nativeQuery = true)
  List<User> findUsersByYear(@Param("year") String year);
  ```

## 3. QueryDSL 라이브러리 사용

- 동적 쿼리 생성에 강점
- 컴파일 시점에 오류 검출 가능
- 타입 안전한 쿼리 작성 가능
- 예시:
  ```java
  public List<User> findDynamicQuery(String username, Integer age) {
      return queryFactory
          .selectFrom(user)
          .where(usernameEq(username),
                 ageGt(age))
          .fetch();
  }
  
  private BooleanExpression usernameEq(String username) {
      return username != null ? user.username.eq(username) : null;
  }
  
  private BooleanExpression ageGt(Integer age) {
      return age != null ? user.age.gt(age) : null;
  }
  ```

## 추가 팁

- 성능과 복잡성을 고려하여 적절한 방식 선택
- 단순 CRUD: JPA Query Method
- 복잡한 쿼리 또는 특정 DB 기능 필요: JPQL
- 동적 쿼리 또는 타입 안전성 필요: QueryDSL
- 엔티티 업데이트 시 `@DynamicUpdate` 사용으로 변경된 필드만 업데이트 가능

