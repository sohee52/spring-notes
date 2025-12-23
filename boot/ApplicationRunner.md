## ApplicationRunner

### 1. ApplicationRunner란?

* **Spring Boot에서 제공하는 인터페이스**
* 애플리케이션이 **완전히 기동된 직후 딱 한 번 자동 실행**됨
* 초기화(bootstrap) 로직을 작성할 때 사용

```java
public interface ApplicationRunner {
    void run(ApplicationArguments args) throws Exception;
}
```

---

### 2. 실행 시점

Spring Boot 실행 흐름:

1. `main()` 실행
2. Spring Context 생성
3. Bean 등록 및 의존성 주입 완료
4. **ApplicationRunner 실행 ← 이 시점**
5. 웹 서버(Tomcat 등) 시작
6. 요청 수신 가능

➡️ 모든 Bean 사용 가능 (Repository, Service, Transaction 가능)

---

### 3. 주 사용 목적

* 기본 데이터 초기화

  * 예: 기본 회원, 게시글, 테스트 데이터
* 개발 환경 전용 더미 데이터 세팅
* 캐시 워밍업
* 외부 시스템 연결 확인

➡️ **요청 기반 로직이 아닌, 시작 시 1회 실행 로직**에 적합

---

### 4. 등록 방법

#### 방법 1️⃣ Bean으로 등록 (람다 방식)

```java
@Bean
public ApplicationRunner runner() {
    return args -> {
        // 초기화 로직
    };
}
```

#### 방법 2️⃣ 클래스 구현

```java
@Component
public class InitRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        // 초기화 로직
    }
}
```

➡️ 두 방식은 동작 동일

---

### 5. ApplicationArguments args

* 애플리케이션 실행 시 전달된 인자를 담은 객체
* `main(String[] args)`를 구조화한 형태

예시 실행:

```bash
java -jar app.jar --mode=dev --port=8081
```

사용 예:

```java
args.containsOption("mode");
args.getOptionValues("mode");
```

---

### 6. CommandLineRunner와 차이

| 구분    | ApplicationRunner    | CommandLineRunner |
| ----- | -------------------- | ----------------- |
| 인자 타입 | ApplicationArguments | String[]          |
| 구조화   | O                    | X                 |
| 가독성   | 좋음                   | 보통                |
| 사용 권장 | ✅                    | ⭕                 |

➡️ **ApplicationRunner가 상위 호환**

---

### 7. 여러 개 사용 시 실행 순서

* 기본적으로 순서 보장 ❌
* `@Order`로 제어 가능

```java
@Order(1)
@Bean
ApplicationRunner runner1() { ... }

@Order(2)
@Bean
ApplicationRunner runner2() { ... }
```

숫자가 **작을수록 먼저 실행**

---

### 8. 실무 주의사항 ⚠️

#### ❌ 무거운 작업 금지

* 대량 데이터 삽입
* 오래 걸리는 외부 API 호출
  → 서버 기동 지연

#### ❌ 운영 환경 자동 실행 주의

* 실수로 운영 DB에 더미 데이터 삽입 가능

➡️ 보통 프로파일로 제한

```java
@Profile("dev")
@Bean
ApplicationRunner runner() { ... }
```

---

### 9. 한 줄 요약

> **ApplicationRunner는 Spring Boot가 완전히 시작된 직후, 한 번만 자동 실행되는 초기화 전용 인터페이스다.**