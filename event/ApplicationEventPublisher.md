## ApplicationEventPublisher

`ApplicationEventPublisher`는 **Spring에서 “이벤트를 발생시키는 역할”**을 하는 인터페이스다.
핵심은 **“어떤 일이 일어났음을 알리기만 하고, 누가 처리하는지는 신경 쓰지 않는다”**는 점이다.

---

### 1. 왜 필요한가

* **비즈니스 로직과 부가 로직을 분리**하기 위해 사용한다.
* 어떤 작업이 끝났을 때
  → *로그 저장*, *알림 전송*, *후처리 작업* 등을
  **직접 호출하지 않고 이벤트로 알린다.**

즉,

> “이 일이 끝났어”라고 알리기만 하고
> “그걸 누가 어떻게 처리할지는 모른다”

---

### 2. 기본 개념 구조

```
[이벤트 발생자]
   |
   | publishEvent()
   ↓
[Spring 이벤트 시스템]
   |
   ↓
[이벤트 리스너 (@EventListener / @TransactionalEventListener)]
```

* **ApplicationEventPublisher** → 이벤트 발행자
* **Listener** → 이벤트를 받아 처리하는 쪽

---

### 3. 사용 예시

#### 1️⃣ 이벤트 발행

```java
@RequiredArgsConstructor
@Service
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    public void completeOrder(Long orderId) {
        // 주문 완료 로직
        eventPublisher.publishEvent(new OrderCompletedEvent(orderId));
    }
}
```

#### 2️⃣ 이벤트 클래스

```java
public record OrderCompletedEvent(Long orderId) {}
```

#### 3️⃣ 이벤트 수신(리스너)

```java
@Component
public class OrderEventListener {

    @EventListener
    public void handle(OrderCompletedEvent event) {
        // 알림 전송, 로그 기록 등
    }
}
```

---

### 4. @TransactionalEventListener 와의 관계

`ApplicationEventPublisher`는 **이벤트를 “언제” 발생시킬지만 결정**한다.
실제로 **언제 실행될지는 리스너의 설정**에 달려 있다.

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void handle(OrderCompletedEvent event) {
    // 트랜잭션 커밋 이후 실행
}
```

* 트랜잭션 **커밋 후**
* **롤백되면 실행 안 됨**

👉 이게 실무에서 매우 중요하다.

---

### 5. 직접 메서드 호출 vs 이벤트 방식

#### ❌ 직접 호출

```java
orderService.complete();
notificationService.send();
logService.save();
```

* 강한 결합
* 수정 시 영향 범위 큼

#### ✅ 이벤트 방식

```java
orderService.complete();
eventPublisher.publishEvent(new OrderCompletedEvent());
```

* 느슨한 결합
* 확장 쉬움
* 리스너 추가/삭제 자유로움

---

### 6. 정리

* **ApplicationEventPublisher**
  → *“Spring 애플리케이션 내부에서 이벤트를 발생시키는 도구”*
* **역할**
  → 발생 사실만 알리고, 처리는 리스너에게 위임
* **목적**
  → 관심사 분리, 결합도 감소, 확장성 증가