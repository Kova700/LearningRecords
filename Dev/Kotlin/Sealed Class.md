지인분에게 Enum Class와 Sealed Class의 차이점에 대한 질문을 받았었는데
깔끔하게 설명하지 못했었기에 정리하는 글을 남긴다.
## Sealed Class란?
***
Sealed == 밀봉된, 봉인된
클래스의 **상속가능 범위를 동일한 패키지 내로 제한**시킨 ==**추상 클래스**==
그래서 외부 패키지에서 해당 클래스의 서브 클래스를 생성하지 못한다.
(서브 클래스의 형태로는 object, data class, class 3가지가 있다.)
객체가 가져야할 프로퍼티가 있다면 data class, class
딱히 상태를 관리할 프로퍼티가 필요하지 않다면 object를 사용하면된다.
## Sealed Class를 사용하는 이유
***
#### 1. Sealed Class의 타입 분기문을 가진 when 문을 사용할 때, <br>빠트린 분기는 없는지 컴파일러가 알려준다. 
보통 Interface 혹은 abstract 클래스로 계층 구조를 나타내었을 때, 컴파일러가 서브 클래스들이 어떤 것들이 있는지 파악하지 못한다.
Interface 혹은 abstract 클래스의 가시성에 부합하다면 어떤 패키지에서든 서브 클래스를 생성할 수 있기 때문에 
모든 패키지를 뒤져서 어떤 서브 클래스가 있는지 찾는 것은 너무 리소스를 많이 소모하는 작업이다.

따라서 sealed class는 자식 클래스(child class)에 대한 선언을 같은 패키지 내로 제한한다. 
그래서 일반 abstract 클래스, Interface와 달리 자식들이 몇개가 있고 어떤게 자식인지 IDE가 알 수 있다.
(이러한 이점은 Enum 클래스도 가지고 있다. Enum 클래스 내부에만 상수를 선언할 수 있기 때문)
```kotlin
interface interfaceLoginEvent  
class MoveToMain : LoginEvent  
class MoveToSignUp : LoginEvent  
class ForbiddenUser : LoginEvent  
class NetworkError : LoginEvent  
class UnknownError : LoginEvent
```
<img width="80%" src="https://github.com/Kova700/LearningRecords/blob/master/Dev/Res/Pasted%20image%2020230924144039.png?raw=true"/>

```kotlin
abstract class LoginEvent
class MoveToMain : LoginEvent()  
class MoveToSignUp : LoginEvent()  
class ForbiddenUser : LoginEvent()  
class NetworkError : LoginEvent()  
class UnknownError : LoginEvent()  
```
<img width="80%" src="https://github.com/Kova700/LearningRecords/blob/master/Dev/Res/Pasted%20image%2020230924144225.png?raw=true"/>

```kotlin
sealed class SealedLoginEvent {  
    object MoveToMain : SealedLoginEvent()  
    object MoveToSignUp : SealedLoginEvent()  
    object ForbiddenUser : SealedLoginEvent()  
    object NetworkError : SealedLoginEvent()  
    object UnknownError : SealedLoginEvent()  
}
```
<img width="80%" src="https://github.com/Kova700/LearningRecords/blob/master/Dev/Res/Pasted%20image%2020230924144344.png?raw=true"/>
위와 같이 sealed class를 사용하면 when 분기문에서 빠진 분기는 없는지 알 수 있다.

추가로 enum또한 빠진 분기가 있는지 여부를 알려준다. (Enum 클래스 내부에 모든 상수가 선언되므로)
<img width="80%" src="https://github.com/Kova700/LearningRecords/blob/master/Dev/Res/Pasted%20image%2020230924151508.png?raw=true"/>

#### 2. Enum은 통일된 프로퍼티를 가져야하는 반면에 Sealed는 통일되지 않은 프로퍼티를 가질 수 있다. <br> (각 클래스에 적합한 프로퍼티를 가질 수 있음)
```kotlin
sealed class SealedLoginEvent {  
    object MoveToMain : SealedLoginEvent()  
    object MoveToSignUp : SealedLoginEvent()  
    object ForbiddenUser : SealedLoginEvent()  
    data class NetworkError(val error: Throwable?) : SealedLoginEvent()  
    data class UnknownError(val error: Throwable?) : SealedLoginEvent()  
}

sealed class CafeMenu {
    data class Americano(val price: Int, val 디카페인: Boolean) : CafeMenu()
    data class Latte(val price: Int) : CafeMenu()
    data class Frappe(val price: Int, val fruit: String) : CafeMenu()
    data class IceTea(val price: Int, val ice: Int) : CafeMenu()
    //각 메뉴별로 각자에게 필요한 프로퍼티들만 가질 수 있다.
}

enum class EnumCafe(val price: Int, val sugar: Int) {
    Americano(3500,  10),
    Latte(4000, 25),
    Frappe(5000, 100),
    IceTea(4500, 50)
}
//필요하지 않을 수도 있는데 같은 Enum class 안에 있다는 이유로 모두 같은 프로퍼티를 가져야함
```
맨 위 코드와 같이 Error 코드가 필요한 상황에 Event 객체 안에 Error 내용을 담아서 전달 할 수 있다.

