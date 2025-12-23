## RuntimeException

### RuntimeException 상속
- `extends RuntimeException` 과 같이 RuntimeException 클래스를 상속받아 사용자 정의 예외 클래스를 만들 수 있습니다.
- 생성자에 super(msg) 를 호출하여 msg에 개발자가 정의한 예외 메시지를 전달할 수 있습니다.

```java
@Getter
public class DomainException extends RuntimeException {
    private final String resultCode;
    private final String msg;

    public DomainException(String resultCode, String msg) {
        super(resultCode + " : " + msg); // RuntimeException의 생성자는 메시지를 인자로 받음
        this.resultCode = resultCode;
        this.msg = msg;
    }
}
```

### DomainException에서 에러 코드와 메시지를 분리하는 이유

### 1. 왜 에러 코드와 메시지를 분리하는가?

#### 1️⃣ 에러 코드 (resultCode)

* **기계가 이해하는 식별자**
* 프론트엔드 / 다른 서버에서 **분기 처리 기준**으로 사용
* 메시지가 바뀌어도 **의미는 변하지 않음**
* 다국어 처리와 무관

```json
{
  "code": "USER_NOT_FOUND",
  "message": "사용자를 찾을 수 없습니다."
}
```

```ts
if (error.code === 'USER_NOT_FOUND') {
  // 특정 처리
}
```

👉 메시지를 기준으로 분기하면 문구 수정 시 로직이 깨짐
👉 **에러 코드는 절대 바뀌지 않는 약속**

---

#### 2️⃣ 메시지 (msg)

* **사람이 읽는 설명**
* 사용자에게 보여주거나 로그용
* 다국어(i18n) 처리 가능
* 정책 변경에 따라 수정 가능

```java
throw new DomainException(
    "USER_NOT_FOUND",
    "존재하지 않는 사용자입니다."
);
```

---

#### 3️⃣ HTTP 상태 코드만으로는 부족

HTTP Status는 범위가 큼

| 상황      | HTTP |
| ------- | ---- |
| 사용자 없음  | 404  |
| 비밀번호 틀림 | 400  |
| 탈퇴한 회원  | 403  |

→ 실제 원인 구분은 **에러 코드로 처리**

---

### 2. `super(resultCode + " : " + msg)` 를 사용하는 이유

```java
super(resultCode + " : " + msg);
```

* `RuntimeException`의 메시지를 설정
* 로그, 콘솔, 모니터링 도구(Sentry 등)에 바로 출력됨
* `e.getMessage()`가 의미 있는 값이 됨

👉 **예외도 로그 자원**이므로 메시지를 채우는 것이 좋음

---

## 3.  실무에서 자주 하는 개선 포인트

#### 1️⃣ 에러 코드는 `String` 대신 `enum` 사용

* 오타 방지
* 중앙 관리 가능

```java
public enum ErrorCode {
    USER_NOT_FOUND,
    DUPLICATE_EMAIL,
    INVALID_PASSWORD
}
```

```java
public class DomainException extends RuntimeException {
    private final ErrorCode errorCode;

    public DomainException(ErrorCode errorCode, String msg) {
        super(errorCode.name() + " : " + msg);
        this.errorCode = errorCode;
    }
}
```

---

#### 2️⃣ 메시지를 예외에 넣지 않는 방식도 있음

* 메시지를 응답 계층에서 생성
* 메시지 변경이 잦은 서비스에서 선호

```java
throw new DomainException(ErrorCode.USER_NOT_FOUND);
```

---

### 5. DomainException을 사용하는 경우

#### ✅ 사용해야 하는 경우 (비즈니스 예외)

* 사용자 없음
* 이미 처리된 요청
* 권한 없음
* 상태 전이 불가
* 도메인 규칙 위반

#### ❌ 사용하지 않는 경우

* NullPointerException
* DB 연결 실패
* 시스템 / 인프라 예외

👉 **비즈니스 규칙 위반 전용 예외**로 사용하는 것이 적절함

---

## 핵심 요약

* 에러 코드: **기계용, 고정, 분기 기준**
* 메시지: **사람용, 가변, 설명 목적**
* 분리하는 설계는 표준에 가까움
* 실무에서는 `enum ErrorCode + GlobalExceptionHandler` 구조로 발전시킴