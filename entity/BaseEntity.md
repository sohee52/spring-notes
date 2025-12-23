## BaseEntity

### BaseEntity 예시
```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### @MappedSuperclass

- 부모 클래스의 필드를 “자식 엔티티의 컬럼으로 매핑” 해주는 JPA 애노테이션

#### 언제 쓰는가
- id
- createdAt / updatedAt
- deleted
- version

처럼 모든 엔티티가 공유하는 컬럼

### @EntityListeners(AuditingEntityListener.class)
- 엔티티의 생명주기 이벤트를 가로채서 생성/수정 시점을 자동으로 채워주는 리스너 등록
- **“이 엔티티에서 auditing 이벤트를 듣겠다”**는 선언
- ❌ 이거 없으면 → Auditing이 이 엔티티에 적용되지 않음

#### 왜 필요한가?
```java
@CreatedDate
private LocalDateTime createdAt;

@LastModifiedDate
private LocalDateTime updatedAt;
```
- 이 애노테이션들은 혼자서는 동작하지 않음
- `AuditingEntityListener`가 있어야 엔티티가 생성/수정될 때 자동으로 값을 채워줌

#### 꼭 함께 사용해야 하는 설정
- Spring Boot 메인 클래스에 `@EnableJpaAuditing` 추가

```java
@EnableJpaAuditing
@SpringBootApplication
public class Application {
}
```

#### @EnableJpaAuditing
- JPA Auditing 기능 자체를 켜는 스위치
- Spring Data JPA 레벨 설정
- 애플리케이션 전체에 1번만
- ❌ 이거 없으면 → @CreatedDate, @LastModifiedDate 절대 동작 안 함

#### 비교: @EntityListeners vs @EnableJpaAuditing
- `@EnableJpaAuditing`        ← 기능 활성화
- `@EntityListeners(...)`     ← 엔티티에 적용

### 정리

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {
}
```
  
- 이 클래스는 테이블은 아니지만 자식 엔티티의 컬럼을 제공하고, 엔티티 생명주기 이벤트를 함께 처리한다