## JpaRepository에서 자동으로 생성하는 메서드

`JpaRepository`가 **자동으로 제공하는 메서드**는 크게 3가지로 나눌 수 있습니다.

---

### 1️⃣ 기본 CRUD 메서드 (`CrudRepository` → `JpaRepository`)

#### 📌 저장 / 수정

```java
<S extends T> S save(S entity);
<S extends T> List<S> saveAll(Iterable<S> entities);
```

* `id == null` → INSERT : 새 엔티티 저장
* `id != null` → UPDATE : 기존 엔티티 수정

- `save()` → 단일 엔티티 저장/수정
- `saveAll()` → 여러 엔티티 저장/수정

---

#### 📌 조회

```java
Optional<T> findById(ID id);
boolean existsById(ID id);
List<T> findAll();
List<T> findAllById(Iterable<ID> ids);
long count();
```

- `count()` → 전체 엔티티 개수 반환

---

#### 📌 삭제

```java
void deleteById(ID id);
void delete(T entity);
void deleteAll();
void deleteAllById(Iterable<? extends ID> ids);
```

---

### 2️⃣ JPA 전용 확장 메서드 (`JpaRepository` 고유)

#### 📌 flush / batch

```java
void flush();
<S extends T> S saveAndFlush(S entity);
void deleteAllInBatch();
void deleteAllInBatch(Iterable<T> entities);
```

* `flush()` → 영속성 컨텍스트를 DB에 즉시 반영
* `deleteAllInBatch()` → **벌크 DELETE (영속성 컨텍스트 무시)**

⚠️ Batch 계열은 **영속성 컨텍스트와 동기화 안 됨**

---

### 3️⃣ 쿼리 메서드 자동 생성 (이름 기반)

> **메서드 이름만으로 JPQL 자동 생성**

#### 📌 기본 패턴

```java
findBy필드명
```

```java
User findByEmail(String email);
List<User> findByAge(int age);
```

---

#### 📌 조건 키워드

| 키워드                                    | 의미    |
| -------------------------------------- | ----- |
| And / Or                               | 논리 조건 |
| Between                                | 범위    |
| LessThan / GreaterThan                 |       |
| IsNull / IsNotNull                     |       |
| Like / NotLike                         |       |
| In / NotIn                             |       |
| StartingWith / EndingWith / Containing |       |
| True / False                           |       |

```java
List<User> findByAgeGreaterThan(int age);
List<User> findByNameContaining(String name);
List<User> findByStatusIn(List<Status> statuses);
```

---

#### 📌 정렬 / 페이징

```java
List<User> findByAgeOrderByNameDesc(int age);
Page<User> findByAge(int age, Pageable pageable);
Slice<User> findByAge(int age, Pageable pageable);
```

---

#### 📌 삭제 / 존재 여부

```java
void deleteByEmail(String email);
boolean existsByEmail(String email);
long countByStatus(Status status);
```

---

### 4️⃣ 반환 타입별 동작 차이

```java
User findByEmail(String email);           // 없으면 null (비권장)
Optional<User> findByEmail(String email); // 권장
List<User> findByEmail(String email);     // 여러 개
```
- 반환 타입을 `User`와 같이 직접 엔티티로 받을 경우, 결과가 없으면 `null` 반환
- → 컴파일 시점에 null 가능성 인지 불가하고, 런타임에서 NullPointerException 발생할 수 있다.
- 따라서 `Optional<User>` 사용을 권장한다.
- `Optional`은 결과가 없을 때 `Optional.empty()`를 반환하여 null 안전성을 제공한다.
- 사용할 땐, `optionalUser.isPresent()` 또는 `optionalUser.orElse()` 등으로 처리한다.
- `List<User>`는 결과가 여러 개일 때 사용하며, 결과가 없으면 빈 리스트를 반환한다.

---

### 5️⃣ 자동 생성 안 되는 경우 (주의)

❌ 필드명이 엔티티와 다를 때
❌ 복잡한 JOIN / 서브쿼리
❌ 집계 + GROUP BY 복잡한 경우

➡️ 이 경우:

```java
@Query("select u from User u join u.orders o where ...")
```

---

### 🔑 핵심 요약

* `JpaRepository`는 **CRUD + JPA 특화 기능**을 자동 제공
* 메서드 이름 규칙만 지키면 **JPQL 자동 생성**
* 배치 삭제는 **영속성 컨텍스트 주의**
* 실무에서는
  👉 **조회: 쿼리 메서드**
  👉 **복잡한 로직: `@Query` / QueryDSL**