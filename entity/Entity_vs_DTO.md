## Entity와 DTO

### 엔티티를 그대로 쓰지 않고 DTO로 감싸는 이유

① **도메인(Entity) 보호**

* 엔티티는 **비즈니스 규칙과 상태를 가진 객체**
* 외부(API 요청/응답, 화면)에 그대로 노출되면

  * 의도치 않은 필드 변경
  * 내부 구조 유출
  * 검증 없는 값 주입
    이 발생할 수 있음

👉 **DTO는 “외부와의 계약”**, 엔티티는 “내부 규칙”

---

② **변경 전파 방지 (강한 결합 제거)**

* 엔티티 구조가 바뀌면 API 스펙까지 함께 깨짐
* DTO를 사용하면

  * 엔티티 변경 ≠ API 변경
  * 내부 리팩토링이 자유로움

👉 실무에서 가장 큰 이유

---

③ **의도치 않은 영속성 변경 방지 (JPA 핵심)**

* 엔티티는 **영속 상태**일 수 있음
* 컨트롤러에서 엔티티 값을 바꾸면

  * 트랜잭션 종료 시 자동 UPDATE 발생 (Dirty Checking)

```java
// 위험한 예
@PostMapping("/users")
public void update(User user) {
    user.setRole("ADMIN"); // DB에 바로 반영될 수 있음
}
```

👉 DTO는 **영속성과 완전히 무관**
👉 명시적으로 `toEntity()` / `update()` 호출해야만 변경됨

---

④ **요청/응답에 맞는 형태로 가공 가능**

* 엔티티 ≠ 화면에 필요한 데이터 구조

```java
// 엔티티
User { id, email, password, role, createdAt }

// 응답 DTO
UserResponse { id, email }
```

👉 보안 + 가독성 + 명확한 책임

---

⑤ **검증(@Valid)의 책임 분리**

* DTO에 검증 로직을 둔다

```java
public class UserCreateRequest {
    @NotBlank
    @Email
    private String email;

    @Size(min = 8)
    private String password;
}
```

👉 엔티티는 **비즈니스 규칙**
👉 DTO는 **입력 규칙**

---

### DTO vs Entity 차이 정리

| 구분        | Entity         | DTO    |
| --------- | -------------- | ------ |
| 목적        | 도메인 모델         | 데이터 전달 |
| JPA 어노테이션 | 있음 (`@Entity`) | 없음     |
| 생명주기      | 영속성 컨텍스트 관리    | 단순 객체  |
| 비즈니스 로직   | 포함됨            | 없어야 함  |
| 검증        | 비즈니스 규칙        | 입력값 검증 |
| 외부 노출     | ❌ 금지           | ⭕ 허용   |
| 변경 영향     | DB 반영 가능       | 영향 없음  |

---

### 실무에서의 정석 흐름

```text
Controller → DTO
DTO → Entity (변환)
Entity → Service (비즈니스 처리)
Entity → DTO (응답 변환)
```

```java
// Controller
@PostMapping("/users")
public void create(@RequestBody @Valid UserCreateRequest dto) {
    userService.create(dto);
}

// Service
public void create(UserCreateRequest dto) {
    User user = dto.toEntity();
    userRepository.save(user);
}
```

---

### 한 줄로 요약

> 엔티티는 내부 규칙을 지키는 객체이고, DTO는 외부와 안전하게 데이터를 주고받기 위한 껍데기다.
> 엔티티를 직접 넘기는 순간, 설계는 무너진다.
