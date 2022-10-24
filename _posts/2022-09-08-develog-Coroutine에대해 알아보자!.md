---
title: "Devlog: Corutine에대해 알아보자!"
excerpt_separator: "<!--more-->"
categories:
  - Devlog
tags:
  - Devlog
---

# 배경

- 현재 우리 시스템에 사용되는 coroutine이 어떻게 왜 사용되고 있는지 coroutine에 개념을 익혀, 이해해보기 위함

# 목표

- corutine 원리 (개념을 익혀보자!)
- suspend 함수에 대해서 알아보자!
- suspend 함수 내부 동작 및 최적화 (다 이해 못함..)

# 목표가 아닌 것

- coroutine외 reactivex? java에서 사용하는 비동기 라이브러리
- 쓰레드와 코루틴의 차이

## Co-Routines 란?

코루틴이란? 함께(Co) 수행되는 함수(Routine)이다.

![Untitled](/assets/images/corutin-img_1.png)

- 위의 이미지 같이 Coroutine1 함수안에서 Coroutine2가 수행되는 것이 아니다. 각각은 서로 다른 함수(Routine)이며, 함께 수행 되고 있다.
- 이들이 하나의 Thread를 점유하고 있을 때 한 Routine이 다른 Routine에게 Thread 점유 권한을 양보함으로써 함께 수행된다.

## CoRoutine 개념을 익혀보자!

- 협력형 멀티태스킹!?
    - 위의 내용과 일맥상통한다. 우리가 알고 있는 Routine에는 흔히 main routine, sub routine이 존재한다.
    - main routine과 sub routine은 아래와 같다.

    ![Untitled](/assets/images/corutine-img-2.png)
    
    - 말 그대로 main 함수를 Main Routine이라고 하고, 우리가 만든 함수들을 Sub Routine으로 볼 수 있다.
    - sub routine의 시작점과 끝나는 점은 명확하다.
    - main routine이 sub routine을 호출하고 return을 만나거나, sub routine 이 끝나는 괄호를 만나면 끝나게 될 것입니다.
    - 그리고 진입점과 탈출점 사이 sub routine이 실행되는 동안 쓰레드는 블락이 되어있다.

    ![Untitled](/assets/images/corutine-img-3.png)
    
    - 위의 이미지에서 코루틴의 진입점과 탈출점을 알 수 있습니다.
    - 코루틴을 사용하게 되면 실행 함수에 대해서 탈출을 점이 여러개가 된다.
    - 3번에서와 같이 모든 suspend 함수를 실행 시키고 난뒤에 메인 쓰레드는 다른 작업을 진행하고 있다가 모든 suspend 함수가 작업이 완료되면 그 아래에서 다시 작업을 하게 됩니다.

## Suspend function

- suspend의 사전적인 의미는 “중지하다" 라는 뜻입니다.
- coroutine에서의 suspend keyword는 무엇을 의미 할까?
    - 시작하고, 멈추고, 다시 시작할 수 있는 함수라고 합니다.
- suspend란 비동기 실행을 위한 중단 지점의 의미
- 즉, 잠시 중단 한다는 의미이고, 잠시 중단한다면 언젠가는 다시 시작(resume)된다는 뜻입니다.
- suspend 함수를 사용하지 않을 경우와 사용할 경우에 쓰레드 자원을 어떻게 사용하는지 확인 해보려고 합니다.
    - suspend 함수에서는 쓰레드 자원이 블락이 되지 않고, suspend된 상태에서 다른 작업을 수행하는지?
    - blocked된 작업이 suspend 되고 다시 resume 되는지?
    
    ### suspend 함수를 쓰지 않은 경우

    ![Untitled](/assets/images/corutine-img-4.png)
    
    ### suspend 함수를 쓸 경우

    ![img.png](assets/images/corutine-img-5.png)
    
- suspend function 이 suspend 된 후 resume 할 수 있는 이유?
- 함수에 suspend 키워드를 추가하면 바이트코드를 보면 해당 함수에 Continuation 이라는 인자값이 추가 됩니다.
- 이 Continuation이 프로그램의 현재 상태를 저장 합니다. Continuation wrapper 내에 나머지 코드를 전달하는 것처럼 생각할 수 있습니다????( 이 내용이 정확히 무엇인지 아직 이해를 못했습니다… )
- 현재 작업이 완료되면 continuation block이 실행된다.
- 아래는 suspendTask1 의 디컴파일된 코드 입니다. Continuation 객체가 인자값으로 들어가 있는 것을 확인 할 수 있습니다.

![Untitled](assets/images/corutine-img-6.png)

- 일단, Continuation interface 가 어떻게 정의되어 있는지 보면

```kotlin
interface Continuation<in T> {
  public val context: CoroutineContext
  public fun resumeWith(value: Result<T>)
}
```

