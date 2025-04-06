# 프로세스와 쓰레드의 차이
- 오늘 내가 할일이 밥먹기, 공부하기, 운동하기라면?
  - 밥먹기, 공부하기 -> 프로세스
  - 밥을 먹기 위해.. 밥짓기, 반찬 준비하기, 수저놓기,.. -> 쓰레드
  - 누구는 밥짓고,누구는 반찬준비하고, .. 동시에 처리 -> 멀티 쓰레드
- 차이점은 결국 독립성! 
  - 프로세스는 다른 프로세스에 영향을 미치지 않지만, 스레드의 작업은 다른 스레드에 영향을 미칠 수 있다.

# 경량 스레드
- 기존 언어의 스레드 모델보다 더 작은 단위로 실행단위를 나눈다! 

# 코루틴은 경량 쓰레드?
- 코루틴은 쓰레드가 아니고 쓰레드 안에서 동작하는 하나의 work 단위
- 여러개의 코루틴을 하나의 특정 쓰레드에서 동작시킬 수 있으나 코루틴을 동시에 실행하는 것은 불가능

```kotlin
// 쓰레드에 생성해서 1000번 반복 -> 1000번의 쓰레드 생성
fun main() = runBlocking {
    val threads = mutableListOf<Thread>()
    println("처음 : ${Thread.activeCount()}")
    repeat(1000) {
        threads += Thread {
            Thread.sleep(1000)
        }.also {
            it.start()
        }
    }
    println("끝 : ${Thread.activeCount()}")
}

// 코루틴 사용 -> 쓰레드 수가 2개로 같다
fun main() = runBlocking {
	val jobs = mutableListOf<Job>()
	println("처음 : ${Thread.activeCount()}")
	repeat(1000) {
		jobs += launch {
			delay(1000)
		}
	}
	println("끝 : ${Thread.activeCount()}")
}
```

# 자바 21의 Virtual Thread
