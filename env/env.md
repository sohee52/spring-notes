# env

## 1. application.yml

* 코드 내에 직접 값을 적지 않고 환경 변수(`Environment Variable`)를 참조한다.
* **형식:** `${변수명}`
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          kakao:
            client-id: ${KAKAO_CLIENT_ID}

```



## 2. .env (로컬 관리용)

* 로컬 개발 시 사용할 실제 키 값을 정의하는 파일.
* **주의:** 보안을 위해 반드시 `.gitignore`에 등록하여 커밋되지 않도록 한다.
```properties
KAKAO_CLIENT_ID=abc12345...
KAKAO_CLIENT_SECRET=xyz98765...

```



## 3. Configuration (IntelliJ 실행 설정)

* Spring Boot 실행 시 환경 변수를 주입하는 방법이다.
* **방법:** `Run/Debug Configurations` -> `Environment variables` 항목에 값 입력.
* **형식:** 세미콜론(;)으로 구분하며, 공백 없이 작성한다.
* `KAKAO_CLIENT_ID=값;KAKAO_CLIENT_SECRET=값`


* *(참고)* `.env` 파일 자체를 자동으로 읽어오게 하려면 **EnvFile** 같은 IDE 플러그인을 사용하면 편리하다.