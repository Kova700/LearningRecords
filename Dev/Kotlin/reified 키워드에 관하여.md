## reified(실체화 된) 키워드란?
***
JVM위에서 작동하는 언어의 제네릭은 컴파일 타임에 타입 검증을 마치면, 타입 정보가 지워진 채로 바이트 코드(.class)가 생성된다.    
**즉, 런타임에 제네릭 클래스는 타입 파라미터에 대한 정보를 가지고 있지 않다.**  

그래서 제네릭 클래스를 **변성(Variance) 규칙**을 따르지 않고 함부로 타입 캐스팅하면,   
아래와 같이 타입 안정성을 보장 받기 힘들다.  
(이를 보고 [힙 오염(Heap Pollution)](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%9E%99-%EC%98%A4%EC%97%BC-Heap-Pollution-%EC%9D%B4%EB%9E%80)이 발생했다고 한다.)    
```kotlin
private fun main() {  
	val list = mutableListOf(1,2,3,4,5)  
	val any :Any = list  
	val list2 :MutableList<String> = any as MutableList<String> 
  
	list2.add("asd")  //:MutableList<Int> 였으나 String이 들어가는 상황이 발생
  
	println(list2) //[1, 2, 3, 4, 5, asd]  
}
```
그래서 안전한 타입 캐스팅을 하지 않을 것이라면 Any로 무작정 타입 캐스팅을 하는 것은 좋지 않다.  
변성 규칙에 관해서는 다른 글에서 정리를 할 예정이고,  
이번 글에서는 런타임에도 타입 파라미터에 대한 정보를 가지고 있을 수 있는 **reified** 키워드에 관해서 알아보자.  
</br>
***

코틀린에서는 inline함수와 **reified**(실체화 된) 키워드를 이용해서   
제네릭에 명시된 타입 파라미터의 정보를 런타임에도 가지고 있을 수 있다.  
이를 **실체화된 타입 파라미터** 라고 칭한다.  
</br>

### 실체화..?   
### 뭔가 와닿지 않는다.  

제네릭의 타입 파라미터는 컴파일 타임에만 존재하고,   
런타임에는 존재하지 않는 실체화 되지 않는 **비 실체화 타입(Non -Reifiable Type)** 이다.  
(즉 런타임에는 존재하지 않는 **유령 같은** 타입 파라미터가 **실체를 가지고 런타임에 존재**한다는 의미로 받아들여도 무방할 듯하다.)   

```kotlin
fun <T> isA(value :Any) = value is T //Cannot check for instance of erased type: T
```
컴파일러가 지워진 인스턴스 타입 T에 대한 체크를 할 수 없다고 에러를 띄운다.  
</br>

### 타입 파라미터 실체화를 어떻게 할 수 있는 걸까?  
inline 함수는 **함수가 호출된 곳에 함수의 본문에 대한 바이트 코드를 생성해준다.(코틀린 컴파일러 캐리)**   
이런 inline함수의 특성을 이용하여 inline함수의 내용에 대한 바이트 코드를 생성할 때,    
**reified** 키워드로 받은 타입 파라미터에 해당하는 타입 정보를 바이트 코드에 같이 넘겨줌으로써,  
런타임에도, 타입에 대한 정보를 가지고 있을 수 있다.  
(타입 파라미터로 넘겨준 타입에 대한 코드를 작성해준다고 보면 된다.)  

```kotlin
inline fun <reified T> isA(value: Any) = value is T

private fun main() {  
	println(isA<String>("asd")) //true
}

//아래와 같은 코드로 코틀린 컴파일러가 바이트 코드를 만들어줬다 생각하면 됨
private fun main() {  
	val value = "asd"  
	println(value is String)
}
```

### 활용
***
평소에 ::class 혹은 ::class.java로 클래스 타입을 검증하던 함수를  
reified 키워드를 이용해 아래와 같이 수정할 수 있다. 
(안드로이드에서 Activity를 띄울 때, 호출하는 코드다.)

```kotlin
//이렇게 호출하던 코드를
val intent = Intent(this, LoginActivity::class.java)  
startActivity(intent)  

//이렇게 제네릭으로 깔끔하게 호출가능하다.
startActivity<LoginActivity>()

  
inline fun <reified T : Activity> Context.startActivity() {  
    val intent = Intent(this, T::class.java)  
    startActivity(intent)  
}
```


## Reference 
***
[kotlin in action 도서 9장 - 제네릭스 ](https://www.yes24.com/Product/Goods/55148593)  
https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%9E%99-%EC%98%A4%EC%97%BC-Heap-Pollution-%EC%9D%B4%EB%9E%80  
