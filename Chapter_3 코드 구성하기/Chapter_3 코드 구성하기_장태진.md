코드 구성하기
==

패키지 구조
--
계층형

buckpal - domain/Account/Repository/Service - persistence/RepositoryImpl - web/Controller

1. 기능 조각이나 특성을 구분 짓는 패키지 경계가 없음.
2. 기능 추가시 각 계층별 패키지에 추가가 되고, 연관되지 않은 기능들끼리 엮일 가능성 농후함.
3. 어떤 유스케이스들을 제공하는지 파악할 수 없음. 네이밍이 기능을 표현하지 않기 때문.

기능으로 구성하기 

buckpal - account - Account/AccountController/AccountRepository/SendMoneyService 등

1. 아키텍처가 명확하게 보이지 않는다.
2. 송금하기 유스케이스를 명확한 서비스네이밍으로 찾을 수 있다. > 소리치는 아키텍처(screaming architecture)
3. 가시성이 많이 떨어짐. 
4. 도메인코드가 영속성코드에 의존하는것을 막을 수 없음.(같은 패키지 private)

아키텍처적으로 표현력 있는 패키지 구조
```html


buckpal - account 
            -adapter
                -in
                    -web
                        -AccountController
                -out
                    -persistence
                        -AccountPersistenceAdapter
                        -SpringDataAccountRepository
            -domain
                -Account
                -Activity
            -application
                -SendMoneyService
                -port
                    -in
                        =SendMoneyUseCase
                    -out
                        -LoadAccountPort
                        -UpdateAccountStatePort
```

1. 육각형을 이해한다면 매우 직관적임. 
2. 기능 추가시 연관된 작업을 비교적 쉽게 시작할 수 있음.
3. 아키텍처-코드 갭 or 모델-코드 갭을 효과적으로 다룰 수 있음.
4. 의존성에 대해 명확한 이해가 필요함.
5. DDD의 바운디드 컨텍스트 와 잘 어울림.

