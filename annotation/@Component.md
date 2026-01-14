# @Component

`@Component`는 **스프링 컨테이너에 이 클래스를 “빈(Bean)”으로 등록하라고 알려주는 애너테이션**이다.
정확히 말하면, **이 클래스의 객체 생성을 스프링이 직접 관리하게 만든다**는 의미다.

---

## 핵심 요약

```java
@Component
public class KakaoOAuthClient implements OAuthClient
```

이 한 줄이 의미하는 것:

> **KakaoOAuthClient를 스프링이 자동으로 생성하고, 필요한 곳에 주입해줄 수 있게 한다**

---

## 구체적으로 무슨 일이 일어나는가

### 1️⃣ 컴포넌트 스캔 대상이 됨

스프링 부트는 실행 시

* `@Component`
* `@Service`
* `@Repository`
* `@Controller`

같은 애너테이션이 붙은 클래스를 **컴포넌트 스캔**으로 찾는다.

👉 `KakaoOAuthClient`가 스캔되면
스프링 컨테이너 내부에 **Bean 객체로 등록**된다.

---

### 2️⃣ 객체 생성을 스프링이 담당

`@Component`가 없으면:

```java
KakaoOAuthClient client = new KakaoOAuthClient(); // 직접 생성
```

`@Component`가 있으면:

```java
@Autowired
private KakaoOAuthClient kakaoOAuthClient;
```

또는 (권장)

```java
@RequiredArgsConstructor
public class AuthFacade {
  private final OAuthClient kakaoOAuthClient;
}
```

👉 **new를 직접 쓰지 않는다**
👉 생성 시점, 생명주기, 의존성 주입을 전부 스프링이 관리

---

### 3️⃣ 인터페이스 기반 주입이 가능해짐

```java
public class KakaoOAuthClient implements OAuthClient
```

```java
@RequiredArgsConstructor
public class AuthFacade {
  private final OAuthClient oAuthClient;
}
```

이게 가능한 이유:

* `KakaoOAuthClient`가 `@Component`로 등록된 Bean이고
* `OAuthClient` 타입을 구현했기 때문

👉 **DIP(의존성 역전 원칙)** 충족
👉 구현체 교체 가능 (Kakao → Google, Naver 등)

---

## 만약 `@Component`가 없다면?

```java
@Component ❌
public class KakaoOAuthClient implements OAuthClient
```

이 경우:

* 스프링이 이 클래스를 모름
* `@Autowired` / 생성자 주입 시점에

❌ `NoSuchBeanDefinitionException` 발생

---

## 그럼 `@Service`랑 차이는?

기능적으로는 **완전히 동일**하다.

| 애너테이션         | 의미적 용도              |
| ------------- | ------------------- |
| `@Component`  | 가장 일반적인 Bean        |
| `@Service`    | 비즈니스 로직             |
| `@Repository` | DB 접근 계층 (예외 변환 포함) |
| `@Controller` | 웹 요청 처리             |

👉 `KakaoOAuthClient`는

* 외부 OAuth 서버와 통신하는 **infra / client 역할**
* 비즈니스 로직 아님

➡️ **`@Component`가 가장 적절**
(또는 아키텍처에 따라 `@Component` + 패키지로 구분)

---

## 한 줄로 정리하면

> `@Component`는
> **“이 클래스는 스프링이 관리하는 객체니까, 필요하면 주입해서 써라”**
> 라고 선언하는 애너테이션이다.