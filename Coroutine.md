# Coroutine이란?

실행의 지연과 재개를 허용함으로써, 비선점적 멀티태스킹을 위한 서브 루틴을 일반화한 컴퓨터 프로그램 구성요소이다.

## 비전섬적 멀티태스킹?

- 비선점형 : 하나의 프로세스가 CPU를 할당받으면 종료되기 전까지 다른 프로세스가 CPU를 강제로 차지할 수 없다.(coroutine)
- 선점형 : 하나의 프로세스가 다른 프로세스 대신에 프로세서를 강제로 차지할 수 있다.(thread)

따라서 코루틴은 **병행성은 제공하지만, 병렬성은 제공하지 않는다.**

- 병행성(=동시성) : 논리적으로 병렬로 작업이 실행되는 것처럼 보이는 것<br/>
- 병렬성 : 물리적으로 병렬로 실제로 작업이 실행되는 것

# Thread vs Coroutine

쓰레드와 코루틴 모두 비동기 작업 처리와 멀티태스킹이 가능하다. 그러나 왜 쓰레드를 사용하지 않고 코루틴을 사용할까?

다음은 음식을 주문하는 프로그램의 코드이다.

```kotlin
fun main() {
    orderFood()
    prepareFood()
    deliverFood()
}

fun orderFood() {
    println("Ordering food")
    Thread.sleep(2000)
}

fun deliverFood() {
    println("Delivering food")
    Thread.sleep(2000)
    println("Food delivered")
}

fun prepareFood() {
    println("Preparing food...")
    Thread.sleep(1000)
    println("Food prepared")
}

```

위 코드는 프로그램의 시작점인 main()은 메인 루틴이 되고 그 안에 호출되는 함수들은 모두 서브 루틴이다. 서브루틴은 진입점과 종료시점을 갖게 되는데 일반적으로 사용하는 함수의 호출 시점과 return 시점이 진입과 종료 시점이 된다. 서브루틴은 진입점 부터 종료시점까지 중단없이 실행이 되기 때문에 각각의 서브루틴들 사이의 관계는 계층적, 직렬적 관계가 되고 즉, 동기적인 코드가 된다.

## 메인 & 서브 루틴?

프로그램은 여러 루틴의 조합으로 진행되는데, 메인 루틴과 서브 루틴으로 나뉜다.

- 메인 루틴 : 프로그램 전체의 개괄적인 동작으로 main 함수에 의해 수행되는 흐름
- 서브 루틴 : 반복적인 기능을 모은 동작으로 main 함수 내에서 실행되는 개별 함수의 흐름

서브 루틴은 메모리에 기능을 모아 놓고, 호출 시 저장된 메모리에 이동한 뒤 실행 후 반환문을 통해 원래 호출 위치로 돌아온다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8dcaf93-7993-4615-abbc-cea9b7d78243/2b2da12e-9969-4998-8081-15f9be807932/Untitled.png)

코루틴은 서브 루틴과 비슷하지만 큰 차이점이 있다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8dcaf93-7993-4615-abbc-cea9b7d78243/2109b71d-e743-478b-b98a-5914757a7a57/Untitled.png)

서브 루틴은 단일 진입 지점에서 시작 후 단일 반환 지점에서 종료되는 것에 반해, 코루틴은 진입 후 반환문이 없더라도 임의 지점에서 실행 중 동작을 중단하고 이후 해당 지점에서부터 실행을 재개한다. (진입과 반환이 여러 개이다.) 이는 내부적으로 Continuation Passing Style(CPS, 연속 전달 방식)과 State machine을 이용하여 동작한다.

코루틴에서 함수 호출 시 연산 결과 및 다음 수행 작업과 같은 제어 정보를 가진 일종의 콜백 함수인 Continuation을 전달하며 각 함수의 작업이 완료되면 Continuation을 호출한다. 이를 통해 상태를 연속적으로 전달하며 컨텍스트를 유지하고 코루틴 실행 관리를 이한 State machine에 따라 코드 블록을 구분해 실행한다.

다음은 **Thread**를 사용한 **비동기 작업 코드**이다.

```kotlin
fun main() {
    asyncOrderFood {
        stopNotification()
        prepareFood()
        asyncDeliverFood {
            stopNotification()
        }
        asyncSendNotification()
    }

    asyncSendNotification()
    
    Thread {
        for (i in 1..5) {
	            println("Main thread doing other tasks: $i")
            Thread.sleep(500)
        }
    }.start()
}

fun asyncOrderFood(onOrderComplete: () -> Unit) {
    Thread {
        println("Ordering food")
        Thread.sleep(2000)
        onOrderComplete.invoke()
    }.start()
}

fun asyncDeliverFood(onDeliveryComplete: () -> Unit) {
    Thread {
        println("Delivering food")
        Thread.sleep(2000)
        onDeliveryComplete.invoke()
        println("Food delivered")
    }.start()
}

var sendingNotification = false

fun asyncSendNotification() {
    Thread {
        println("Send notification")
        sendingNotification = true
        while (sendingNotification) {
            println("Notification: Food is being prepared...")
            Thread.sleep(500)
        }
    }.start()
}

fun stopNotification() {
    sendingNotification = false
    println("Stop notification")
}

fun prepareFood() {
    println("Preparing food...")
    Thread.sleep(1000)
    println("Food prepared")
}

```

