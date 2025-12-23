## @ManyToOne 또는 @OneToMany

### mappedBy 속성
- `@OneToMany` 관계에서 `mappedBy` 속성은 양방향 관계에서 주체가 아닌 쪽(즉, "many" 쪽)을 지정하는 데 사용됩니다.
- 이는 외래 키가 어느 테이블에 있는지를 나타내며, 주로 데이터베이스 설계에서 중요한 역할을 합니다.

### cascade 속성
- `cascade` 속성은 엔티티 간의 관계에서 특정 작업(예: 저장, 삭제 등)이 연쇄적으로 적용되도록 설정하는 데 사용됩니다.
- 예를 들어, 부모 엔티티가 삭제될 때 자식 엔티티도 함께 삭제되도록 설정할 수 있습니다.
- 주요 cascade 옵션:
  - `CascadeType.PERSIST`: 부모 엔티티가 저장될 때 자식 엔티티도 함께 저장됩니다.
  - `CascadeType.MERGE`: 부모 엔티티가 병합될 때 자식 엔티티도 함께 병합됩니다.
  - `CascadeType.REMOVE`: 부모 엔티티가 삭제될 때 자식 엔티티도 함께 삭제됩니다.
  - `CascadeType.REFRESH`: 부모 엔티티가 새로 고침될 때 자식 엔티티도 함께 새로 고침됩니다.
  - `CascadeType.DETACH`: 부모 엔티티가 분리될 때 자식 엔티티도 함께 분리됩니다.

### orphanRemoval 속성
- `orphanRemoval` 속성은 부모 엔티티와의 관계가 끊어진 자식 엔티티를 자동으로 삭제하는 데 사용됩니다.
- `orphanRemoval=true`로 설정하면, 자식 엔티티가 부모 엔티티에서 제거될 때 자동으로 삭제됩니다.

### fetch 속성
- `fetch` 속성은 연관된 엔티티를 로드하는 방식을 지정합니다.
- 주요 fetch 옵션:
  - `FetchType.EAGER`: 연관된 엔티티를 즉시 로드합니다.
  - `FetchType.LAZY`: 연관된 엔티티를 필요할 때까지 로드하지 않습니다.
  - 기본적으로 `@ManyToOne`은 `EAGER`이고, `@OneToMany`는 `LAZY`입니다.
  - EAGER 로딩은 즉시 데이터를 가져오므로 편리하지만, 불필요한 데이터 로드로 인해 성능 저하가 발생할 수 있습니다.
  - LAZY 로딩은 성능 최적화에 유리하지만, 프록시 객체로 인해 예상치 못한 동작이 발생할 수 있으므로 주의가 필요합니다.

