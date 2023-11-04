Obejct.hashCode()는 하위 클래스로 내려오면서 재정의가 가능하다.   
보통 Obejct.hashCode() 메소드를 재정의하는 목적으로는 **간단하게 숫자만으로 객체 간에 동등성을 검사**하기 위해서 사용된다.   

예를 들면 String 클래스와 같이 객체의 내용물을 검사할 때,  
보통 equals()메소드 내부에서 재정의된 hashCode()함수의 결과 값을 비교한다.  

Obejct.hashCode()의 반환 값은 주로 인스턴스의 내용물을 hash함수를 돌려서   
객체 간의 동등성을 검사하기 위해서 사용하는 숫자임으로 인스턴스 간의 고유한 숫자가 아니다.  
내용물이 같으면 hashCode가 같아지고, 내용물이 다르면 hashCode가 달라진다.  

또한 Obejct.hashCode()는 객체의 내용물에 의해 생성됨으로   
Obejct.hashCode()가 같다는 의미가 인스턴스의 내용물이 같다는 의미이지  
인스턴스 메모리 주소가 같다는 의미가 아님을 주의하자.   

String의 경우에도 실제 메모리 주소가 달라도 내부 값이 같으면 같은 hashCode를 반환한다.   
그래서 equals()를 호출할 때 true가 반환되는 것이다.  

만약, 객체 간의 동일성을 검사하고 싶다면   
Java에서는 \==, Kotlin에서는 \=== 를 사용하면 되지만, hashcode값으로 확인하고 싶다면
System.identityHashCode()를 이용하면 메모리 주소를 기반으로 생성된 hashcode를 얻을 수 있다.
이렇게 얻은 hashcode는 메모리 구조를 기반으로 생성되었기 때문에 인스턴스 간의 고유한 숫자이다.
고로 인스턴스 간의 동일성 검사가 가능하다.   

</br> 

## Reference
***
https://codechacha.com/ko/java-system-identityhashcode/