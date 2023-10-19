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
#### 그렇다면, 수정 가능한 변수도 그냥 람다 객체에 복사하면 되지, 왜 final 변수만 사용할 수 있게 제한을 둔 거냐?

**그 이유는 JVM 메모리 구조를 보면 알 수 있다.**  

![[Pasted image 20231019233815.png]]
(출처 : https://asfirstalways.tistory.com/158)

**Heap 영역**과 **Method (static)영역**은 독립적으로 존재하는데 반해, 지역 변수가 저장되는 **Stack 영역은 Thread안에 속해 있다.**
즉, **Stack 영역은 Thread들 간에 공유되지 않는 영역이다.**
스레드간에 값 불일치가 생길 수 있다.



### 코틀린은 어떻게 수정 가능한 변수(var)를  람다 내부에서 사용할 수 있는걸까
***



## Reference
***
[kotlin in action 도서 5장 - 람다로 프로그래밍 ](https://www.yes24.com/Product/Goods/55148593)  
https://vagabond95.me/posts/lambda-with-final/   
https://asfirstalways.tistory.com/158  