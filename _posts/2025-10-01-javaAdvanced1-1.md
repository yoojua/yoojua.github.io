---
layout: post
title: "실전 자바 고급 1편 (1) 프로세스와 스레드 생성과 실행"
---

# 1. 프로세스와 스레드
> - 프로세스는 실행되는 프로그램의 독립적인 인스턴스로 자체 메모리 공간을 가진다.
- 스레드는 같은 프로세스 내에서 코드, 데이터, 힙 영역을 공유하고 각자 스택만 따로 갖는다.

1. 멀티프로세싱 vs 멀티태스킹
멀티프로세싱은 하드웨어 장비의 관점이고, 멀티태스킹은 운영체제 소프트웨어의 관점이다.
- 멀티프로세싱: 여러 CPU 코어를 사용하여 동시에 여러 작업을 수행하는 것
  - 예) 다중 코어 프로세서를 사용하는 현대 컴퓨터 시스템
- 멀티태스킹: 단일 CPU 코어가 여러 작업을 동시에 수행하는 것처럼 보이는 것
  - 예) 현대 운영 체제에서 여러 애플리케이션이 동시에 실행되는 환경
 
 2. 프로세스
>프로세스: 운영체제 안에서 실행 중인 프로그램
 - 프로세스는 실행 중인 프로그램의 인스턴스이다.
 - 각 프로세스는 독립적인 메모리 공간을 갖고 있고, 운영체제에서 별도의 작업 단위로 분리해 관리한다.
 - 스레드의 컨테이너라고 해석해도 된다.
- 프로세스의 메모리 구성
    - 코드 섹션: 실행할 프로그램의 코드가 저장되는 부분
    - 데이터 섹션: 전역 변수 및 정적 변수가 저장되는 부분(그림에서 기타에 포함)
    - 힙 (Heap): 동적으로 할당되는 메모리 영역
    - 스택 (Stack): 메서드(함수) 호출 시 생성되는 지역 변수와 반환 주소가 저장되는 영역(스레드에 포함)
 
3. 스레드
>`스레드`: 프로세스 내에서 실행되는 작업의 단위
- 프로세스는 하나 이상의 스레드를 반드시 포함한다.
- 한 프로세스 내에서 여러 스레드가 존재할 수 있으며, 프로세스가 제공하는 동일한 메모리 공간을 공유한다.
- 스레드의 메모리 구성
  - 공유 메모리: 같은 프로세스의 코드 섹션, 데이터 섹션, 힙(메모리)은 프로세스 안의 모든 스레드가 공유한다.
  - 개별 스택: 각 스레드는 자신의 스택을 갖고 있다.
>`단일 스레드`: 한 프로세스 내에 하나의 스레드만 존재
`멀티 스레드`: 한 프로세스 내에 여러 스레드가 존재

4. `컨텍스트 스위칭`: 현재 작업하는 문맥이 변하는 것
- 컨텍스트 스위칭 과정에서 비용이 발생한다.

- `CPU 바운드 작업 vs I/O 바운드 작업`
각각의 스레드가 하는 작업은 크게 2가지로 구분할 수 있다.
> - `CPU-바운드 작업 (CPU-bound tasks)`
    - CPU의 연산 능력을 많이 요구하는 작업을 의미한다.
    - 이러한 작업은 주로 계산, 데이터 처리, 알고리즘 실행 등 CPU의 처리 속도가 작업 완료 시간을 결정하는
    경우다.
    - 예시: 복잡한 수학 연산, 데이터 분석, 비디오 인코딩, 과학적 시뮬레이션 등
    
  >- `I/O-바운드 작업 (I/O-bound tasks)`
    - 디스크, 네트워크, 파일 시스템 등과 같은 입출력(I/O) 작업을 많이 요구하는 작업을 의미한다.
    - 이러한 작업은 I/O 작업이 완료될 때까지 대기 시간이 많이 발생하며, CPU는 상대적으로 유휴(대기) 상태
    에 있는 경우가 많다. 쉽게 이야기해서 스레드가 CPU를 사용하지 않고 I/O 작업이 완료될 때 까지 대기한
    다.
    - 예시: 데이터베이스 쿼리 처리, 파일 읽기/쓰기, 네트워크 통신, 사용자 입력 처리 등.

실무에서는 보통 CPU-바운드 작업 보다는 I/O-바운드 작업이 많다.


    
- 실무 이야기
`CPU 코어 수 + 1개`로 스레드를 맞추면 특정 스레드가 잠시 대기할 때 남은 스레드를 활용할 수 있다.
  - `CPU - 4개, 스레드 2개`
스레드의 숫자가 너무 적으면 모든 CPU를 100% 다 활용할 수 없지만, 스레드가 몇 개 없으므로 컨텍스트 스위칭 비용
    이 줄어든다.
  - `CPU - 4개, 스레드 100개`
스레드의 숫자가 너무 많으면 CPU를 100% 다 활용할 수 있지만 컨텍스트 스위칭 비용이 늘어난다.
  - `CPU - 4개, 스레드 4개`
스레드의 숫자를 CPU의 숫자에 맞춘다면 CPU를 100% 활용할 수 있고,
컨텍스트 스위칭 비용도 자주 발생하지 않기 때문에 최적의 상태가 된다.

---

# 2. 스레드
스레드는 객체이다.

스레드를 만들 때는 
`Thread 클래스`를 상속 받는 방법과 `Runnable 인터페이스`를 구현하는 방법이 있다.
실무에서는 `Runnable 인터페이스`를 구현한다.

