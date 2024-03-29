웹 어댑터 구현하기
==

웹 인터페이스 : 웹 브라우저를 통해 상호 작용할 수 있는 UI나 다른 시스템에서 우리 애플리케이션으로 호출하는 방식으로 상호작용하는 HTTP API가 대표적임.

외부세계와의 모든 커뮤니케이션은 어댑터를 통해 이루어진다.


의존성 역전
--
웹 어댑터는 '주도하는' or '인커밍' 어댑터다. 

외부로부터 요청을 받아 애플리케이션 코어를 호출하고 무슨일을 해야할 지 알려준다.

제어흐름<br>
--
웹 어댑터에 있는 컨트롤러(adapter.in.web) -> 포트(interface - application.port.in) -> 서비스(application.service)<br>
(웹 어댑터가 들어올 수 있는 포트는 서비스가 제공하기 때문에 인터페이스로 만들어서 어댑터는 포트만 호출하면 된다!)

포트를 없애도 호출 할 수 있음. but 간접계층을 넣음으로써 코어가 외부 세계와 통신할 수 있는 곳에 대한 명세인 포트를 알려주기 위함.<br>
(엔지니어에게도 유리함. 직관적이기 때문)

웹 어댑터의 책임<br>
--
1. HTTP 요청을 자바 객체로 매핑
2. 권한 검사
3. 입력 유효성 검증
4. 입력을 유스케이스의 입력 모델로 매핑
5. 유스케이스 호출
6. 유스케이스의 출력을 HTTP로 매핑
7. HTTP 응답을 반환

웹어댑터가 인증과 권한 부여를 수행하고 실패할 경우 에러를 반환함.

객체의 상태 유효성 검증을 해야하는데, 앞에서 말한 유스케이스의 입력 모델에서 일어나는 입력 유효성 검증과는 다르다.<br>
웹 어댑터의 입력 모델을 유스케이스의 입력 모델로 변환할 수 있다는 것을 검증해야함.<br>
-> 이 변환을 방해하는 모든 것이 유효성 검증 에러임.

애플리케이션에서 신경 쓰면 안되는 것들에 대한 책임을 지는 것이 웹어댑터.<br>
-> 유스케이스 호출, 유스케이스의 출력을 반환, HTTP응답으로 직렬화해서 호출자에게 전달, 예외 던지기, 에러를 메시지로 변환하기

이 규칙을 적용하려면 웹 계층부터 개발이 아닌, 도메인과 애플리케이션 계층부터 개발을 해야함!

> 컨트롤러는 적은것보단 많은게 좋다.
> 각 연산에 대해 가급적이면 별도의 패키지 안에 별도의 컨트롤러를 만드는 방식을 선호함.(저자)

유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까?
--
웹 어댑터는 도메인 로직을 수행하지 않아야 한다.<br>
애플리케이션 계층은 HTTP에 대한 상세 정보를 노출시키지 않도록 해야한다. 작업금지.<br>
-> 그래야 어댑터도 쉽게 교체가 됨.