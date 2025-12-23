## `@TransactionalEventListener(phase = AFTER_COMMIT)`

### 역할

* **트랜잭션 이벤트를 “커밋이 완료된 후”에 처리**하도록 지정하는 애노테이션
* 트랜잭션이 **정상적으로 commit 되었을 때만** 이벤트 리스너가 실행됨

### `phase = AFTER_COMMIT` 의미

* 트랜잭션 생명주기 중 **commit 이후**에 실행
* 즉, **DB 커밋 성공 이후에만 실행**
* rollback 발생 시 → **절대 실행되지 않음**

### 왜 AFTER_COMMIT을 쓰는가?

* DB 상태가 **확정된 이후에만 실행되어야 하는 로직**을 안전하게 처리하기 위함
* 예시:

  * 이메일 발송
  * 알림 전송
  * 외부 API 호출
  * 로그/이력 저장

> ❗ commit 전에 실행하면
> → 롤백 시 실제 DB에는 반영되지 않았는데 외부 시스템은 동작해버리는 문제가 생김

---

## `@Transactional(propagation = REQUIRES_NEW)`

### 역할

* **기존 트랜잭션과 완전히 분리된 새로운 트랜잭션을 강제로 생성**
* 호출한 쪽 트랜잭션과 **성공/실패가 서로 영향 없음**

### `REQUIRES_NEW` 동작 방식

* 기존 트랜잭션이 존재하면 → **일시 중단**
* 새로운 트랜잭션 시작
* 메서드 종료 시 → **독립적으로 commit / rollback**
* 즉, **메인 트랜잭션과 완전히 분리**되어 동작

### 왜 REQUIRES_NEW를 쓰는가?

* 이벤트 처리 로직이 **실패하더라도**
  * 이미 commit 된 메인 비즈니스 로직에는 영향을 주지 않기 위해
* 또는
  * 이벤트 처리 결과를 **반드시 DB에 남겨야 할 때**

---

## 두 애노테이션을 같이 쓰는 이유

```java
@TransactionalEventListener(phase = AFTER_COMMIT)
@Transactional(propagation = REQUIRES_NEW)
```

> **메인 트랜잭션이 성공적으로 커밋된 이후,
> 그와 완전히 분리된 새 트랜잭션에서 이벤트 로직을 실행한다**

### 이 조합이 보장하는 것

1. 메인 트랜잭션이 **성공한 경우에만 실행**

2. 이벤트 로직 실패 시에도

   * 메인 트랜잭션 결과는 **절대 롤백되지 않음**

3. 이벤트 로직 내부 DB 작업은

   * **자체 트랜잭션으로 안전하게 처리**

### 비유

* 먼저 큰 약속(메인 작업)이 성공해서 “완료”된 뒤에만 실행돼요.
* 그다음 일은 따로 작은 약속을 새로 만들어서 진행해요.
* 그래서 뒤에 하는 일이 실패해도 앞에서 끝난 일은 망가지지 않아요.

---

## 언제 쓰는 게 적절한가?

#### 👍 적절한 경우

* 회원 가입 후 이메일 발송
* 주문 완료 후 알림 생성
* 결제 성공 후 로그/이력 저장
* 외부 시스템 연동 (Kafka, Slack, Webhook 등)

#### 👎 주의할 점

* REQUIRES_NEW는 트랜잭션을 하나 더 생성하므로

  * **남용 시 성능 비용 증가**
* 이벤트 로직에서 예외 발생 시

  * 해당 이벤트 트랜잭션은 rollback 됨 (메인은 영향 없음)

---

## 예시

```java
import com.back.boundedContext.member.domain.Member;
import com.back.boundedContext.member.app.MemberService;
import com.back.shared.post.event.PostCommentCreatedEvent;
import com.back.shared.post.event.PostCreatedEvent;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.event.TransactionalEventListener;

import static org.springframework.transaction.annotation.Propagation.REQUIRES_NEW;
import static org.springframework.transaction.event.TransactionPhase.AFTER_COMMIT;

@Component
@RequiredArgsConstructor
public class MemberEventListener {
    private final MemberService memberService;

    @TransactionalEventListener(phase = AFTER_COMMIT)
    @Transactional(propagation = REQUIRES_NEW)
    public void handle(PostCreatedEvent event) {
        Member member = memberService.findById(event.getPost().getAuthorId()).get();

        member.increaseActivityScore(3);
    }

    @TransactionalEventListener(phase = AFTER_COMMIT)
    @Transactional(propagation = REQUIRES_NEW)
    public void handle(PostCommentCreatedEvent event) {
        Member member = memberService.findById(event.getPostComment().getAuthorId()).get();

        member.increaseActivityScore(1);
    }
}
```