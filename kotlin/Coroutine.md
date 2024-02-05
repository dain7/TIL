# 코루틴
- 실행의 지연과 재개를 허용함으로써, 비선점적 멀티태스킹을 위한 서브 루틴을 일반홯한 컴퓨터 프로그램 구성요소

### 비선점적 멀티태스킹
- 비선점형: 하나의 프로세스가 CPU를 할당 받으면 종료되기 전까지 다른 프로세스가 CPU를 강제로 차지할 수 없다. (코루틴)
- 선점형: 하나의 프로세스가 다른 프로세스 대신에 프로세서를 강제로 차지할 수 있다. (쓰레드)

따라서 코루틴은 병행성은 제공하지만, 병렬성은 제공하지 않는다.

- 병행성 (동시성): 논리적으로 병렬로 작업이 실행되는 것처럼 보이는 것
- 병렬성: 물리적으로 병렬로 작업이 실행되는 것

선점형으로 비동기를 할 경우에 문제점
1. 코드의 복잡성
2. 비용

이러한 문제를 해결하기 위한 방법 -> 비선점적 멀티 태스킹. </br>
따라서 코루틴은 쓰레드가 아닌 쓰레드 내 동작하는 하나의 작업 단위이며 정의된 다양한 구성요소의 집합인 context를 오버라이드 하며 실행된다.

### 일시중단 가능한 코루틴
- 코루틴은 기본적으로 일시중단 가능하다. 
- launch로 실행하든, async로 실행하든 내부에 해당 코루틴을 일시중단 해야하는 동작이 있으면 코루틴은 일시 중단된다.
- suspend fun은 일시중단 가능한 함수를 지칭하는 것이다.
ex. 하나의 thread가 block 될 경우, 해당 thread는 다른 작업을 할 수 없는 block 상태에 놓이게 된다. 
그러나 suspend function을 사용한다면 blocked 된 상태에 놓일 때, 그 작업을 suspend하고 그 기강동안 thread에서 다른 작업 수행 가능! 

```kotlin
fun main() {
    launch(Dispatchers.IO) {
      // 병렬로 2개 함수 실행 
      async { suspendTask1() }
      async { suspendTask2() }
    }
}

suspend fun suspendTask1() {
    delay(3000)
    Log.d(TAG, "[suspendTask1] After 3s in (${Thread.currentThread().name})")
    delay(3000)
    Log.d(TAG, "[suspendTask1] After 6s in (${Thread.currentThread().name})")

    Log.d(TAG, "[suspendTask1] END in (${Thread.currentThread().name})*****")
}
suspend fun suspendTask2() {
    delay(1000)
    Log.d(TAG, "[suspendTask2] After 1s in (${Thread.currentThread().name})")
    delay(3000)
    Log.d(TAG, "[suspendTask2] After 4s in (${Thread.currentThread().name})")

    Log.d(TAG, "[suspendTask2] END in (${Thread.currentThread().name}) *****")
}
```


### coroutine scope vs run blocking
- run blocking
  - Runs a new coroutine and blocks the current thread interruptibly until its completion.
  - thread를 작업이 완료될 때까지 Blocking 한다.