- context 와 resumeWith라는 함수가 있습니다.
- context는 연속성에 사용될 CoroutineContext가 된다고 합니다.
- resumeWith 는 정지를 불러일으키거나 예외를 발생하는 계산의 결과값을 가지고 있는 Result를 가진 코루틴을 재실행 한다고 합니다?

설명을 위한 예제 코드

```kotlin
suspend fun loginUser(userId: String, password: String): User {
  val user = userRemoteDataSource.logUserIn(userId, password)
  val userDb = userLocalDataSource.logUserIn(user)
  return userDb
}

// UserRemoteDataSource.kt
suspend fun logUserIn(userId: String, password: String): User

// UserLocalDataSource.kt
suspend fun logUserIn(userId: String): UserDb
```

- 내부적으로
- 컴파일러는 아래와 같이 suspend keyword에 해당하는 함수를 바꿔줍니다.
- 코틀린 컴파일러는 suspend 함수를 가져와서 `유한한 상태 머신` 을 사용하여 최적화된 버전의 콜백으로 변환 합니다!

컴파일러 시기 1

```kotlin
fun loginUser(userId: String, password: String, completion: Continuation<Any?>) {
  val user = userRemoteDataSource.logUserIn(userId, password)
  val userDb = userLocalDataSource.logUserIn(user)
  completion.resume(userDb)
}
```

- 다른 Dispatchers 사용하기!
- Dispatcher 간에 서로 다른 스레드에서 계산을 실행하도록 스왑 할 수가 있다고 합니다!?
- 그럼 코틀린은 중단된 연산을 어떻게 재개 할까요? (이부분은 더 알아봐야될 것 같습니다..)
    - Continuation에는  DispatchedContinuation 이라고 하는 서브 타입이 있는데, 여기서 resume 함수가 CoroutineContext 내부의 Dispatcher로 dispatch 호출을 합니다!
- 이제 코틀린 컴파일러는 함수가 내부적으로 suspend 되는 시기를 식별 합니다.
- 모든 중단 지점을 유한한 상태 머신에서 중단 상태로 표기를 한다고 합니다. 이런 상태 표기는 컴파일러에 의해 라벨로 표시 됩니다.

컴파일러 시기 2

```kotlin
fun loginUser(userId: String, password: String, completion: Continuation<Any?>) {
  // Label 0 -> 첫 실행
  val user = userRemoteDataSource.logUserIn(userId, password)
  // Label 1 -> userRemoteDataSource에 의해 재개
  val userDb = userLocalDataSource.logUserIn(user)
  // Label 2 -> userLocalDataSource에 의해 재개
  completion.resume(userDb)
}
```

- 더 많은 상태를 가지는 상태 머신을 위해서 컴파일러는 다음처럼 여러 상태를 구현 하기 위해 when문을 사용합니다.

컴파일러 시기 3

```kotlin
fun loginUser(userId: String, password: String, completion: Continuation<Any?>) {
  when(label) {
    0 -> { // Label 0 
        userRemoteDataSource.logUserIn(userId, password)
    }
    1 -> { // Label 1 
        userLocalDataSource.logUserIn(user)
    }
    2 -> { // Label 2 
        completion.resume(userDb)
    }
    else -> throw IllegalStateException(...)
  }
}
```

- 다른 상태들이 정보를 공유할 방법이 없어 여기서도 continuation 객체를 사용하여 작업을 수행하게 됩니다.
- 컴파일러는 필요한 데이터를 보유하고 `loginUser` 기능을 재귀적으로 호출하여 실행을 재개하는 private 클래스를 많듭니다.

```kotlin
fun loginUser(userId: String?, password: String?, completion: Continuation<Any?>) {
  class LoginUserStateMachine(
    // completion 파라미터는 함수에 대한 콜백이다.
    // loginUser를 호출한다.
    completion: Continuation<Any?>
  ): CoroutineImpl(completion) {
      
    // suspend 함수의 로컬 변수
    var user: User? = null
    var userDb: UserDb? = null
    // 모든 CoroutineImpls에 대한 공통 개체
    var result: Any? = null
    var label: Int = 0
    // 이 함수는 loginUser를 다시 호출하여 상태 머신을 트리거하고 (라벨은 이미 다음 상태로)
    // 결과는 이전 상태 계산의 결과일 것이다.
    override fun invokeSuspend(result: Any?) {
      this.result = result
      loginUser(null, null, this)
    }
  }
  ...
    
}
```

- invokeSuspend가 Continuation 객체의 정보로 loginUser 함수를 다시 호출합니다.
- 여기서 loginUser의 파라미터 completion은 nullable 입니다.
- 이 시점에 컴파일러는 상태 간 이동 방법에 대한 정보를 추가 합니다.
- 위에서 말한 상태 간 이동 방법에 대한 정보는
    - 해당 함수가 처음 실행하는지
    - 함수를 이전 상태에서 재개한 것인지
- 위의 두가지를 아는 것입니다.
- continuation이 LoginUserStateMachine 타입인지 여부를 확인하면서 다음 작업을 수행합니다!

