---
title: "Develog: "Spring Boot 생성자 주입을 사용해야 하는 이유"
excerpt_separator: "<!--more-->"
categories:
  - Develog
tags:
  - Develog
  - Spring Boot
  - Wrapper
---
    
    
  프로젝트의 구조를 다시 설계 하기위해서 공부를 하다가 스프링의 Dependency Injection시 3가지 방법으로
 주입 할 수 있는 방법에 대해서 공부하게 되었습니다.
 
 DI
 ===
  일단 Dependency Injection 이란 객체 지향언어에서 통용되는 개념이다.
  
  **강한결합**
  
   객체 내부에서 다른 객체를 생성하는 경우가 강한  ``강한결합`` 도를 가진다고 한다.
   A클래스 내부에서 B라는 객체를 직접 주입받게 되면, B객체를 C객체로 변경하게 되면 A클래스도
   수정을 하게된다.
   
 
  **약한결합**
  
  객체를 주입 받는다는 것은 외부에서 생선된 객체를 인터페이스를 통해서 넘겨 받는것이다.
  이렇게 하면 결합도를 낮출수 있고 런타임시 의존관계를 가지게 되어 유연한 구조를 가지게 된다.
  
  SOLID 원칙중 Open Close Principle 을 지키기 위해서 디자인 패턴중 전략패턴을 사용하게 되는데
  , 생성자 주입을 하게되면 전략적 패턴을 사용하게 된다.
  
  Setter Based Injection
  
  의존관계에서는 크게 두가지 방법이 있다 수정자 주입 방법과, 생성자 주입 방법이 있다.
  그 중에서 수정자 주입 방식을 알아보도록 하겠습니다.
  
```
    public class Controller {
        private Service service;
    
        public void setService(Service service) {
            this.service = service;
        }
    
        public void callService() {
            service.doSomething();
        }
    }

```


```
public interface Service {
    void doSomething();
}
```

```
public class ServiceImpl implements Service {
    @Override
    public void doSomething() {
        System.out.println("ServiceImpl is doing something");
    }
}

```

```
public class Main {
    public static void main(String[] args) {
        Controller controller = new Controller();

        // 어떤 구현체이든, 구현체가 어떤방법으로 구현되든 Service 인터페이스를 구현하기만 하면 된다.
        controller.setService(new ServiceImpl1());
        controller.setService(new ServiceImpl2());

        controller.setService(new Service() {
            @Override
            public void doSomething() {
                System.out.println("Anonymous class is doing something");
            }
        });

        controller.setService(
          () -> System.out.println("Lambda implementation is doing something")
        );

        // 어떻게든 구현체를 주입하고 호출하면 된다.
        controller.callService();
    }
}

```

익명 클래스나 람다로 구현 할 수 있었던 것은 인터페이스가 함수형 인터페이스 이기 때문입니다.

함수형 인터페이스란 인터페이스가 추상메소드를 하나만 가지고 있는것을 말합니다.

- Controller class는 callService() 메소드 Service 타입의 객체에 의존합니다.
- Service는 인터페이스이고 인터페이스는 인스턴스화 할 수 없으므로 구현체가 필요합니다.
- Service 인터페이스를 구현하기만 했다면 어떤 타입의 객체라도 Controller에서 사용할수 있다(다형성)
  Controller에서는 구현체 내부에서 어떤일을 하는지 알필요가 없다.
- main 함수에서 Controller에서 사용하는 것을 보면, 수정자 메소드인 setService() 함수 Service 인터페이스 구현체를 넘겨주면 된다.

수정자 주입으로 의존관계 주입시 런타임시 할 수 있도록 낮은결합도를 가지게 할 수 있다.
하지만 수정자 주입으로 Service의 구현체를 주입하지 않아도 Controller에서 동작이 가능하다.
Controller에서 객체 생성이 가능하다는 것은 callService()의 호출이 가능하게 된다는 것이다.
그렇게 되면 NullPointException이 발생하게 된다. 주입이 필요한 객체에 주입되지 않아도 객체가 생성된다는게 문제이다

이 문제점을 해결할수있는 부분이 생성자 주입이다.


Constructor based Injection (생성자를 통한 주입)
==

Controller에서 setter을 없애고 생자를 이용해서 주입한다.

```
 public class Controller {
     private Service service;
 
     public Controller(Service service) {
         this.service = service;
     }
 
     public void callService() {
         service.doSomething();
     }
 }

```

이렇게 생성자 주입으로 변경되게 되면 코드는 아래와 같이 변한다.

```
public class Main {
    public static void main(String[] args) {

        // Controller controller = new Controller(); // 컴파일 에러

        Controller controller1 = new Controller(new ServiceImpl());
        Controller controller2 = new Controller(
            () -> System.out.println("Lambda implementation is doing something")
        );
        Controller controller3 = new Controller(new Service() {
            @Override
            public void doSomething() {
                System.out.println("Anonymous class is doing something");
            }
        });

        controller1.callService();
        controller2.callService();
        controller3.callService();
    }
}
```
  
 이를 통해 이득과 하나의 보너스가 생기게 된다.
 
 1. NullPointException 문제가 해결 된다.
 2. 의존관계 주입을 하지 않게 되면 컴파일에러가 발생하게 된다.
 
 final로 선언을 할 수 있게 된다. final로 선언된 레퍼런스타입 변수는 반드시 선언과 함께 초기화가 되어야 된다.
 수정자 주입시에는 의존관계를 주입 받을수가 없게 된다.
 
 final의 장점으로는 누군가 final로 선언되 service객체를 바꿀수가 없게 된다.
 
  
 