Thread를 사용하여 비동기 처리한 코드입니다. 비동기 처리가 되어 좀 더 나아진 코드인 것 같아 보이나 thread와 callback을 이용한 선점형 비동기처리는 몇가지 문제점을 가지고 있다.

1. 코드의 복잡성 & 가독성
    - 각각의 루틴들은 독립적인 thread 안에서 동작을 한다. 따라서 각 루틴들이 서로에게 영햐을 주기 위해서는 thread 사이에 통신이 필요하다. 이는 코드를 복잡하게 하고 관리하기 힘들게 된다.
    - thread/callback 구조는 코드 상으로 흐름을 파악하기가 쉽지 않다.
2. 비용
    - 기본적으로 thread는 운영체제에서 할당하고 관리를 하게 된다. 운영체제에서 thread들의 작업을 적절하게 분배하기 위해 코어에 각각의 task들을 적절하게 할당, 회수 작업을 하게 된다. 이처럼 운영체제에 의해서 작업이 할당되는 것을 선점형 멀티태스킹(preemptive-multitasking) 이라고 한다. 운영체제가 각 thread의 작업을 스케줄링 할 때 context switching이 필요하게 되는데 이때 context switching 비용이 발생한다.무분별한 thread 생성이 결국 많은 리소스를 소비하게 되어 전체적인 프로그램 성능을 저하시킨다.

다음은 **Coroutine**를 사용한 **비동기 작업 코드**이다.

```kotlin
fun main() {
    runBlocking {
        val orderFood = launch {
            coroutineOrderFood()
        }

        val playMusicWithOrderFood = launch {
            coroutinePlayMusic()
        }

        orderFood.join()
        playMusicWithOrderFood.cancel(
        coroutinePrepareFood()

        val deliverFood = launch {
            coroutineDeliverFood()
        }

        val playMusicWithDeliverFood = launch {
            coroutinePlayMusic()
        }

        deliverFood.join()
        playMusicWithDeliverFood.cancel(
        coroutineTakeFood()
    }
}

suspend fun coroutineOrderFood() {
    println("Ordering food")
    delay(2000)
}

fun coroutinePrepareFood() {
    println("Preparing food...")
    Thread.sleep(1000) // Note: Ideally should use delay in a coroutine
    println("Food prepared")
}

suspend fun coroutineDeliverFood() {
    println("Delivering food")
    delay(2000)
    println("Food delivered")
}

fun coroutineTakeFood() {
    println("Taking the food")
}

suspend fun coroutinePlayMusic() {
    println("Play music")
    while (true) {
        println("Listening..")
        delay(500)
    }
}
```

runBlocking은 현재 thread를 block하는 coroutine을 생성하는 함수이다. 즉 runBlocking이 호출된thread는 runBlocking내의 작업이 완료되기 전까지 다른 작업을 하지 못한다.

launch함수는 현재 thread에 대한 blocking없이 실행되는 coroutine을 생성한다. 즉 현재 thread에 다른 작업을 할당 할 수 있다.

위 코드에서는 크게 runBlocking, orderFood, playMusic 3개의 coroutine이 상호작용 하고 있다. runBlocking은 main thread를 잡고 있으며 작업이 완료 될 때까지 프로그램이 종료 되지 않도록 한다. 즉 main routine이 된다.

main routine은 orderFood와 playMusic이라는 2개의 coroutine을 생성하고 실행한다. orderFood와 playMusic은 동시성을 보장하면서 각각의 routine을 수행한다. 그 후 main routine에서 orderFood.join()을 호출한다. 이는 orderFood coroutine이 완료 될 때까지 현재 routine을 일시정지 시키고 orderFood coroutine이 완료가 되면 그 때 routine을 다시 재개 하겠다라는 의미이다. orderFood coroutine이 완료가 되면 playMusic coroutine을 cancle 시키고 coroutinePrepareFood와 coroutineDeliverFood를 호출하는 구조다.

위 코드 처럼 coroutine을 이용하여 routine과 routine간의 관계 정의만을 통해서 동시성이 보장되는 비동기 프로그래밍을 하였다.일반적인 subroutine과는 다르게 coroutine에서는 비동기적으로 routine을 실행 할 수 있었으며 각 루틴에서 실행되는 작업들을 중간에 일시정지(join)하고 임의의 시점에서 재개 할 수 있다. 이를 통해 코드의 가독성은 높아졌으며 개발자가 정의한 모든 subroutine을 같은 context에서 쉽게 실행 할 수 있게 한다. 이처럼 coroutine은 routine과 routine간의 관계를 정의하고 정의된 관계에 따라 스케줄링을 언어레벨에서 해줌으로써 코드를 좀 더 명확하게 하고 context switching 비용을 줄일 수 있게 한다.
