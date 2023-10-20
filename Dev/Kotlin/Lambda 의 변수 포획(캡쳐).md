람다 내부에 정의된 지역변수가 아닌 **람다 외부에 정의된 변수를 람다 내부에서 사용할 때**,   
람다가 해당 변수를 **포획** 혹은 **캡쳐(Caputure)** 했다고 정의한다.  

자바에선 람다 외부 변수 중 final로 선언된 변수만 람다안에서 사용이 가능하다.   
그에 반해 코틀린은 수정 가능한(fianl이 아닌) 변수도 람다 안에서 사용이 가능하다.   

**자바 람다에서는 왜 final 변수만 사용 가능하게 제한을 두었는지** 알아보고    
코틀린과 자바의 어떤 내부적 차이가 있길래,   
**코틀린 람다에서는 어떻게 수정 가능한 변수를 사용할 수 있는지** 알아보자.  
(코틀린 val은 자바 final 변수와 같음으로, 편의상 코틀린으로 예를 들겠다. )

## 자바에서는 왜 람다에서 final 변수만 사용 가능하게 제한을 둔 걸까?
***
기본적으로, 함수에서 사용된 지역변수는 **Stack영역**에 메모리 할당이 되고,  
함수가 종료되면, 해당 함수에서 사용되었던 지역변수는 **Stack영역**에서 제거된다.  

하지만 람다에서는 함수의 지역변수를 복사해서 람다 객체 내부에 저장하기 때문에,   
함수가 종료되어도 그 종료된 함수에서 정의된 지역 변수 값을 사용할 수 있다.  

그래서 아래 코드에서 oneStep()함수가 종료되어도,   
testClass.action() 람다가 정상적으로 10을 출력할 수 있다.  
(action이라는 람다 객체 안 변수에 10의 값이 담긴다.)  

```kotlin
fun main() {
    val testClass = oneStep()
    testClass.action() //10 출력됨
}

fun oneStep() :TestClass{
    val testNum = 10 //이놈은 oneStep가 종료되면 사라지고 (함수가 종료되어도 살아 남는게 아님)
    return TestClass { println(testNum) } //action 람다에 값이 복사되어 저장된다.
}

class TestClass(val action: () -> Unit)
```

</br>

### 그렇다면, 수정 가능한 변수도 그냥 람다 객체에 복사하면 되잖아. </br>왜 final 변수만 사용할 수 있게 제한을 둔 거냐?

</br>

**그 이유는 JVM 메모리 구조를 보면 알 수 있다.**  