```kotlin
fun loginUser(
	userId: String?,
	password: String?, 
	completion: Continuation<Any?>
) {
  ...
  val continuation = 
		completion as? LoginUserStateMachine ?: LoginUserStateMachine(completion)
  ...
}
```

- 만약 함수가 처음 호출된 경우에는 LoginUserStateMachine 인스턴스를 생성하고 수신한 completion을 매개변수로 저장하여 이 인스턴스를 호출하는 기능을 다시 시작 한다는 것을 기억해야 됩니다.
- 이렇게 호출했는지 정보를 추가하지 않을 경우에 상태 머신(일시 정지)의 실행을 계속 할 것입니다.

```kotlin
fun loginUser(
	userId: String?, 
	password: String?, 
	completion: Continuation<Any?>) 
{
    ...
    val continuation = 
    completion as? LoginUserStateMachine ?: LoginUserStateMachine(completion)

    when(continuation.label) {
        0 -> {
            // 실패 확인
            throwOnFailure(continuation.result)
            // 다음 번에 이 continuation이 실행되면 상태 1로 넘어간다.
            continuation.label = 1
            // continuation 객체가 loginUser로 전달되어 이 상태 시스템이 완료되면
            // 실행을 재개한다.
            userRemoteDataSource.logUserIn(userId!!, password!!, continuation)
        }
        1 -> {
            // 실패 확인
            throwOnFailure(continuation.result)
            // 이전 상태의 결과물 가져오기
            continuation.user = continuation.result as User
            // 다음 번에 이 continuation이 실행되면 상태 2로 넘어간다.
            continuation.label = 2
            // 위와 같은 상황 반복
            userLocalDataSource.logUserIn(continuation.user, continuation)
        }
        ... // 마지막 상태는 일부러 빼놓았다
    }
}
```

- 위의 코드를 보면 when문의 인자값은 LoginUserStateMachine 인스턴스 내부의 라벨 입니다.
- 새로운 상태가 될 때 마다, 이 기능이 정지되어 실패했을 경우를 체크합니다.
- 다음 suspend 함수를 호출하기 전에 LoginuserStateMachine 의 인스턴스의 라베일이 다음 상태로 업데이트 합니다.
- 마지막으로, 상태 머신 내부에서 다른 suspend 함수에 대한 호출이 있을때 continuation 인스턴스가 매개 변수로 전달됩니다.
- 호출된 suspend 함수는 컴파일러에 의해 변경되고 continuation 객체를 파라 미터로 가지는 같은 유형의 또 다른 상태 머신에 의해 관리되고 suspend의 상태 머신이 완료되면, 다시 상태 머신의 실행을 재개합니다.

```kotlin
fun loginUser(
	userId: String?, 
	password: String?,
	completion: Continuation<Any?>) 
{
    ...

    val continuation = 
		completion as? LoginUserStateMachine ?: LoginUserStateMachine(completion)

    when(continuation.label) {
        ...
        2 -> {
            throwOnFailure(continuation.result)
            // Gets the result of the previous state
            continuation.userDb = continuation.result as UserDb
            // 이 함수를 호출한 함수 다시 실행
            continuation.cont.resume(continuation.userDb)
        }
        else -> throw IllegalStateException(...)
    }
}
```

- 마지막 2번 라벨을 실행 시키면서 이 함수를 실행을 재개해야 하므로 마지막 상태는 다르게 됩니다!

# 함께 읽기

- kotlin 문서

[](https://kotlinlang.org/docs/coroutines-guide.html)

- co - routine 이란?

[[Coroutine] Coroutine(코루틴)과 Subroutine(서브루틴)의 차이 - Coroutine이란 무슨 뜻일까?](https://kotlinworld.com/214)

- coroutine 사용하기 (non-blocking 처리 순서제어 예제)

[Kotlin Coroutine 사용하기](https://ellune.tistory.com/64)

- coroutine 개념 익히기

[코틀린 코루틴(coroutine) 개념 익히기](https://wooooooak.github.io/kotlin/2019/08/25/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0/)

- coroutine의 스레드 최적화를 어떻게 하는가?

[[Coroutine] 1. Coroutine 은 어떻게 스레드 작업을 최적화 하는가?](https://kotlinworld.com/139)

- suspend 함수의 내부 동작 원리

[Suspend 한정자 - 내부 동작](https://medium.com/hongbeomi-dev/suspend-%ED%95%9C%EC%A0%95%EC%9E%90-%EB%82%B4%EB%B6%80-%EB%8F%99%EC%9E%91-55d2dc819044)

- 코루틴 이란? (2)

[코루틴 소개](https://medium.com/@jooyunghan/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EC%86%8C%EA%B0%9C-504cecc89407)

- Continuation interface kotlin docs

[Continuation - Kotlin Programming Language](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-continuation/)

- Dispatcher 란?