지인분에게 Enum Class와 Sealed Class의 차이점에 대한 질문을 받았었는데
깔끔하게 설명하지 못했었기에 정리하는 글을 남긴다.
## Enum Class란?
***
자바에서 상수를 객체지향적으로 객체화해서 관리하기 위해서 생겨난 클래스
Enum 은 Enumeration 의 약자로 열거형을 의미하고 연관되거나 관련이 있는 **상수들의 집합**을 표현할 때 사용한다.
> 상수 : 어떤 값인지는 모르겠지만 값이 변하지 않고 항상 일정한 값으로 변수와는 반대되는 단어

## 기본 형태
```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST;

	//공통으로 사용될 때 일반 함수 
	fun print() = println("${this}..")
}
```

```kotlin
enum class State {  
	WAITING {  
		override fun print() {  
			println("Waiting..")  
		}  
	},  
	PROCESSING {  
		override fun print() {  
			println("Processing..")  
		}  
	},  
	DONE {  
		override fun print() {  
			println("Task Done")  
		}  
	};  
	
	//개별적 처리가 필요할 땐 추상 함수 (각각 다 구현을 해주어야함)
	//혹은 일반 함수에 When 분기문 추가
	abstract fun print()
}
```

```kotlin
enum class Color(val strRgb: String, val hexaRgb: Int) {
    RED("#f00", 0xFF0000),
    GREEN("#0f0", 0x00FF00),
    BLUE("#00f",0x0000FF )
}
//RED(255,0,0)를 RED(255,0,20)으로 변경하고 싶지만 
//Enum 클래스는 최초에 설정한 enum 각각에 대한 상태를 변경할 수 없다.
//Sealed Class는 가능
```

String처럼 [[Stack 영역]]에 있는 변수들이 [[Heap 영역]]에 있는 데이터의 주소 값을 저장함으로써 참조 형태를 띄게 된다. 
그래서 다음과 같이 같은 enum 타입 변수끼리 같은 상수 데이터를 바라봄으로써 
둘의 주소를 비교하는 **연산 결과는 true가 되게 된다.**
```kotlin
val a = Direction.NORTH  
println(a == Direction.NORTH)  //true
println(a === Direction.NORTH)  //true
```

## Enum의 내부 구성 

내부 구성은 아래와 같은 느낌으로 표현이 가능하다. 
#### Kotlin
```kotlin
class Season private constructor(val season: String) : Enum() {  
	companion object {  
		val SPRING = Season("SPRING") // 자기자신의 인스턴스를 만들어 상수화
		val SUMMER = Season("SUMMER")  
		val AUTUMN = Season("AUTUMN")  
		val WINTER = Season("WINTER")  
	}  
}
```
### Java
```java
final class Season extends Enum {
    public static final Season SPRING = new Season("SPRING"); // 자기자신의 인스턴스를 만들어 상수화
    public static final Season SUMMER = new Season("SUMMER");
    public static final Season AUTUMN = new Season("AUTUMN");
    public static final Season WINTER = new Season("WINTER");
    
    private String season;
    
    private Season(String season) {
        this.season = season;
    }
    
    public String getSeason() {
        return season;
    }
}
```
위 코드에서 알 수 있는 정보는
- 생성자의 접근 제어자가 private이므로, 외부에서 해당 클래스의 객체를 생성할 수가 없다.
- 클래스의 프로퍼티(멤버 변수)들을 객체를 만들지 않고, 전역에서 접근할 수 있다. (companion object , static)
- 상속이 불가능하다 (final, kotlin은 기본적으로 open키워드가 안 붙어 있으면 상속이 불가능)
	- Enum 클래스(Enum에 대한 구현을 가진 클래스)를 상속받고 있기 때문에 다른 클래스를 상속받을 수 없다. (다중 상속은 불가능함으로)
	- (name, ordinal이라는 프로퍼티를 Enum클래스로부터 상속받는다.)
	- ==> 상속을 받을 수도, 해줄 수도 없다.
- 컴파일타임에 Enum상수가 가질 변수 값을 알고 있어야 하고, 런타임에는 Enum상수가 가진 변수를 수정할 수 없다.
- Enum 상수들을 호출할 때마다 객체가 생성되는 게 아닌, 한 번 생성되면 그 후 같은 객체가 호출된다. (싱글톤)
- Enum class는  primitive 타입이 아닌 referece 타입이다. 그래서 enum 상수 값은 [[Heap 영역]]에 저장되게 된다. 
	- [[Method 영역]]에 저장된 static변수를 [[Stack 영역]]의 변수로 할당받아 [[Heap 영역]]에 저장된 객체를 가져와서 사용하는 느낌

## Enum Class를 사용하는 이유
- 가독성이 높아진다. 
	- 단순히 상수를 넣는 게 아닌 상수에 해당 상수가 어떤 의미를 가지는지 이름이 붙기 때문
- 상수 값의 타입 안정성이 보장된다. 
	- 정수로 된 일반 상수로 값을 사용하면 의도하지 않은 다른 정수 값이 전달되어도 타입 구분이 되지 않음으로 예외 처리가 힘듬
	- 특정 Enum 타입으로 when문을 짜는 경우에 새로운 Enum타입이 추가되면 IDE가 분기문에 새로운 Enum타입을 추가하라고 경고를 날려 줌  <br> (IDE 가 특정 Enum 상수에 대한 분기문이 빠져있음을 알 수 있음)
- 상수와 연관된 변수를 상수에 저장할 수 있다.(enum 생성자)

## 주의 사항 
- [[Sealed Class]]와 달리 상속을 지원하지 않는다. (상속을 해줄 수도, 받을 수도 없다.)
	- 고로 Enum Class에 대한 서브 클래스를 생성할 수 없다.
	- 어떠한 Class로부터도 상속받을 수 없다. 상속을 원한다면 Interface를 활용해보자.
- [[Sealed Class]]와 달리 최초에 설정한 enum 속성 값을 런타임에 할당하거나 변경할 수 없다.
	- ex) RED(255,0,0)를 RED(255,0,20)으로 코드를 수정하지 않는 이상 내부 프로퍼티 값을 변경할 수 없다. 

## Related Link
***
[[Sealed Class]]

## Reference
***
https://kotlinlang.org/docs/enum-classes.html#working-with-enum-constants

https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%97%B4%EA%B1%B0%ED%98%95Enum-%ED%83%80%EC%9E%85-%EB%AC%B8%EB%B2%95-%ED%99%9C%EC%9A%A9-%EC%A0%95%EB%A6%AC

https://kt.academy/article/kfde-enum

https://kotlinworld.com/82