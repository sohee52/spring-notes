# Question

Java에서 다음 두 생성자는 동작이 동일한가요?

```java
public RsData(String resultCode, String msg) {
    this(resultCode, msg, null);
}
```

```java
public RsData(String resultCode, String msg) {
    this.resultCode = resultCode;
    this.msg = msg;
    this.data = null;
}
```

# Answer

네. **동작 결과는 동일하다.**
다만 **구현 방식과 유지보수 관점에서 차이가 있다.**

---

## 왜 동일한가 (정확한 이유)

```java
public RsData(String resultCode, String msg) {
    this(resultCode, msg, null);
}
```

이 코드는 **같은 클래스의 다른 생성자**를 호출한다.

```java
public RsData(String resultCode, String msg, T data) {
    this.resultCode = resultCode;
    this.msg = msg;
    this.data = data;
}
```

→ 따라서 `data`에 `null`이 들어가고
→ `resultCode`, `msg`도 동일하게 초기화된다.

즉, **최종 필드 값은 아래 코드와 완전히 같다.**

```java
public RsData(String resultCode, String msg) {
    this.resultCode = resultCode;
    this.msg = msg;
    this.data = null;
}
```

---

## 그런데 왜 위 방식을 더 많이 쓰는가?

### 1️⃣ 중복 제거 (가장 중요)

생성자가 여러 개일 때:

❌ 이렇게 쓰면

```java
public RsData(String resultCode, String msg) {
    this.resultCode = resultCode;
    this.msg = msg;
    this.data = null;
}

public RsData(String resultCode, String msg, T data) {
    this.resultCode = resultCode;
    this.msg = msg;
    this.data = data;
}
```

→ 필드 초기화 로직이 **중복**

---

### 2️⃣ 유지보수 안전성

필드가 하나 추가되면?

```java
private LocalDateTime createdAt;
```

❌ 직접 초기화 방식
→ 모든 생성자 다 수정해야 함 (버그 위험)

⭕ 생성자 위임 방식 (`this(...)`)
→ **한 곳만 수정하면 끝**

---

### 3️⃣ 생성자 체이닝(constructor chaining) – 권장 패턴

```java
public RsData(String resultCode, String msg) {
    this(resultCode, msg, null);
}
```

* Java에서 매우 일반적인 패턴
* Lombok `@AllArgsConstructor` / `@RequiredArgsConstructor` 내부도 이 방식

---

## 차이점 정리 (한 줄 요약)

| 구분    | 결과         | 차이 |
| ----- | ---------- | -- |
| 필드 값  | 동일         | ❌  |
| 실행 흐름 | 다름         | ⭕  |
| 유지보수  | 위 코드가 더 좋음 | ⭕  |

---

## 결론

* **의미·결과는 완전히 동일**
* 하지만
  👉 `this(resultCode, msg, null)` 방식이 **더 안전하고 권장됨**
* 실무/오픈소스/프레임워크 코드에서도 이 패턴이 표준처럼 쓰임