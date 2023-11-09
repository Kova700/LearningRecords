## 가시성 해결을 위해서
```java

public class JavaThreadVisibilityTest {
    private  static boolean stopRequested;  //volatile 유무에 따라 결과가 다름

    public static void main(String[] args) throws InterruptedException {
        Thread background = new Thread(() -> {
            for (int i = 0; !stopRequested ; i++);
            System.out.println("background 쓰레드가 종료되었습니다!");
        });

        background.start(); // (A)

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true; // (B)
        System.out.println("main 쓰레드가 종료되었습니다!");
    }
}

```
volatile 붙이지 않으면 backgroundThread가 속한 CPU 캐시 값은 바뀌지 않음으로
backgroundThread는 종료되지 않음
같은 CPU의 Thread일지, 다른 CPU의 Thread일지 알 수 없음
그래서 다중 스레드 환경에서 특정 변수의 값을 봐야 할 일이 있다면,
스레드 간 가시성을 맞추기 위해서 volatile 키워드를 써줘야함.

## But!!
자원을 저장하는 메모리는 하나가 되기 때문에 가시성 문제는 해결할 수 있으나,

여러 스레드에서 Main Memory에 있는 공유 자원에 동시에 접근할 수 있으므로 
여러 스레드에서 수정하게 되면, 계산 값이 덮어씌워지게 되므로 동시 접근 문제를 해결할 수 없다.
**정리하면, 가시성 문제는 해결할 수 있지만, 동시 접근 문제는 해결할 수 없다.**

이를 해결하기 위해서는,
1. synchronized키워드를 사용할 수 있지만, 비용이 크다.( 접근을 위해서 대기 현상이 벌어지게됨 )
2. Atomic 클래스를 이용해야한다.