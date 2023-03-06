##### Intro
* 애플리케이션이 시작될 때 클래스를 인스턴스화하고 묶기 위해서 의존성 주입 메커니즘을 이용한다
* 평범한자바, 스프링, 스프링부트 프레임워크 각각 어떻게 하나?
##### 왜 조립까지 신경 써야 할까?
> 왜 유스케이스와 어댑터를 그냥 필요할 때 인스턴스화 하면 안 되는 걸까? (? 누가 안된다고 했나?? -> 아 저자의 의도는 유스케이스와 어댑터를 분리하지 않고 함께 인스턴스화화면 안된다고 한것)
 1. 코드 의존성이 올바른 방향을 가리키게 하기 위해서 -> 모든 의존성은 안쪽으로
    * 애플리케이션의 도메인 코드 방향으로 향해야 도메인 코드가 바깥 계층의 변경으로부터 안전하다
    * 유스케이스는 인터페이스만 알아야 하고, 런타임에 이 인터페이스의 구현을 제공 받아야 한다 -> 올바른 의존성 방향 유지
 2. 테스트 하기 쉽다.
    * 한 클래스가 필요로 하는 모든 객체를 생성자로 전달할 수 있다면 -> Q. 네??
> 객체 인스턴스를 생성할 책임은 누구에게 있을까? 그리고 어떻게 의존성 규칙을 어기지 않으면서 그렇게 할 수 있을까? -? Q. 갑자기 책임이요?
* 아키텍처에 대해 중립적이고 인스턴스 생성을 위해 모든 클래스에 대한 의존성을 가지는 설정 컴포넌트가 있어야 한다.
* 설정 컴포넌트 위치 : 의존성 규칙에 정의된 대로 모든 내부 계층에 접근할 수 있는 원의 가장 바깥쪽에 위치
* 설정 컴포넌트는 우리가 제공한 조각들로 애플리케이션을 조립하는 것을 책임짐
* 컴포넌트의 역할
  * 웹 어댑터 인스턴스 생성
  * HTTP 요청이 실제로 웹 어댑터로 전달되도록 보장
  * 유스케이스 인스턴스 생성
  * 웹 어댑터에 유스케이스 인스턴스 제공
  * 영속성 어댑터 인스턴스 생성
  * 유스케이스에 영속성 어댑터 인스턴스 제공
  * 영속성 어댑터가 실제로 데이터베이스에 접근할 수 있도록 보장
  * 설정 파일이나 커맨드라인 파라미터 등과 같은 설정 파라미터의 소스에도 접근할 수 있어야 함
* 위 여러가지 책임(변경할 이유)은 단일 책임 원칙을 위반한 것이 맞지만 애플리케이션의 나머지 부분을 깔끔하게 유지하고 싶다면 이처럼 구성요소들을 연결하는 바깥쪽 컴포넌트가 필요
* 이 컴포넌트는 작동하는 애플리케이션으로 조립하기 위해 애플리케이션을 구성하는 모든 움직이는 부품을 알아야 한다.
  
##### 평범한 코드로 조립하기
```java
class Application {
    // 메인 메서드 : 애플리케이션 시작
    public static void main(String[] args) {
        
        AccountRepository accountRepository = new AccountRepository();
        AcitivityRepository acitivityRepository = new ActivityRepository();
        
        AccountPersistenceAdapter accountPersistenceAdapter = 
                new AccountPersistenceAdapter(accountRepository, acitivityRepository);
        
        SendMoneyUseCase sendMoneyUseCase = 
                new SendMoneyUseCaseService(
                        accountPersistenceAdapter, // LoadAccountPort
                        accountPersistenceAdapter // UpdateAccountStatePort
                );
        
        SendMoneyController sendMoneyController = 
                new SendMoneyController(sendMoneyUseCase);
        
        // 웹 컨트롤러를 HTTP로 노출하는 신비한 메서드인 startProcessing WebRequests()를 호출
        startProcessingWebRequest(sendMoneyController);
    }
}
```
* 위 코드 방식 단점
  * 무수히 많은 웹 컨트롤러, 유스케이스, 영속성 어댑터가 추가되어져야 함.
  * 추가하는 클래스 모두 public이어야 함. 유스케이스가 영속성 어댑터에 직접 접근하는 것을 막지 못하게 됨.
  * 웹과 데이터베이스 환경을 지원하는 스프링을 이용하면 위와 같은 지저분한 작업이나 startProcessingWebRequest() 메서드 같은 것 구현할 필요 없음.
##### 스프링의 클래스패스 스캐닝으로 조립하기
* 애플리케이션 컨텍스트(application context) 
  * 스프링 프레임워크를 이용해서 애플리케이션을 조립한 결과물
  * 이 컨텍스트는 애플리케이션을 구성하는 모든 객체(자바 용어로 bean)를 포함한다