![](https://github.com/Kova700/LearningRecords/blob/master/Dev/Res/Pasted%20image%2020231019233815.png)  
(출처 : https://asfirstalways.tistory.com/158)

**Heap 영역**과 **Method(static)영역**은 Thread와 독립적으로 존재하는데 반해,   
지역 변수가 저장되는 **Stack 영역은 Thread 안에 속해 있다.**  
즉, **Stack 영역은 Thread들 간에 공유되지 않는 영역이다.**  

만약, **람다가 포획한(복사해간) 지역변수를 선언한 스레드**와   
**지역 변수를 포획한 람다를 사용하는 스레드**가 다르다면 
서로가 가지고 있는 변수의 이름은 같지만, 값이 다른 **변수 불일치**가 발생한다.
말로 하면 잘 이해가 안 될 테니 코드를 보자. 

```kotlin
fun main() {
    val workerThread = oneStep()
    workerThread.start() 
}

fun oneStep(): Thread {
    var testNum = 10 //메인 스레드에서 선언된 변수
    val workerThread = Thread { println(testNum) } //Worker 스레드에서 testNum 변수를 포획
    testNum = 20
    return workerThread
}
```

위와 같이 testNum이 10일 때, workerThread가 람다로 testNum변수를 포획했다.  
그 후에 testNum이 20으로 수정되고, onStep()함수가 종료되었다.    
main()함수에서 workerThread.start()를 호출했을 때,   
**workerThread는 10을 출력해야할까 20을 출력해야할까?**    

위에서 람다가 변수를 포획해갈 때, 변수를 복사해서 람다 객체 내부에 저장한다고 설명했었다.  
workerThread가 testNum(10)을 람다 내부에 복사해서 저장한 다음, 메인 스레드가 testNum을 20으로 수정했다.  
하지만, wokerThread가 가지고 있는 testNum은 이미 10으로 복사된 상태이기 때문에  
workerThread가 가지고 있는 testNum의 값은 10이고, 메인스레드가 가지고 있는 testNum은 20인   
같은 변수를 선언했지만 스레드 간에 값이 다른 상황이 생긴다.  

**Stack영역은 스레드 간에 공유되지 않는 영역이므로 wokerThread가 testNum을 복사해가는 순간  
메인 스레드의 testNum과 wokerThread의 testNum은 서로 다른 영역에 존재한다.**  
(wokerThread내부 Stack영역에 testNum이라는 같은 이름으로 변수가 선언된 것)  

위 코드를 짠 개발자는 20을 출력하고자 하니까 testNum을 뒤늦게라도 20으로 수정했을 것이다.  
하지만 개발자의 의도와 달리 변수 불일치가 발생한다.  
그래서 이런 예측하기 힘들고, 실수할 확률이 높은 경우를 방지하기 위해서 람다는 final 키워드로 선언된  
수정 불가능한 변수만 포획 가능하게 제한된 것이다.  

</br>

### **하지만 사실 위 코드에서 workerThread.start()는 정상적으로 20을 출력한다.**  
이는 자바는 final 변수(val)만 포획이 가능한데, 코틀린은 수정 가능한 변수(var)도 포획이 가능하다.   
이유를 알아보자.  

</br>

## 코틀린은 어떻게 수정 가능한 변수(var)를  람다 내부에서 사용할 수 있는걸까
***
위에서 봤던 JVM의 Runtime Data Area의 메모리 구조를 다시 보자.  

![](https://github.com/Kova700/LearningRecords/blob/master/Dev/Res/Pasted%20image%2020231019233815.png)  
(출처 : https://asfirstalways.tistory.com/158)

**Heap영역**은 **Stack영역**과 달리 스레드간에 공유되는 영역이다.  
즉, 스레드간에 항상 같은 값을 본다는 뜻이다.    
코틀린은 이 원리를 이용했다.   
아래는 위 코드를 자바로 디컴파일한 코드이다.  

```kotlin
   public static final void main() {
      Thread testThread = oneStep();
      testThread.start(); 
   }

   public static final Thread oneStep() {
      final Ref.IntRef testNum = new Ref.IntRef(); //래퍼 클래스 생성
      testNum.element = 10; //그 안에 값을 집어 넣는다.
      
      Thread thread = new Thread((Runnable)(new Runnable() {
         public final void run() {
            int var1 = testNum.element; //객체의 프로퍼티에 접근한다.
            //외부에 선언된 testNum변수에 접근 가능한 이유가 내부적으로 객체 참조를 가지고 있다는 뜻
            System.out.println(var1);
         }
      }));
      
      testNum.element = 20; //객체안 프로퍼티의 값을 수정
      return thread;
   }
```
**final**로 선언된 testNum변수에 Ref.IntRef라는 래퍼 클래스의 인스객체를 선언하고,  
래퍼클래스 객체 안에 그 값을 담는다.  
이렇게 하면, 항상 같은 메모리를 보고 변수를 가져오기 때문에, 
스레드간에 변수 불일치가 발생하지 않는다.  

## Reference
***
[kotlin in action 도서 5장 - 람다로 프로그래밍 ](https://www.yes24.com/Product/Goods/55148593)  
https://vagabond95.me/posts/lambda-with-final/   
https://asfirstalways.tistory.com/158  