## 🔍스레드 이론
![](https://velog.velcdn.com/images/mymy/post/910042a7-c0e4-4cc0-bf99-dbc7a2f3c251/image.png)


```java
public class HelloThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloThread helloThread = new HelloThread();

        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        helloThread.start();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}

```

- HelloThread 스레드 객체 생성후, `start()` 메서드 호출시
자바는 스레드를 위한 별도의 스택 공간을 할당한다.
  - main 스레드는 `main()` 메서드의 스택 프레임을 스택에 올리면서 시작한다.
  - 직접 만드는 스레드는 run() 메서드의 스택 프레임을 스택에 올리면서 시작한다.
  핵심은 main 스레드가 아니라 Thread-0 스레드가 `run()` 메서드를 실행한다.
  - ✅즉, 스레드의 start() 메서드는 스레드에 스택 공간을 할당하면서 시작하는 아주 특별한 메서드이다.
![](https://velog.velcdn.com/images/mymy/post/e748124d-93d2-4cd2-a355-ec3227a88b3f/image.png)

- 스레드 간의 실행 순서와 실행 기간은 보장하지 않는다. 이것이 멀티스레드 이다.



### 데몬 스레드
데몬 스레드: 사용자에게 직접적으로 보이지 않으면서 시스템의 백그라운드에서 작업을 수행하는 스레드
(예: 사용하지 않는 파일, 메모리 정리 작업)

> 사용자 스레드
- 프로그램의 주요 작업을 수행한다.
- 작업이 완료될 때까지 실행된다.
- 모든 user 스레드가 종료되면 JVM도 종료된다.

> 데몬 스레드
- 백그라운드에서 보조적인 작업을 수행한다.
- 모든 user 스레드가 종료되면 데몬 스레드는 자동으로 종료된다.

✅JVM은 데몬 스레드의 실행 완료를 기다리지 않고 종료된다. 
데몬 스레드가 아닌 모든 스레드가 종료되면, 자바 프로그램도 종료된다.

```java
public class DaemonThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        DaemonThread daemonThread = new DaemonThread();
        daemonThread.setDaemon(true); // 데몬 스레드 여부
        daemonThread.start();

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }

    static class DaemonThread extends Thread {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": run() start");
            try {
                Thread.sleep(10000); // 10초간 실행
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(Thread.currentThread().getName() + ": run() end");
        }
    }
}

```


---

## (1) 스레드 생성
Runnable 인터페이스를 구현해 스레드를 생성해보자.
```java
package java.lang;

public interface Runnable {
    void run();
}
```

```java
public class HelloRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}
```

```java
public class HelloRunnableMain {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloRunnable runnable = new HelloRunnable();
        Thread thread = new Thread(runnable);
        thread.start();

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}
```

로거 만들기
```java
package util;

import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

public abstract class MyLogger {
    private static final DateTimeFormatter formatter =
            DateTimeFormatter.ofPattern("HH:mm:ss.SSS");

    public static void log(Object obj) {
        String time = LocalTime.now().format(formatter);
        System.out.printf("%s [%9s] %s\n", time,
                Thread.currentThread().getName(), obj);
    }
}
```
```java
package util;

import static util.MyLogger.*;

public class MyLobberMain {
    public static void main(String[] args) {
        log("hello thread");
        log(123);
    }
}

```


### 여러 스레드 만들기
1. 중첩 클래스를 사용해 만들 수 있다.
```java
import static util.MyLogger.log;

public class InnerRunnableMainV1 {
    public static void main(String[] args) {
        log("main() start");

        Runnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();

        log("main() end");
    }

    static class MyRunnable implements Runnable {
        @Override
        public void run() {
            log("run()");
        }
    }
}

```
2. 익명 클래스를 사용해 특정 메서드 안에서만 간단히 정의할 수 있다.
```java
import static util.MyLogger.log;

public class InnerRunnableMainV2 {
    public static void main(String[] args) {
        log("main() start");

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        };

        Thread thread = new Thread(runnable);
        thread.start();

        log("main() end");
    }
}

```
3. 람다를 사용해 코드 조각을 전달할 수 있다.
```java
import static util.MyLogger.log;

public class InnerRunnableMainV4 {
    public static void main(String[] args) {
        log("main() start");

        // 람다를 배우면 이해
        Thread thread = new Thread(() -> log("run()"));
        thread.start();

        log("main() end");
    }
}

```

## (2) 스레드 실행
스레드 3개를 만들어보자.
```java
import static util.MyLogger.log;

public class ManyThreadMainV1 {
    public static void main(String[] args) {
        log("main() start");

        HelloRunnable runnable = new HelloRunnable();
        Thread thread1 = new Thread(runnable);
        thread1.start();
        Thread thread2 = new Thread(runnable);
        thread2.start();
        Thread thread3 = new Thread(runnable);
        thread3.start();

        log("main() end");
    }
}

```
실행결과
![](https://velog.velcdn.com/images/mymy/post/48e7ac1b-2b48-4800-a47e-c21451663cc1/image.png)
![](https://velog.velcdn.com/images/mymy/post/51d7fb30-50ad-44d7-b51b-86a3aa7109dd/image.png)

실행결과처럼 스레드의 순서는 보장되지 않는다.
- 스레드3개를 생성할 때 모두 같은 HelloRunnable 인스턴스( x001 )를 스레드의 실행 작업으로 전달했다
- Thread-0~2는 모두 HelloRunnable 인스턴스에 있는 run() 메서드를 실행한다.

> 다음 포스팅에서 이와 같은 멀티스레드 환경에서 스레드를 제어해보자!