* 스프링 컨텍스트를 조립하기 위한 방법
  1. 클래스패스 스캐닝(classpath scanning)
    * 스프링은 클래스패스 스캐닝으로 클래스패스에서 접근 가능한 모든 클래스를 확인해서 @Component 애너테이션이 붙은 클래스를 찾는다.
    * @Component가 붙은 각 클래스의 객체를 생성 (필요한 모든 필드를 인자로 받는 생성자를 가지고 있어야 함)
    ```java
    @ReqiredArgsConstructor // 필요한 모든 필드를 인자로 받는 생성자
    @Component
    class AccountPersistenceAdaptor implements LoadAccountPort, UpdateAccountStatePort {
    
        private final AccountRepository accountRepository;
        private final ActivityRepository activityRepository;
        private final AccountMapper accountMapper;
  
        @Override
        public Account loadAccount(
            AccountId accountId,
            LocalDateTime baselineDate)  
        {}
        
        @Override
        public void updateActivities(Account account) {}
    } 
    ```
    * 스프링은 이 생성자를 찾아서 생성자의 인자로 사용된 @Component가 붙은 클래스들을 찾고, 이 클래스들의 인스턴스를 만들어 애플리케이션 컨텍스트에 추가
    * 필요한 모든 객체들이 모두 생성되면 AccountPersistenceAdapter의 생성자를 호출하고 생성된 객체도 마찬가지로 애플리케이션 컨텍스트에 추가
    * 애너테이션을 직접 만들수도 있음
      ```java
      @Target({ElementType.Type})
      @Retention(RetentionPolicy.RUNTIME)
      @Document
      @Component
      public @interface PersistenceAdapter {
        @AliasFor(annotaion = Component.class)
        String value() default "";
      }
      ```
      * 메타 애너테이션 / @Component를 포함하고 있어서 스프링이 클래스패스 스캐닝을 할 때 인스턴스를 생성할 수 있도록 한다.
      * 아키텍처를 더 쉽게 파악할 수 있게 됨
    * 하지만 클래스패스 스캐닝의 단점은..
      * 클래스에 프레임워크에 특화된 애너테이션을 붙여야 한다는 점에서 침투적이다.
      * 라이브러리 사용자가 스프링 프레임워크의 의존성에 엮이게 됨
##### 스프링의 자바 컨피그로 조립하기
* 이 방식은 애플리케이션 컨텍스트에 추가할 빈을 생성하는 설정 클래스를 만듦
```java
@Configuration // 이 클래스가 스프링의 클래스패스 스캐닝에서 발견해야 할 설정 클래스임을 표시, 여전히 클래스패스 스캐닝을 이용하나 모든 빈을 가져오는 대신 설정 클래스만 선택
@EnableJpaRepositories 
class PersistenceAdapterConfiguration {
    
    @Bean // 설정 클래스 내의 @Bean이 붙은 팩터리 메서드를 통해 빈이 생성. 스프링이 자동으로 Adapter 클래스 내부에서 생성자 입력으로 받는 것을 팩터리 메서드에 입력으로 제공
    AccountPersistenceAdapter accountPersistenceAdapter(
            AccountRepository accountRepository,
            ActivityRepository activityRepository,
            AccountMapper accountMapper) {
      
        return new AccountPersistenceAdapter(
              accountRepository,
              activityRepository,
              accountMapper
      );  
    }
    
    @Bean
    AccountMapper accountMapper() {
        return new AccountMapper();
    }
    
}
```
* @EnableJpaRepositories 
  * 사용하면 스프링이 자동으로 repository를 직접 생성해서 제공
  * 스프링부트가 이 애너테이션을 발견하면 자동으로 우리가 정의한 모든 스프링 데이터 리포지토리 인터페이스의 구현체를 제공
  * 설정클래스가 아닌 메인 애플리케이션에 붙이는 경우
    * 애플리케이션을 시작할 때마다 JPA를 활성화해서 영속성이 실질적으로 필요 없는 테스트에서 애플리케이션을 실행할 때도 JPA 리포지토리를 활성화하게 됨
    * 따라서 이러한 '기능 애너테이션'을 별도의 설정 '모듈'로 옮기는 편이 애플리케이션을 더 유연하게 만들고, 항상 모든 것을 한꺼번에 시작할 필요 없게 해줌.
* PersistenceAdapterconfiguration 클래스를 이용해서 영속성 계층에서 필요로 하는 모든 객체를 인스턴스화하는 매우 한정적인 범위의 영속성 모듈을 만들게 됨.
##### 유지보수 가능한 소프트웨어를 만드는 데 어떻게 도움이 될까?
* 스프링과 스프링부트(비슷한 프레임워크 포함) : 개발을 편하게 만들어주는 다양한 기능들을 제공하고, 그 중 하나가 클래스(부픔)를 이용하여 애플리케이션을 조립하는 것이다.
* 클래스패스 스캐닝
  * 스프링에 패키지만 알려주면 거기서 찾은 클래스로 애플리케이션을 조립
  * 그러나 코드의 규모가 커지면, 어떤 빈이 애플리케이션 컨텍스트에 올라오는지 정확히 알 수 없게 되고, 테스트에서 애플리케이션 컨텍스트의 일부만 독립적으로 띄우기가 어려워짐
* 애플리케이션 조립을 책임지는 전용 설정 컴포넌트
  * 책임으로부터 자유로워짐