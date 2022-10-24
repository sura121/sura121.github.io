---
title: "Devlog: 일급컬렉션에 대해서"
excerpt_separator: "<!--more-->"
categories:
  - Devlog
tags:
  - Devlog
  - JAVA
---
  
일급 컬렉션  
--

Collection을 Wrapping하면서, 그 외 다른 멤버 변수가 없는 상태를 일급 컬렉션 이라고 합니다.
Collection을 Wrapping하게 되면 다음과 같은 이점을 가지게 됩니다.

    1. 비즈니스에 종속적인 자료구조
    2. Collection의 불변성을 보장
    3. 상태와 행위를 한곳에서 관리
    4. 이름이 있는 컬렉션
    


1. 비즈니스에 종속적인 자료구조

예를 들어 다음과 같은 조건으로 로또 복권 게임을 만든다고 가정하겠습니다.

로또 복권은 다음의 조건이 있습니다.

 - 6개의 번호가 존재
 - 6개의 번호는 서러 종북되지 않아야  
 
 일반적으로 이런 일은 서비스 메소드에서 진행합니다.
 
 그래서 구현을 해보면 아래처럼 됩니다.
 
 ```java

public class LottoService {
    private static final int LOTTO_NUMBERS_SIZE = 6;
    
    public void createLottoNumber() {
        List<Long> lottNumbers = createNonDouplicateNumbers();
        validateSize(lottNumbers);
        validateDuplicate(lottNumbers);
        
        //이후 로직 쭉쭉 실행
        
    }
    
    private void validateSize(List<Long> lottoNumbers) {
        if(lottoNumbers.size() != LOTTO_NUMBERS_SIZE) {
            throw new IllegalArgumentException("로또 번호는 6개만 가능합니다.");
        }
    }
    
    private void validateDuplicate(List<Long> lottoNumbers) {
        Set<Long> nonDuplicateNumbers = new HashSet<>(lottoNumbers);
        if(nonDuplicateNumbers.size() != LOTTO_NUMBERS_SIZE) {
            throw new IllegalArgumentException("로또 번호들을 중복될 수 없습니다.");
        }
    }
}
```

서비스 메소드에서 비지니스 로직을 처리했습니다.
이럴 경우 큰 문제가 있습니다.

로또 번호가 필요한 모든 장소에선 검증로직이 들어가야만 합니다.

 - List<Long> 으로 된 데이터는 모두 검증 로직이 필요할까요?
 - 신규 입사자분들은 어떻게 이 검증로직이 필요한지 알 수 있을까요?
 
 등등 모든 코드와 도메인을 알고 있지 않다면 언제든 문제가 발생할 여지가 있습니다.
 
 그렇다면 이문제를 어떻게 깔끔하게 해결할 수 있을까요?
  
  - 6개의 수자로만 이루어져야만 하고
  - 6개의 수자는 서로 중복되지 않아야만 하는
  
  자료 구조를 직접 만들면 됩니다.
  

아래와 같이 해당 조건으로만 생성 할 수 있는 자료구조를 만들면 위에서 언급한 문제들이 모두 해결됩니다.

그리고 이런 클래스를 우린 **일급 컬렉션** 이라고 부릅니다.

```java

public class LottoTicket {
    private static final int LOTTO_NUMBERS_SIZE = 6;
    
    private final List<Long> lottoNumbers;
    
    public LottoTicket(List<Long> lottonumbers) {
        
    }
    
    private void validateSize(List<Long> lottoNumbers) {
        if(lottoNumbers.size() != LOTTO_NUMBERS_SIZE) {
            throw new IllegalArgumentException("로또 번호는 6개만 가능합니다.");
        }    
    }
    
    private void validateDuplicate(List<Long> lottoNumbers) {
        Set<Long> nonDuplicateNumbers = new Hast<>(lottoNumbers);
        
        if(nonDuplicateNumbers.size() != LOTTO_NUMBERS_SIZE) {
            throw new IllegalArgumentException("로또 번호들은 중복될 수 없습니다.");
        }
    }
}
```

