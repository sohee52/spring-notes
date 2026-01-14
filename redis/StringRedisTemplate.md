# StringRedisTemplate

**`StringRedisTemplate`**은 **Spring Data Redis**에서 제공하는 클래스이고,
**Redis를 “문자열(String) 중심”으로 안전하고 간단하게 다루기 위한 전용 템플릿**이다.

---

## 한 줄 요약

> **Redis의 key/value를 모두 `String`으로만 다루는 Redis 전용 헬퍼 클래스**

---

## 왜 `StringRedisTemplate`가 필요한가?

Redis는 본질적으로 **문자열 기반 저장소**다.
하지만 Spring에서 제공하는 일반 템플릿인 `RedisTemplate<K, V>`는

* 직렬화 방식 설정이 필요하고
* 잘못 설정하면 **깨진 값, 디버깅 지옥**이 된다

👉 그래서 **문자열만 쓰는 경우를 위한 안전한 기본값**으로 나온 게 `StringRedisTemplate`.

---

## 핵심 특징

### 1️⃣ Key / Value 타입이 고정

```java
StringRedisTemplate
// key   : String
// value : String
```

* 제네릭 없음
* 타입 캐스팅 실수 없음
* 직렬화 고민 없음

---

### 2️⃣ 기본 직렬화 전략

| 항목    | 직렬화 방식                |
| ----- | --------------------- |
| key   | StringRedisSerializer |
| value | StringRedisSerializer |

👉 Redis CLI에서 **사람이 읽을 수 있는 값** 그대로 보인다.

```bash
127.0.0.1:6379> GET auth:token:123
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### 3️⃣ Redis 자료구조를 그대로 제공

```java
stringRedisTemplate.opsForValue(); // String
stringRedisTemplate.opsForList();  // List
stringRedisTemplate.opsForSet();   // Set
stringRedisTemplate.opsForHash();  // Hash
stringRedisTemplate.opsForZSet();  // Sorted Set
```

---

## 사용 예시

### ✔ 값 저장 / 조회

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

public void save() {
    stringRedisTemplate.opsForValue()
        .set("auth:code:123", "ABCDEF", Duration.ofMinutes(5));
}

public String get() {
    return stringRedisTemplate.opsForValue()
        .get("auth:code:123");
}
```

---

### ✔ TTL 설정

```java
stringRedisTemplate.expire("auth:code:123", 5, TimeUnit.MINUTES);
```

---

### ✔ 카운터 (조회수, 시도 횟수)

```java
Long count = stringRedisTemplate.opsForValue()
    .increment("login:fail:sohee");
```

---

## `RedisTemplate` vs `StringRedisTemplate`

| 구분    | RedisTemplate | StringRedisTemplate |
| ----- | ------------- | ------------------- |
| 타입    | `<K, V>` 제네릭  | String 고정           |
| 직렬화   | 직접 설정 필요      | 기본 제공               |
| 가독성   | ❌ (깨질 수 있음)   | ✅                   |
| 추천 용도 | 객체 캐싱         | 토큰, 인증, 카운터         |

👉 **실무에서는 80% 이상 `StringRedisTemplate` 사용**

---

## 언제 쓰는 게 맞을까?

### 👍 `StringRedisTemplate` 추천

* JWT / OAuth state
* 이메일·SMS 인증 코드
* 로그인 실패 횟수
* 락 키 (distributed lock)
* rate limit
* 캐시 키 관리

### 👎 비추천

* 복잡한 객체를 그대로 Redis에 저장할 때
  → 이 경우 `RedisTemplate + JSON 직렬화`가 적합

---

## 실무 관점 한 줄 정리

> **“Redis에 문자열만 넣을 거면 무조건 `StringRedisTemplate` 써라.”**