# RsData_vs_ResponseEntity

```java
package com.back.global.rsData;

import lombok.AllArgsConstructor;
import lombok.Getter;

@AllArgsConstructor
@Getter
public class RsData<T> {
    private final String resultCode;
    private final String msg;
    private final T data; // Post, Member 등 다양한 타입이 올 수 있으므로 제네릭으로 처리

    public RsData(String resultCode, String msg) {
        this(resultCode, msg, null);
    }
}
```

위 코드는 ResponseEntity랑 비슷한거야?

---

네, **비슷한 역할을 하지만, 사용 목적과 범위가 조금 다릅니다.**

사용자님의 직관대로 두 클래스 모두 **"응답을 클라이언트에게 전달하는 역할"**을 한다는 점에서는 비슷합니다. 하지만 **`ResponseEntity`는 'HTTP 표준'** 에, **`RsData`는 '우리 앱의 비즈니스 규칙'** 에 더 초점이 맞춰져 있습니다.

이해하기 쉽게 비교해 드릴게요.

---

### 1. 차이점 핵심 요약

| 구분 | **ResponseEntity** (Spring 기본 제공) | **RsData** (직접 만든 커스텀 클래스) |
| --- | --- | --- |
| **주요 목적** | **HTTP 프로토콜** 제어 | **응답 데이터의 구조** 통일 |
| **포함 내용** | HTTP 상태 코드(200, 404 등), 헤더, 바디 | 성공/실패 코드(S-1, F-1 등), 메시지, 실제 데이터 |
| **비유** | **택배 상자** (송장, 배송 상태, 박스 포장) | **상자 안의 내용물** (편지, 선물, 설명서) |

### 2. 왜 `RsData` 같은 클래스를 따로 만들까?

`ResponseEntity`만 사용하면, 실제 데이터(`Post`, `Member` 등)만 덜렁 보내게 되는 경우가 많습니다. 이렇게 되면 프론트엔드 입장에서는 요청이 실패했는지, 성공했다면 어떤 메시지를 띄워야 하는지 파악하기가 불친절할 수 있습니다.

**`RsData`를 사용하면 다음과 같은 장점이 있습니다.**

1. **일관성:** 모든 API 응답이 `{ resultCode, msg, data }` 형태로 통일됩니다.
2. **명확한 소통:** HTTP 상태 코드는 200(성공)이라도, 비즈니스 로직상 실패(예: "포인트 부족")인 경우를 `resultCode`로 명확히 구분할 수 있습니다.
3. **유연성:** 데이터(`T`)가 무엇이든 같은 껍데기에 담아서 보낼 수 있습니다.

### 3. 함께 사용하는 패턴 (Best Practice)

실무에서는 보통 **`ResponseEntity` 안에 `RsData`를 담아서** 리턴합니다.

**예시 코드:**

```java
@GetMapping("/{id}")
public ResponseEntity<RsData<Post>> getPost(@PathVariable Long id) {
    Post post = postService.findById(id);

    if (post == null) {
        // HTTP 상태는 404, 바디에는 실패 이유를 담음
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new RsData<>("F-1", "게시물이 존재하지 않습니다."));
    }

    // HTTP 상태는 200, 바디에는 성공 코드와 데이터 담음
    return ResponseEntity
            .ok(new RsData<>("S-1", "조회 성공", post));
}

```

**실제 클라이언트가 받는 JSON 결과:**

```json
{
  "resultCode": "S-1",
  "msg": "조회 성공",
  "data": {
    "id": 1,
    "title": "안녕하세요",
    "content": "반갑습니다"
  }
}

```

### 요약

* **`ResponseEntity`**: 웹 통신의 **겉봉투** (HTTP 상태, 헤더 담당)
* **`RsData`**: 봉투 안에 담긴 **규격화된 편지지** (실제 데이터와 결과 메시지 담당)