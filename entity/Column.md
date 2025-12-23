## Column

### columnDefinition
- columnDeifinition은 컬럼의 SQL 정의를 직접 지정할 때 사용됩니다.

```java
@Column(columnDefinition = "TEXT")
private String description;
```
- 위 예시에서 `description` 필드는 데이터베이스에서 `TEXT` 타입으로 생성됩니다.

- Text말고도 다양한 SQL 타입을 지정할 수 있습니다.
- 예를 들어:
  - `VARCHAR(255)`
  - `INT`
  - `BOOLEAN`
  - `DATE`
  - `TIMESTAMP`