이제 로또 번호가 필요한 모든 로직은 이 일급 컬렉션만 있으면 됩니다.


2.불변
-

일급 컬렉션은 컬력센의 불면을 보장합니다.

여기서 `final`을 사용하면 안되나요? 라고 할 수 있습니다.

java의 final은 정확히는 불변을 만들어주는 것은 아니며, 재할당만 금지 합니다.

final로 할당된 코드에 재할당할순 없기 때문입니다.

요즘과 같이 소프트웨어 규모가 커지고 있는 상황에서 불변 객체는 아주 중요합니다.
각각의 객체들이 절대 값이 바뀔일이 없다는게 보장되면 그만큼 코드를 이해하고 수정하는데 사이드 이펙트가 최소화되기 때문입니다.

java에서는 final로 그 문제를 해결할 수 없기 때문에 일급 컬렉션 (First class Collection)과 래퍼 클래스 (Wrapper Class)등의 방법으로 해결해야만 됩니다.

그래서 아래와 같이 컬렉션의 값을 변경할 수 있는 메소드가 없는 컬렉션을 만들면 불변 컬렉션이 됩니다.

```java

public class Orders { 
    private final List<Order> oders;
    
    public Orders(List<Order> orders) {
        this.orders = orders;
    }
    
    public long getAmountSum() {
        return orders.stream()
        .mapToLong(Order::getAmout)
        .sum();
    }
}
```

이 클래스는 생성자와 getAmountSum() 외에 다른 메소드가 없습니다.
즉, 이 클래스의 사용법은 새로 만들거나 값을 가져오는 것 뿐입니다.

List라는 컬렉션에 접근할 수 있는 방법이 없기 때문에 값을 변경/추가가 안됩니다.

이렇게 일급 컬렉션을 사용하면, 불변 컬렉션을 만들수 있습니다.

3.상태와 행위를 한곳에서 관리
---

이 부분읜 Enum의 장점과 일맥상통합니다.

예를 들어 여러 Pay들이 모여있고, 이 중 NaverPay금액의 합이 필요하다고 가정해보겠습니다.

일반적으로는 아래와 같이 작성합니다.

```java
@Test
public void 로직이_밖에_있는_상태(){
    //given
    List<Pay> pays = Arrays.asList(
            new Pay(NAVER_PAY, 1000),
            new Pay(NAVER_PAY, 1500),
            new Pay(KAKAO_PAY, 2000),
            new Pay(TOSS, 3000L));
    
    //when
    Long naverPaySum = pays.stream()
    .filter(pay->pay.getPayType().equlas(NAVER_PAY))
    .matToLong(Pay::getAmount)
    .sum();
    
    //then
    assertThat(naverPaySum).isEqultTo(2500);
}
```

 - List에 데이터를 담고
 - Service 혹은 Util 클래스에서 필요한 로직 수행
 

이 상황에서는 문제가 있습니다.
결국 pays라는 컬렉션과 계산 로직은 서로 관계가 있는데, 이를 코드로 표현이 안됩니다.

Pay타입의 상태에 따라 지정된 메소드에서만 계산되길 원하는데, 현재 상태로는 강제할 수 있느 수단이 없습니다.

만약에 네이버 페이외에 카카오 페이의 총금액도 필요하게 된다면 코드가 흩어질 확률이 높습니다.

```java

public class PayGroups {
    private List<Pay> pays;
    
    public PayGroup(List<Pay> pays) {
        this.pays = pays;
    }
    
    public Long getNaverPaySum() {
        return pays.stream()
        .filter(pay -> payTypes.isNaverpay(pay.getpayType()))
        .matToLong(Pay::getAmount)
        .sum();
    }
}
```

이렇게 일급 컬렉션이 생김으로 상태와 로직이 한곳에서 관리 될 수 있습니다.


 [출처] https://jojoldu.tistory.com/412