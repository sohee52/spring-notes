
# Spring 모듈 간 API 통신 정리

## 1. 개념: 모듈 간 통신 방식

### Event vs API 호출

| 구분 | Event | API 호출 |
|------|-------|----------|
| 통신 방식 | 비동기 (fire & forget) | 동기 (request-response) |
| 응답 여부 | ❌ 받을 수 없음 | ✅ 받을 수 있음 |
| 사용 시점 | "이런 일이 일어났어" 알림 | "이 데이터 줘" 요청 |
| 결합도 | 낮음 | 낮음 (HTTP 통신) |
| 예시 | 글 생성됨 알림 | 보안 팁 문자열 요청 |

### 왜 직접 메서드 호출 안 하고 API?

```java
// ❌ 직접 호출 - 결합도 높음
private final MemberFacade memberFacade;
memberFacade.getRandomSecureTip();

// ✅ API 호출 - 결합도 낮음
private final MemberApiClient memberApiClient;
memberApiClient.getRandomSecureTip();
```

- 모듈 간 의존성 제거
- 나중에 마이크로서비스로 분리 시 URL만 변경하면 됨

---

## 2. 계층 구조 (Hexagonal Architecture)

```
boundedContext/
└── member/
    ├── in/      ← 인바운드 (외부 → 내부) : Controller
    ├── app/     ← 애플리케이션 : Facade, UseCase
    ├── domain/  ← 도메인 : Entity
    └── out/     ← 아웃바운드 (내부 → 외부) : Repository, ApiClient
```

---

## 3. API 제공하는 쪽 (Server)

### Controller 작성

```java
package com.back.boundedContext.member.in;

import lombok.RequiredArgsConstructor;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController  // JSON 반환하는 REST API 컨트롤러
@RequestMapping("/member/api/v1/members")  // 기본 URL 경로
@RequiredArgsConstructor
public class ApiV1MemberController {
    
    private final MemberFacade memberFacade;

    @GetMapping("/randomSecureTip")  
    // → GET http://localhost:8080/member/api/v1/members/randomSecureTip
    @Transactional(readOnly = true)
    public String getRandomSecureTip() {
        return memberFacade.getRandomSecureTip();
    }
}
```

### 주요 어노테이션

| 어노테이션 | 설명 |
|-----------|------|
| `@RestController` | `@Controller` + `@ResponseBody`, JSON 반환 |
| `@RequestMapping` | 클래스 레벨 기본 URL 설정 |
| `@GetMapping` | GET 요청 매핑 |
| `@PostMapping` | POST 요청 매핑 |
| `@Transactional(readOnly = true)` | 읽기 전용 트랜잭션 (성능 최적화) |

---

## 4. API 호출하는 쪽 (Client)

### RestClient 사용법 (Spring 6.1+)

```java
package com.back.boundedContext.member.out;

import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class MemberApiClient {
    
    private final RestClient restClient = RestClient.builder()
            .baseUrl("http://localhost:8080/member/api/v1")
            .build();

    // GET 요청
    public String getRandomSecureTip() {
        return restClient.get()
                .uri("/members/randomSecureTip")
                .retrieve()
                .body(String.class);
    }
    
    // POST 요청 예시
    public MemberDto createMember(MemberCreateRequest request) {
        return restClient.post()
                .uri("/members")
                .contentType(MediaType.APPLICATION_JSON)
                .body(request)
                .retrieve()
                .body(MemberDto.class);
    }
    
    // PathVariable 포함 예시
    public MemberDto getMember(Long id) {
        return restClient.get()
                .uri("/members/{id}", id)
                .retrieve()
                .body(MemberDto.class);
    }
}
```

### RestClient 메서드 체이닝

```java
restClient
    .get() / .post() / .put() / .delete()  // HTTP 메서드
    .uri("/path")                           // 엔드포인트
    .header("key", "value")                 // 헤더 추가
    .contentType(MediaType.APPLICATION_JSON) // Content-Type
    .body(requestObject)                    // 요청 본문 (POST/PUT)
    .retrieve()                             // 응답 받기
    .body(ResponseType.class);              // 응답 본문 변환
```

---

## 5. 사용하는 쪽 (UseCase)

```java
@Service
@RequiredArgsConstructor
public class PostWriteUseCase {
    
    private final PostRepository postRepository;
    private final EventPublisher eventPublisher;
    private final MemberApiClient memberApiClient;  // API Client 주입

    public RsData<Post> write(Member author, String title, String content) {
        // 1. 글 저장
        Post post = postRepository.save(new Post(author, title, content));

        // 2. Event 발행 (응답 필요 없음)
        eventPublisher.publish(new PostCreatedEvent(new PostDto(post)));

        // 3. API 호출 (응답 필요함)
        String randomSecureTip = memberApiClient.getRandomSecureTip();

        // 4. 결과 반환
        return new RsData<>("201-1", 
                "%d번 글이 생성되었습니다. 보안 팁 : %s".formatted(post.getId(), randomSecureTip), 
                post);
    }
}
```

---

## 6. 전체 흐름도

```
[Post 모듈]                              [Member 모듈]
     │                                        │
PostWriteUseCase                              │
     │                                        │
     ├── postRepository.save()                │
     │                                        │
     ├── eventPublisher.publish()             │
     │   (비동기, 응답 없음)                    │
     │                                        │
     └── memberApiClient                      │
              .getRandomSecureTip()           │
                    │                         │
                    │  HTTP GET               │
                    │  /member/api/v1/        │
                    │  members/randomSecureTip│
                    ├────────────────────────→│
                    │                  ApiV1MemberController
                    │                         │
                    │                  memberFacade
                    │                  .getRandomSecureTip()
                    │                         │
                    │←────────────────────────┤
                    │  "보안 팁 문자열"         │
                    │                         │
     return RsData  │                         │
```

---

## 7. 코드 템플릿

### API 제공 (Controller)

```java
@RestController
@RequestMapping("/{모듈명}/api/v1/{리소스}")
@RequiredArgsConstructor
public class ApiV1{리소스}Controller {
    
    private final {리소스}Facade facade;

    @GetMapping("/{endpoint}")
    @Transactional(readOnly = true)
    public {ResponseType} get{Something}() {
        return facade.get{Something}();
    }
}
```

### API 호출 (Client)

```java
@Service
public class {모듈명}ApiClient {
    
    private final RestClient restClient = RestClient.builder()
            .baseUrl("http://localhost:8080/{모듈명}/api/v1")
            .build();

    public {ResponseType} get{Something}() {
        return restClient.get()
                .uri("/{리소스}/{endpoint}")
                .retrieve()
                .body({ResponseType}.class);
    }
}
```

---

## 8. 참고: RestClient vs RestTemplate vs WebClient

| 구분 | RestClient | RestTemplate | WebClient |
|------|------------|--------------|-----------|
| Spring 버전 | 6.1+ | 3.0+ | 5.0+ |
| 방식 | 동기 | 동기 | 비동기/동기 |
| 상태 | ✅ 권장 | ⚠️ 유지보수 모드 | ✅ 리액티브 환경 권장 |
| 특징 | 모던한 fluent API | 레거시 | Reactor 기반 |

---

이 정도면 나중에 다시 짤 때 참고하기 좋을 거야! 추가로 필요한 부분 있으면 말해줘.