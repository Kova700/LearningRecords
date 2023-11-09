원자성이란 **멀티스레드 환경**에서 **여러 개의 스레드가 공유 자원에 대한 작업 (Write)을 할 때, 모두 반영되는 상태**를 말한다.
원자성이 없는 타입에 대해서도 Synchronized 키워드 혹은Atomic 클래스를 통해서 원자성을 가질 수 있다.



 x++ 과 같은 3가지 작업이 한 번에 일어나는 코드의 경우 
 volatile 같은 키워드로 가시성을 해결 했더라도, 예상하지 못한 결과가 나타난다.
 ```java
 public class Main {
    public volatile static int count = 0;            // 캐시 메모리 사용 X --> 가시성 해결
    public static void main(String[] args) throws Exception {
        Test t1 = new Test();
        Test t2 = new Test();

        t1.start();
        t2.start();

        while (t1.isAlive() || t2.isAlive());            // t1, t2 스레드 종료될 때까지 대기
        System.out.println(count);            //20000보다 더 적은 수가 나옴
    }
}

class Test extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            Main.count++;
        }
    }
}
```
위와 같은 현상이 발생한 근본적인 원인은 비원자 연산때문이다.

“count++” 코드는
① count 변수의 값을 가져온다  
② count 변수의 값을 증가시킨다  
③ 변경된 count 변수를 저장한다.
위 세가지 작업으로 나눠지는 비원자적 연산이다.

그래서 두 개의 스래드로 해당 코드에 접근하면 아래와 같은 상황이 발생한다.
```
① 첫 번째 스레드에서 메인 메모리로부터 count 변수의 값을 가져온 다음 인터럽트가 발생하여 해당 스레드는 실행 대기 상태로 들어간다.  
② 두 번째 스레드에서 메인 메모리로부터 count 변수의 값을 가져온다.  
③ 두 번째 스레드에서 count 변수의 값을 증가시킨다.  
④ 두 번째 스레드에서 값이 변경된 count 변수를 메인 메모리에 저장한다.  
⑤ 실행 대기 상태가 된 첫 번째 스레드가 실행 상태로 바뀌고, count 변수의 값을 증가시킨다.
⑥ 두 번째 스레드에서 메인 메모리로부터 count 변수의 값을 가져온다.  
⑦ 첫 번째 스레드에서 값이 변경된 count 변수를 메인 메모리에 저장한다.
```
- 이처럼 비원자 연산은 작업이 여러 개로 분리되어 이루어져 한 번에 처리되지 않기 때문에 위와 같은 결과가 나타난 것이다.


- Synchronized 키워드
- Atomic 클래스 - 해당 작업을 CPU low level에서 Atomic하게 수행해주는 클래스
	- 그렇기 때문에, lock을 걸지 않고도 연산자들의 원자성을 보장할 수 있다.
위 두 가지로 해당 코드가 원자성을 가지게 만들 수 있다.


## Reference
***
https://shuu.tistory.com/58  
https://sslblog.tistory.com/201  
https://javaplant.tistory.com/23  
https://velog.io/@backtony/Java-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88%EC%99%80-Atomic-%EC%82%AC%EC%9A%A9%EB%B2%95   
https://saltyzun.tistory.com/37  
https://jupiny.com/2020/06/23/use-atomicreferencefieldupdater/  