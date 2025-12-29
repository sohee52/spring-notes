# getReferenceById

`JpaRepository`의 **`getReferenceById`**는 데이터베이스 성능 최적화를 위해 사용하는 중요한 메서드입니다.

한마디로 정의하자면, **"DB 조회를 미루고, 가짜 객체(프록시)만 가져오는 메서드"**입니다.

---

### 1. 핵심 특징

* **지연 로딩 (Lazy Loading):** 이 메서드를 호출하는 시점에는 DB에 `SELECT` 쿼리를 날리지 않습니다.
* **프록시(Proxy) 반환:** 실제 엔티티 객체 대신, 실제 객체의 상속받은 껍데기(프록시) 객체를 반환합니다. 이 객체는 **ID(식별자) 값만 가지고 있습니다.**
* **쿼리 시점:** 반환받은 프록시 객체의 내부 필드(예: `user.getName()`)를 실제로 사용할 때 비로소 DB에 쿼리를 날립니다.

### 2. `findById` vs `getReferenceById` 비교

가장 많이 헷갈리는 두 메서드의 차이는 다음과 같습니다.

| 특징 | `findById(id)` | `getReferenceById(id)` |
| --- | --- | --- |
| **동작 방식** | 호출 즉시 DB에 `SELECT` 쿼리 전송 | DB 조회 없이 **프록시 객체**만 즉시 반환 |
| **반환 타입** | `Optional<T>` | `T` (엔티티 타입) |
| **실제 사용 시점** | 이미 데이터가 로딩되어 있음 | 내부 필드 접근 시점에 쿼리 발생 (초기화) |
| **데이터 없음** | `Optional.empty()` 반환 | 사용 시점에 `EntityNotFoundException` 발생 |
| **주 사용처** | 실제 데이터를 조회해서 보여줄 때 | **연관 관계(Foreign Key)만 맺어줄 때** |

> **참고:** 과거의 `getOne(id)`이나 `getById(id)`가 현재 `getReferenceById(id)`로 이름이 변경되었습니다. (Spring Data JPA 2.5/2.7 이후)

---

### 3. 언제, 왜 사용하는가? (가장 중요한 이유)

주로 **INSERT 성능 최적화**를 위해 사용합니다.

예를 들어, `Member`(회원)가 글을 작성하여 `Post`(게시글)를 저장한다고 가정해 보겠습니다. `Post`는 `Member`를 참조(Foreign Key)해야 합니다.

**[비효율적인 방식: `findById` 사용]**

```java
// 1. DB에서 Member를 조회함 (SELECT 쿼리 발생 - 낭비!)
Member member = memberRepository.findById(memberId).get();

// 2. 게시글 생성
Post post = new Post();
post.setMember(member); // FK 설정을 위해 객체가 필요함

// 3. 저장
postRepository.save(post);

```

* 게시글을 저장할 때는 `member_id` 값만 있으면 됩니다. 굳이 Member의 이름, 나이 등을 DB에서 가져올 필요가 없습니다.

**[효율적인 방식: `getReferenceById` 사용]**

```java
// 1. DB 조회 안 함! ID만 가진 프록시 객체 생성 (SELECT 쿼리 0건)
Member memberRef = memberRepository.getReferenceById(memberId);

// 2. 게시글 생성
Post post = new Post();
post.setMember(memberRef); // 프록시를 넣어도 JPA가 알아서 ID만 꺼내서 FK로 저장함

// 3. 저장 (INSERT 쿼리만 발생)
postRepository.save(post);

```

* 이렇게 하면 불필요한 `SELECT` 쿼리 한 번을 아낄 수 있습니다.

---

### 4. 주의사항 ⚠️

1. **LazyInitializationException:**
* 트랜잭션(`@Transactional`) 범위 밖에서 프록시 객체의 필드에 접근하려고 하면 에러가 발생합니다. (세션이 이미 닫혔기 때문)


2. **데이터 존재 여부 확인 불가:**
* 메서드 호출 시점에는 DB를 확인하지 않으므로, 해당 ID가 실제 DB에 있는지 없는지 모릅니다. 나중에 필드를 사용할 때 없으면 예외(`EntityNotFoundException`)가 터집니다.



### 요약

**"단순히 다른 테이블과 관계(FK)를 맺기 위해 객체가 필요할 때는 `findById` 대신 `getReferenceById`를 사용하여 불필요한 조회를 줄이세요."**