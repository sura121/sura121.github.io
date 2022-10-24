---
title: "Devlog: AOP, FILTER, INTERCEPTOR 차이 및 사용"
excerpt_separator: "<!--more-->"
categories:
  - Devlog
tags:
  - Devlog
  - Spring Boot
  - aop
  - filter
  - interceptor
---

토이 프로젝트를 진행하면서 AOP, INTERCEPTOR, FILTER 개념 및 사용에 대한 궁금증이 생겨서 정리하게 되었습니다.

일단 aop, filter, intercepotr의 차이점은 실행되는 시점이 다르다는 것입니다.

공통점으로는 셋다 사후처리, 사전처리가 가능합니다. 어떻게 보면 셋다 같은 기능일수 있지만 개발자가 필요한
시점에 따라서 좀더 효율적으로 사용할 수 있습니다.

filter
===

![](https://t1.daumcdn.net/cfile/tistory/996C413359C9BF2917)

이미지 참조 https://hayunstudy.tistory.com/53 님의 블로그 

필터의 경우 ServletRequest, ServletResponse 

서블릿 컨테이너가 실행되는 시점에서 실행됩니다. 

- 클라이언 요청 정보를 제공하는 객체를 정의
- IP, hostName, 프로토콜 형식, 서버이름 , 포트번호, Context-Type 등을 가져 올 수 있습니다.
- Servlet 의 web.xml 정의 (Spring boot 로 오면서 web.xml을 사용하지 않고  org.springframework.boot.web.servlet 의 RegistrationBean 을 통해 등록해야합니다.)
- doFilter 메소드의 chain.doFilter(request,response) 를 기점으로 전/후 처리가능


Interceptor
===

![](https://t1.daumcdn.net/cfile/tistory/99C75B3359C9C08312)

![](https://t1.daumcdn.net/cfile/tistory/990C573359C9C2B834)

Interceptor은 Spring의 DispatcherServlet이 실행되는 전, 후에 실행이 된다.
필터는 스프링 컨텍스트 외부에 존재하지만 Interceptor는 스프링 컨텍스트 내부에 존재하게 되어있어
스프링내의 모든 객체(bean)에 접근이 가능하다. 

- preHandle() : 컨트롤러 메소드가 실행되기전
- postHandle() : 컨트롤러 메소드 실행 직후 view 가 렌더링 되기전 
- afterCompletion() : view가 렌더링 된 

AOP
===

관점 지향 프로그래밍

비지니스 로직에 트랜잭션, 로그, 권한 관심사에 대해서 기능을 구현할 수 있다.
개발자가 원하는 세밀한 부분에서 좀더 적극적인 처리를 할 수 있다. 

AOP에 관해서는 좀더 자세히 포스팅을 하도록 하겠습니다.