#### 3.Enum Class는 컴파일 전에 미리 값을 설정해둬야 하는 반면에, <br> Sealed Class는 런타임에 매번 적합한 프로퍼티를 가진 객체를 생성할 수 있다.
```kotlin
sealed class UiState {  
	data class Success<T>(val data: T): UiState()
	data class Error(val error: Throwable?): UiState() 
	object Loading: UiState()
}
//enum은 미리 만들어 놓은 객체를 가져오는 방식이기 때문에, 
//Success와 같이 새로운 데이터를 집어넣어서 만든 새로운 객체를 만들 수 없다. 
```
그래서 네트워크 작업 결과 혹은 Error내용을 sealed class 객체에 담아서 전달하거나
보통 안드로이드에서 View Event를 다룰 때 많이들 사용한다.

#### 4. object키워드로 선언 시 Enum의 싱글톤 특성처럼 메모리 절약을 할 수 있다.
```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST;
}

sealed class Direction {
    object North :Direction()
    object South :Direction()
    object West :Direction()
    object East :Direction()
}

//혹은 아래와 같은 생성자를 가진 형태로도 사용가능 
enum class Direction(val korean :String) {
    NORTH("북"), SOUTH("남"), WEST("서"), EAST("동");
}

sealed class Direction(val korean :String) {
    object North :Direction("북")
    object South :Direction("남")
    object West :Direction("서")
    object East :Direction("동")
}
```
위와 같이 object로 선언하면 싱글톤으로 객체가 생성되기 때문에 상태를 가져야하는 객체가 아니라면
object로 선언함으로써 메모리 절약을 할 수 있다.

### 추가 사항
***
### 굳이 sealed 클래스의 형태를 중첩 클래스 형태로 선언하지 않아도 된다. <br> (sealed 클래스 객체 호출 시 앞에 클래스명이 붙는지 아닌지 차이만 생김)
==> 컨벤션을 정해서 취사 선택하면 될듯하다.
```kotlin
//중첩 클래스 선언 형태
sealed class SealedLoginEvent {  
    object MoveToMain : SealedLoginEvent()  
    object MoveToSignUp : SealedLoginEvent()  
    object ForbiddenUser : SealedLoginEvent()  
    data class NetworkError(val error: Throwable?) : SealedLoginEvent()  
    data class UnknownError(val error: Throwable?) : SealedLoginEvent()  
}
//SealedLoginEvent.MoveToMain 형식으로 호출

//일반 선언 형태
sealed class SealedLoginEvent 
object MoveToMain : SealedLoginEvent()  
object MoveToSignUp : SealedLoginEvent()  
object ForbiddenUser : SealedLoginEvent()  
data class NetworkError(val error: Throwable?) : SealedLoginEvent()  
data class UnknownError(val error: Throwable?) : SealedLoginEvent()  
//MoveToMain 형식으로 호출
```

### 동일한 패키지 내에서만 선언하면 서브 클래스가 선언된 파일이 달라도 상관 없다.
동일한 파일 내에 서브 클래스들이 선언되어야 한다는 블로그 글이 몇 개 있던데 동일한 패키지 내에서만 선언되면 
어떤 파일에서 서브 클래스가 선언되더라도 상관없다.
### Sealed interface도 있다. 
생성자에 프로퍼티를 딱히 가지고 싶지 않으면 interface를 사용하면 된다.
```kotlin
sealed interface Direction {
    object North :Direction
    object South :Direction
    object West :Direction
    object East :Direction
}
```
## 개인적인 의견
***
진짜 간단하게 표현해야겠다 싶으면 Enum, 
계층 구조가 필요하고 각 상황에 맞는 개별적인 로직, 파라미터가 필요하다싶으면 sealed를 사용하는 게 좋아보인다.
개인적인 생각으로는 Enum의 장점인 싱글톤을 sealed 클래스에서도 object로 표현이 가능하므로 sealed클래스로 구현해도 된다고 생각은 된다. 
```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST;
}

sealed class Direction {
    object North :Direction()
    object South :Direction()
    object West :Direction()
    object East :Direction()
}
```
하지만 enum이 훨씬 짧고 간단해 보이기는 하다. 이것도 취사선택의 영역인 것 같다.
개인 기호 혹은, 팀원과 규약을 정해서 사용하면 될듯하다.

## Related Link
***
[[Enum Class]]

## Reference
***
https://kotlinlang.org/docs/sealed-classes.html

https://kotlinworld.com/165

https://developer88.tistory.com/entry/Sealed-Class%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90

https://medium.com/depayse/kotlin-%ED%81%B4%EB%9E%98%EC%8A%A4-9-%EB%B4%89%EC%9D%B8%EB%90%9C-%ED%81%B4%EB%9E%98%EC%8A%A4-sealed-class-%EC%97%B4%EA%B1%B0%ED%98%95-%ED%81%B4%EB%9E%98%EC%8A%A4-enum-class-44a49465a3d

https://codechacha.com/ko/kotlin-sealed-classes/

https://cheonjoosung.github.io/blog/ko-sealed

https://medium.com/hongbeomi-dev/sealed-class%EC%99%80-sealed-interface-db1fff634860
