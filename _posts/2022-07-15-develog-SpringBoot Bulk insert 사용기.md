---
title: "Devlog: SpringBoot Bulk Insert 사용기"
excerpt_separator: "<!--more-->"
categories:
  - Devlog
tags:
  - Devlog
  - SpringBoot
---

# 배경

 Coupon API 대량 지급을 개선하면서, 현재 프로젝트에서는 bulk Insert를 사용을 하지 못하고 있어 JPA를 사용하지 않고, jdbcTemplate를 사용해서 쿼리를 질의 할 수 있도록 개선

# 목표

- Batch Insert의 동작 원리와 설정을 알아보자!
- JPA를 사용하지 않고, Bulk Insert를 사용하는 방법을 알아보자!

# 목표가 아닌것

- JDBCTemplate, JPA saveAll() bulk Insert의 차이점, 성능 비교
- JDBCTemplate가 무엇인지…

# 논의

### spring data jpa batch insert란?

- batch insert라는 것은 여러개의 SQL Statement를 하나의 구문으로 처리할 수 있는것.
- 정확히 위의 spring data jpa batch 기능은 (2)**[write-behind](https://www.blog.ecsimsw.com/entry/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-1%EC%B0%A8-%EC%BA%90%EC%8B%9C-%EC%93%B0%EA%B8%B0-%EC%A7%80%EC%97%B0)** 를 통한 jdbc batch 기능이다. (우리가 알고 있는 (1)**[jdbc](https://devlog-wjdrbs96.tistory.com/139)**란?)
- 여러 개의 구문을 여러 번 network를 통해 보내는 것이 아니라 합쳐서 1개로 보내기!
    - JPA의 경우 트랜잭션이 commit 되는 순간 한꺼번에 flush 가 이루어짐
    - hibernate.jdbc.batch_size 의 정의가 되있지 않으면 batch insert는 이루어지지 않게 된다.

### spring data jpa batch insert 사용 방법

```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 1000
					order_inserts: true
	        order_updates: true
```

- order_inserts, order_updates 이 두개의 옵션을 설명하면, insert문과 update문의 실행 순서를 정렬해 줍니다.
- 정렬을 해야되는 이유는 다음과 같습니다.
  - 하나의 트랜잭션에 영속성을 가지는 여러개의 타입이 나타날 경우 hibernate는 새로운 batch를 만들게 됩니다.
  - 예를들어 A 타입을 생성하고, B타입을 생성한 뒤에 다시 A타입을 생성할 경우에 뒤에 생성한 A 타입은 처음에 생성한 A타입과 같다고 하지만 다른 batch로 묶이게 됩니다.

```sql
    # before apply option
    update test set column01 = 'test3' where regdate = '20210510';
    update test2 set column02 = 'test5' where regdate = '20210511';
    update test set column01 = 'test4' where regdate = '20210512';
    update test2 set column02 = 'test6' where regdate = '20210513';
    
    # after apply option
    update test set column01 = 'test3' where regdate = '20210510';
    update test set column01 = 'test4' where regdate = '20210512';
    update test2 set column02 = 'test5' where regdate = '20210511';
    update test2 set column02 = 'test6' where regdate = '20210513';
```

### spring data jpa batch insert 동작 원리

- hibernate.jdbc.batch_size옵션은 java의 PrepareStatement에서 제공하는 addBatch 함수를 사용해서 batch Insert가 이루어 집니다.
- addBatch란?
  - 쿼리 구문을 메모리에 올려두었다가, 실행 명령이 있으면 한번에 DB 쪽으로 쿼리를 실행하게 됩니다.
- prepareStatement의 addBatch를 호출하여 실행 할 쿼리를 추가하고, 설정한 배치 갯수에 도달하게 되면 prepareStatemnt의 executeBatch()가 실행되게 됩니다.
- executeBatch()가 실행되게 되면, DB 드라이버에서 addBatch()로 추가된 쿼리를 재조합하여 DB로 전송하게 됩니다.
- 이렇게 여러개의 쿼리를 한번에 주기 때문에 DB와의 통신 횟수도 줄고, DB에서 락을 잡는 횟수가 줄어들어 실행 속도가 향상 되게 됩니다.



### statement와 prepared Statement 의 차이

- statment와 preparedStatement의 가장 큰 차이점은 캐시의 여부이다.
  - preparedStatement가 캐시를 함

### 왜 기존 Coupon 프로젝트는 Bulk Insert를 사용하지 못했는가?

- Batch Insert가 필요한 이유
  - 현재 사용되고 있는 쿠폰 대량 발급의 제한 수는 1000건
  - 3만건 정도를 대량 지급을 하기위해서, 스크립트를 1000건씩 30번의 쿠폰 발급 신청을 할 수 있도록 구현함
  - 현재 쿠폰 대량 발급 로직은 `발급신청` 이벤트를 받아서 coupon-api 내부에서 발급 프로세스가 작동되게 되어있음
  - 발급신청을 30건의 요청은 잘 들어갔지만, 발급시 문제 발생 동시에 여러개의 요청이 들어가면서 발급 로직 도중 발급 count를 치는 update문이 한번만 실행 되고, 이후 29건에 대한 로직이 수행이 멈추면서 동시에 발급을 할 수 없는 구조 (해당 이슈는 bulk Insert와 별개의 문제 이므로 나중에 해결책을 찾아봐야 될꺼 같습니다.)

### JDBCTemplate를 적용 이유

- 위와 같은 이유로 bulk insert를 하기위해 하이버네이트 에서 제공하는 batch insert를 적용하기로함
- 문제 발생
  1. jpa batch insert는 엔티티의 GenerationType.IDENTITY 타입은 제공하지 않는다
    1. [실제 하이버 네이트 공식 문서에 적힌 글](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch-insert)

     ![Untitled](/assets/images/spring_document.png)

- Persistence Context에 엔티티를 식별할 때는 엔티티 타입과 엔티티의 id 값으로 식별하지만           IDENTIY 타입일 경우에는 DB에 insert를 해야만 id를 확인 할 수 있기때문이라고 합니다…..
- 만약 GenerationType을 TABLE 로 바꾼다면?
  - TABLE, SEQUENCE 채번 방식으로 바꾼다면?
  - 최적화를 하지 않으면 안쓰는것 보다 못하다?
  - 그 이유로는 채번 방식은 하나의 테이블로 sequence 값을 관리 하는 방법
  - 채번을 만들기 위해 각 insert문 마다 select, update가 필요해서 성능이 더 저하 될 수 있다….
- 또한, 성능 테스트에서 실제로 JDBCTemplate, 하이버네이트 Batch insert와 비교하면 2 ~ 3배 정도 빠르다고 한다. ([성능 비교](https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/))
- 위와 같은 이유로 JDBCTemplate 를 채택하게 되었고, JDBCTemplate를 적용해서 실제로 20분 걸리는 대량 발급을 3분안으로 줄였습니다.

### batch Insert 테스트 후기

- 예제 코드를 받아서 실제로 테스트 코드를 실행
- 하이버네이트는 배치설정에 상관없이 엔티티마다 쿼리를 호출한다.
- **rewriteBatchedStatements=true 란?**
  - Mysql JDBC Spec에 따라서 드라이버를 제작 하였지만 내부 적으로 BatchStatement를 하나의 요청으로 처리하지 않습니다. 이는 DB 내부 적으로 String Buffer에 붙여 넣는 방법으로 처리하게 됩니다. 또한 insert하는 row별 키를 얻어 오기 위해서는 insert문 이외의 명령문은 사용하면 안됩니다. ← 번역본..     이라고 쓰여 있는데 결론 적으로는 rewriteBatchedStatements 값을 true로 해야지 db에서 multi-value insert (batch insert) 가 가능 한데 제약사항이 많다라고 의미하는거 같습니다..

![Untitled](/assets/images/img.png)

- 또한, batchSize도 성능에 중요한 퍼포먼스를 냅니다. 예전 자료이긴 하지만 batchSize를 얼마나 설정하느냐에 따라서 실제로 성능 차이가 크게 나타납니다.
- 경우에 따라서 lager batch size는 오히려 성능을 drop 시킬 수 있다고 한다.

# 출처

- (1) jdbc란?

[[Java] JDBC를 사용한 데이터베이스 연동(Mysql)](https://devlog-wjdrbs96.tistory.com/139)

- (2) write-behind

[](https://www.blog.ecsimsw.com/entry/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-1%EC%B0%A8-%EC%BA%90%EC%8B%9C-%EC%93%B0%EA%B8%B0-%EC%A7%80%EC%97%B0)

- sql 로그

[The best way to log SQL statements with Spring Boot - Vlad Mihalcea](https://vladmihalcea.com/log-sql-spring-boot/)

- 우형 batch 사용기

[MySQL 환경의 스프링부트에 하이버네이트 배치 설정 해보기 | 우아한형제들 기술블로그](https://techblog.woowahan.com/2695/)

- GenrationType에 따른 성능 비교

[dev-tips/JPA-GenerationType-별-INSERT-성능-비교.md at master · HomoEfficio/dev-tips](https://github.com/HomoEfficio/dev-tips/blob/master/JPA-GenerationType-%EB%B3%84-INSERT-%EC%84%B1%EB%8A%A5-%EB%B9%84%EA%B5%90.md)

- 하이버 네이트 유저가이드