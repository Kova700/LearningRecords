## SAM(Single Abstract Method) Interface 란?
***
**하나의 추상 메소드만 있는 인터페이스**를 **함수형(functional) Interface** 또는 **SAM(Single Abstract Method) Interface** 라고 한다.  
(An interface with only one abstract method is called a _functional interface_, or a _Single Abstract Method (SAM) interface_.)  
(출처 : https://kotlinlang.org/docs/fun-interfaces.html)  

</br>

## 그래서 그걸로 뭘 할 수 있는데?
***
클래스가 한 번만 사용되고 더 이상 사용되지 않아서 클래스를 따로 정의하기 애매한 상황에   
아래와 같이 가독성을 위해서 (어떤 동작이 정의되었는지 바로 파악하기 위해서) **익명 객체** 선언을 다들 하곤 한다.  
(사실 OnClickListner를 요즘 잘 쓰진 않지만, 설명을 위한거니 넘어가자.)
##### Java
```java
button.setOnClickListener(new OnClickListner() {
		@Override
		public void onClick(View view) {
			...
		}
	}
)
```
#### Kotlin
```kotlin
button.setOnClickListener(object : OnClickListner {
		override fun onClick(View view){
			...
		}
	}
)
```

코틀린은 그냥 가능하고, 자바는 자바 8 이후부터 (람다가 나온 시점부터)  
위에 정의된 익명 객체들을 아래와 같이 **람다 표현식**으로 대체할 수 있다.  
##### Java
```java
button.setOnClickListener( (view) -> ... ) //매개변수가 하나라면 매개변수 괄호 생략 가능
```
#### Kotlin
```kotlin
button.setOnClickListener { view -> ... }
```

컴파일러가 내부적으로 람다 블럭을 SAM 인터페이스의 익명 객체로 변환을 해주는데  
이 변환을 **SAM conversions(변환)** 이라고 한다.  

이 **SAM 변환**은 모든 인터페이스가 가능한게 아니다.  
**단 하나의 추상 메소드만 가지고 있는(SAM) 인터페이스**에 한해서  
**익명 객체**를 **람다 표현식**으로 대체할 수 있다.  

</br>

## SAM 생성자
***
SAM 인터페이스를 익명 객체의 구문으로 인스턴스화 시키는 방법 말고,  
람다 표현식으로 인스턴스화 시키는 방법도 있다. (SAM 생성자라 칭함)  
(하나만 있던 추상 메소드의 파라미터만 남기고 나머지 내용들은 작성하지 않는 느낌)  
```kotlin
//기존에 사용하던 익명 객체선언
button.setOnClickListener(object : OnClickListner {
		override fun onClick(View view){
			println("버튼눌림")
			// this 호출가능 (OnClickListner 객체 가리킴)
		}
	}
)

//SAM 생성자를 이용한 익명 객체 선언 (컴파일러가 익명 객체로 변환해줌 (SAM 변환))
button.setOnClickListener( OnClickListener { view -> 
		println("버튼눌림") //메소드 명을 작성하지 않아도 됨
		// this 호출시 해당 블록을 둘러싼 클래스의 this가 호출됨
	} 
)

```

하지만,   
람다에는 익명 객체와 달리, 인스턴스 자신을 가리키는 this가 없다.  
따라서, 람다식 객체 선언에는 람다를 변환한 익명 객체를 참조할 방법이 없다.  
(컴파일러 입장에서는 람다는 코드 블록일 뿐이고, 객체가 아니므로 객체처럼 람다를 참조할 수 없다.)  

</br>

## 참고 사항
***
코틀린에서 자바로 정의된 SAM 인터페이스는 컴파일러가 자동으로 SAM 변환을 해준다. (오류를 뱉지 않는다.)  
하지만 코틀린으로 정의된 SAM 인터페이스는 SAM변환이 일어나지 않는데,  
아래와 같이 interface 키워드 앞에 fun 키워드를 붙여주면 정상적으로 SAM변환이 된다.  (kotlin 1.4에서 추가)  
(fun interface == 함수형 인터페이스)  
```kotlin
fun interface OnClickListener {  
	fun click(view: View)  
}
```

</br>

## Reference
***
https://onlyfor-me-blog.tistory.com/808  
https://kotlinlang.org/docs/fun-interfaces.html  
[kotlin in action 도서 5장 - 람다로 프로그래밍 ](https://www.yes24.com/Product/Goods/55148593)