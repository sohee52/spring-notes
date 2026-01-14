## Constructor Annotation

### NoArgConstructor
- 인자가 없는 기본 생성자를 자동으로 생성합니다.

### AllArgConstructor
- 모든 필드를 인자로 받는 생성자를 자동으로 생성합니다.

### RequiredArgConstructor
- `final` 또는 `@NonNull`로 선언된 필드만을 인자로 받는 생성자를 자동으로 생성합니다.

#### RequiredArgConstructor 예시
```java
@RequiredArgsConstructor
public class User {
    private final Long id;
    @NonNull
    private String name;
    private int age;
}

// 생성된 생성자
public User(Long id, String name) {
    this.id = id;
    if (name == null) {
        throw new NullPointerException("name is marked non-null but is null");
    }
    this.name = name;
}
```
- `id`와 `name` 필드는 `final` 및 `@NonNull`로 선언되어 있으므로 생성자에 포함됩니다.
- `age` 필드는 포함되지 않습니다.
- `name` 필드에 대해 null 체크가 자동으로 추가됩니다.
