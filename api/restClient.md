# restClient 관련 코드

```java
package com.back.shared.post.out;

import com.back.shared.post.dto.PostDto;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

import java.util.List;

@Service
public class PostApiClient {
    private final RestClient restClient = RestClient.builder()
            .baseUrl("http://localhost:8080/api/v1/post")
            .build();

    public List<PostDto> getItems() {
        return restClient.get()
                .uri("/posts")
                .retrieve()
                .body(new ParameterizedTypeReference<>() {
                });
    }

    public PostDto getItem(int id) {
        return restClient.get()
                .uri("/posts/%d".formatted(id))
                .retrieve()
                .body(new ParameterizedTypeReference<>() {
                });
    }
}
```

## 전체 역할 요약

`PostApiClient`는
**외부(혹은 다른 모듈)의 Post API를 HTTP로 호출하는 클라이언트 클래스**입니다.

* Spring `@Service` → 비즈니스 레이어에서 주입해서 사용
* Spring 6의 **`RestClient`** 사용 (구 `RestTemplate` 대체)
* Post 관련 API를 호출해서 `PostDto` 또는 `List<PostDto>`로 받음

---

## 패키지 & import

```java
package com.back.shared.post.out;
```

* `out` → 보통 **외부 시스템 호출 (Outbound Port)** 의미
* DDD / Hexagonal Architecture에서 자주 쓰는 네이밍

```java
import org.springframework.web.client.RestClient;
import org.springframework.core.ParameterizedTypeReference;
```

* `RestClient` : HTTP 요청용 클라이언트
* `ParameterizedTypeReference` : **제네릭 타입(List<PostDto>) 역직렬화용**

---

## 클래스 선언

```java
@Service
public class PostApiClient {
```

* Spring Bean으로 등록
* 다른 서비스에서 `@Autowired` / 생성자 주입 가능

---

## RestClient 생성

```java
private final RestClient restClient = RestClient.builder()
        .baseUrl("http://localhost:8080/api/v1/post")
        .build();
```

### 의미

* 이 클라이언트는 항상
  `http://localhost:8080/api/v1/post` 를 기준 URL로 사용
* 이후 `.uri("/posts")` →
  실제 요청은
  `http://localhost:8080/api/v1/post/posts`

⚠️ **주의**

* 실무에서는 보통:

  * `application.yml`에서 baseUrl 관리
  * 생성자 주입으로 `RestClient` 받음
* 현재 코드는 **간단한 예제/로컬용 구조**

---

## getItems()

```java
public List<PostDto> getItems() {
    return restClient.get()
            .uri("/posts")
            .retrieve()
            .body(new ParameterizedTypeReference<>() {
            });
}
```

### 동작 순서

1. `GET /api/v1/post/posts`
2. HTTP 요청 전송
3. 응답 Body를

   * `List<PostDto>`로 변환
4. 그 결과를 반환

### 왜 `ParameterizedTypeReference`를 쓰나?

* Java 제네릭은 런타임에 타입 소거됨
* `List<PostDto>` 같은 **중첩 제네릭 타입**은

  ```java
  body(List.class)
  ```

  로는 정확히 변환 불가

→ 그래서 타입 정보를 유지하기 위해 사용

---

## getItem(int id)

```java
public PostDto getItem(int id) {
    return restClient.get()
            .uri("/posts/%d".formatted(id))
            .retrieve()
            .body(new ParameterizedTypeReference<>() {
            });
}
```

### 동작

* 예: `id = 3`
* 요청:

  ```
  GET /api/v1/post/posts/3
  ```
* 응답 Body → `PostDto`로 변환 후 반환

### 여기서도 `ParameterizedTypeReference`를 쓴 이유

* 사실 **단일 객체는**

  ```java
  .body(PostDto.class)
  ```

  로 써도 됨
* 지금은 **통일성**을 위해 같은 방식 사용한 것

---

## 이 코드의 특징 정리

### 👍 장점

* Spring 6 표준 `RestClient` 사용 (최신)
* 타입 안정성 확보
* 코드 간결함

### ⚠️ 아쉬운 점 (정확히 말하면 개선 포인트)

* baseUrl 하드코딩
* 예외 처리 없음

  * 404 / 500 시 바로 예외 발생
* `RestClient`를 필드에서 직접 생성

  * 테스트 / 설정 확장성 떨어짐

---

## 한 줄 요약

> `PostApiClient`는
> **Post API를 호출해서 JSON 응답을 `PostDto`로 변환해주는 HTTP 클라이언트 서비스**다.