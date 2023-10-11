## final 키워드

`final` 키워드는 프로그래밍 언어에서 ‘constant’, ‘상수’와 같은 단어와 비교되는 단어이다.
자바에서 기본적으로  **final은 해당 entity가 오로지 한 번 할당될 수 있음을 의미한다.**
#### final 멤버 변수가 반드시 상수는 아니다
왜냐면 `final`의 정의가 ‘상수이다’가 아니라 ‘한 번만 초기화 가능하다’이기 때문
```java
public class Test {
  private final int value;

  public Test(int value) {
    this.value = value;
  }

  public int getValue() {
    return value;
  }
}
```
이 코드에서 final 멤버 변수 `value`는 생성자를 통해 초기화 되었다. 
즉 이 클래스의 인스턴스들은 각기 다른 `value` 값을 갖게 된다. 
각 인스턴스 안에서는 변하지 않겠지만, 클래스 레벨에서 통용되는 상수라고는 할 수 없다.

## static 키워드
`static` 키워드는 프로그래밍 언어에서 ‘전역’, ‘정적’의 의미로 통용된다.
> **static은 해당 데이터의 메모리 할당을 컴파일 시간에 할 것임을 의미한다. (정적 바인딩)**

이에 `static` 데이터는 런타임 중에 필요할 때마다 동적으로 할당 및 해제되는 동적 데이터와는 기능과 역할이 구분된다. 
**동적 데이터와 달리, `static` 데이터는 프로그램 실행 직후부터 끝날 때까지 메모리 수명이 유지된다.**

## static final
클래스에서 사용할 해당 멤버 변수의 데이터와 그 의미, 용도를 고정시키겠다라는 의도,
해당 클래스를 쓸 때 변하지 않고, 모든 클래스 인스턴스에서도 계속 일관된 값으로 쓸 것을 멤버 상수로 지정하고자 할 때 사용한다.
**인스턴스가 만들어질 때마다 새로 메모리를 잡고 초기화시키지 말고**, 클래스 레벨에서 한 번만 잡아서 하나의 메모리 공간을 쭉 쓰면 같은 값을 가질 데이터를 위해 인스턴스 생성마다 매번 같은 메모리를 잡는 것보다 더 효율적이기 때문 
(상수로 만들 의도였으니  없고 동시성 문제도 없음으로)

## Reference
***
https://djkeh.github.io/articles/Why-should-final-member-variables-be-conventionally-static-in-Java-kor/
