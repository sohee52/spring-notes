## 1️⃣ `@Transactional`의 기본값 (정확한 사실)

```java
@Transactional
```

의 **기본값은 아래와 같습니다.**

* `readOnly = false`  ✅ (기본)
* 전파 옵션: `Propagation.REQUIRED`
* 격리 수준: `Isolation.DEFAULT`
* 롤백: `RuntimeException`, `Error` 발생 시

즉,

> **아무 옵션 없이 쓰면 “쓰기 가능한 트랜잭션”** 입니다.

---

## 2️⃣ `readOnly = true`는 언제 쓰는가 (명확한 기준)

### ✅ **DB 상태를 절대 변경하지 않는 경우**

아래 조건 중 **하나라도 해당하면 `readOnly = true`** 를 붙입니다.

* `SELECT`만 수행
* 엔티티 값을 **변경하지 않음**
* 저장(`save`), 삭제(`delete`) 없음
* dirty checking이 필요 없음

### 대표 예시

```java
@Transactional(readOnly = true)
public Optional<Member> findById(Long id)

@Transactional(readOnly = true)
public long count()

@Transactional(readOnly = true)
public List<Member> findAll()
```

---

## 3️⃣ `readOnly = true`를 쓰는 이유 (단순 최적화 아님)

### ① **의도 명확화 (가장 중요)**

```java
@Transactional(readOnly = true)
```

→ 이 메서드는 **조회 전용**이라는 명확한 계약

팀원이 봐도:

> “아, 여기서는 수정하면 안 되는구나”

---

### ② **실수 방지**

```java
@Transactional(readOnly = true)
public Member find(...) {
    member.changeNickname("hack"); // ❌ 의도치 않은 변경
}
```

* JPA(Hibernate)는 flush를 하지 않거나
* 일부 DB에서는 write를 막거나
* 최소한 “이상한 코드”라는 신호를 줌

→ **사이드 이펙트 방지**

---

### ③ **성능 최적화 (부가 효과)**

JPA / Hibernate 기준:

* dirty checking 생략
* flush 생략
* snapshot 저장 안 함

> 큰 조회 트래픽에서는 체감 차이가 납니다.

---

## 4️⃣ `readOnly = true`를 쓰면 안 되는 경우

### ❌ **엔티티 상태가 바뀌는 경우**

아래 중 하나라도 있으면 **절대 readOnly 쓰면 안 됨**

```java
member.changePassword(...)
member.increasePoint(...)
member.deactivate()
repository.save(member)
repository.delete(...)
```

### 예시 (잘못된 코드)

```java
@Transactional(readOnly = true) // ❌
public void changeNickname(Long id, String nickname) {
    Member member = repository.findById(id).orElseThrow();
    member.changeNickname(nickname); // dirty checking 필요
}
```

→ 변경이 **DB에 반영되지 않을 수 있음**

---

## 5️⃣ 그럼 readOnly 안 붙여도 되는 경우는?

### ✔ **쓰기 메서드**

```java
@Transactional
public Member join(...)
```

* 기본값 `readOnly = false`
* 따로 명시 안 해도 OK

### ✔ **애매하면 안 붙이는 게 안전**

> “읽기만 할 것 같긴 한데, 나중에 수정 로직 들어갈 수도 있음”

→ 이럴 땐 **readOnly 안 붙이는 게 맞음**

---

## 6️⃣ 실전 판단 기준 (이대로 쓰면 됨)

### 🔹 Facade / Service 기준

| 메서드 성격                     | 권장                                |
| -------------------------- | --------------------------------- |
| 순수 조회                      | `@Transactional(readOnly = true)` |
| 생성 / 수정 / 삭제               | `@Transactional`                  |
| 조회 + 변경                    | `@Transactional`                  |
| 단순 위임(Controller → Facade) | Facade에만                          |

---

## 7️⃣ 한 줄로 정리

* `@Transactional` 기본값 → **`readOnly = false`**
* **조회만 하면 → `readOnly = true`**
* **엔티티 값이 바뀌면 → 절대 readOnly X**
* 애매하면 → **안 붙이는 쪽이 안전**

이 기준만 지키면 트랜잭션 관련해서 실수할 일 거의 없습니다.
