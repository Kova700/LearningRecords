자바에서는 상속 시에 메소드는 override가 가능하지만, 멤버 변수 override가 안되는데
코틀린에서는 상위 타입의 프로퍼티 override가 가능하다.
그 이유가 궁금해서 찾아서 정리해본다.

자바에서
상위 클래스에 정의된 **메소드**와 하위 클래스에 정의된 **메소드**의 이름과 시그니처가 같다면
하위 클래스의 인스턴스로 해당 메소드를 호출했을 때, **하위 클래스에 정의된 메소드가 동작한다.**
즉, 메소드 override가 적용된다.

그렇다면,
상위 클래스에 정의된 **멤버 변수**와 하위 클래스에 정의된 **멤버 변수**의 이름이 같을 때,
하위 클래스의 인스턴스로 해당 멤버 변수를 수정하거나, 호출하면, 하위 클래스에 정의된 멤버 변수가 수정되고 호출될까?

정답은 **아니다.**
아래의 코드를 보자.
```java
class Parent {
    int a = 10;
    
    public void changeA() {
        this.a = 20;
    }
    
    public void shout() {
	    System.out.println("나는 Parent!!");
    }
}

class Child extends Parent {
    int a = 30;
    
	public void shout() {
	    System.out.println("나는 Child!!");
    }
}

public class MainClass {
    public static void main(String args[]) {
        Child child = new Child();

		System.out.println(child.a); //30
        child.changeA();
        System.out.println(child.a); //30 (20이 아닌 30이 출력됨)
        child.shout();               // 나는 Child!! (메소드 overriding)
        
        Parent parent = child;
        System.out.println(parent.a); //20 (상위 타입에 정의된 a변수가 20으로 변경됨)
    }
}
```
메소드는 override된 메소드를 잘 찾아서 호출하지만,
멤버 변수는 override(덮어쓰기)되지 않고, 상위 클래스의 멤버 변수와 하위 클래스 멤버 변수가 따로 존재하고 있는 걸 볼 수 있다.
즉, 자바에서는 메소드는 override는 가능하지만, 멤버 변수는 override가 불가능하다.

a 변수가 override되지 않기 때문에,
상위 클래스에 정의된 changeA() 메소드는 하위 타입에 정의된 a 변수에 접근하지 않고, 가장 가까이 있는 상위 타입에 정의된 a변수를 수정하게 된다.
또한, a 변수가 override되지 않기 때문에, 
인스턴스 참조 변수의 타입이 어떤 타입이냐에 따라, 같은 이름의 멤버 변수 일지라도, 참조하는 값이 달라진다.

그렇다면 코틀린은 어떨까?
아래의 코드를 보자.
```kotlin
open class Parent {
    open var a = 10
    var b = 1

    fun changeA(){
        a = 20
    }
    
    open fun shout(){
        println("나는 Parent!!")
    }
}

class Child : Parent() {
    override var a = 30
    
    override fun shout(){
        println("나는 Child!!")
    }
    
    fun printSueprA(){
        println(super.a)
    }
}

fun main(){
    val child = Child()
    println(child.a) 	//30
    
    child.changeA()
    println(child.a) 	//20 (자바와 달리 20으로 변경되었다.)
    child.shout()		// 나는 Child!! (메소드 overriding)
    
    child.printSueprA() //10 (상위 타입의 변수는 그대로 10 유지)
}
```
코틀린에서는 changeA() 함수가 하위 타입의 프로퍼티를 20으로 잘 변경시켰다. (override가 잘 되었다.)
분명 멤버 변수 override는 안될텐데
상위 클래스에 정의된 함수로 하위 클래스에 정의된 프로퍼티의 값을 어떻게 변경 시켰을까?

코틀린의 프로퍼티(Property)는
프로퍼티의 별다른 getter, setter를 정의하지 않으면,
코틀린 컴파일러가 내부적으로 기본적인 getter와 setter를 작성해준다.
기본적으로 작성된 getter와 setter를 통해서 값을 호출하고 수정한다.

즉, 코틀린 코드는 자바 코드로 따지면,
자바의 멤버 변수에 getter , setter변수가 정의된 상태라는 것
자바 코드로 보자면 아래와 같다
```java
class Parent {
    int a = 10;

    int getA() {
        return this.a;
    }

    void setA(int value) {
        this.a = value;
    }
    
    public void changeA() {
        this.setA(20);
    }
    
}

class Child extends Parent {
    int a = 30;
    
    int getA() {
        return this.a;
    }

    void setA(int value) {
        this.a = value;
    }
    
}

public class MainClass {
    public static void main(String args[]) {
        Child child = new Child();

        System.out.println(child.getA()); //30
        child.changeA();
        System.out.println(child.a); //20
        
        Parent parent = child;
        System.out.println(parent.a); //10
    }
}
```
뭔가 감이 오지 않는가.

**getter와 setter의 메소드명과 시그니처가 같다. (default 생성자)**
**메소드 overriding이  발생해서, 상위 클래스에 정의된 changeA() 메소드가 overriding된 Child 클래스의 setA()를 호출하고 있다.**
그래서 정상적으로 상위 클래스에 정의된 메소드로 하위 클래스의 변수를 수정할 수 있었던 것이었던 것이었다.
(맨 아래 나온 10처럼 상위 타입의 a변수는 여전히 메모리에 따로 존재한다.)

아래는 위에 올렸던 코틀린 코드를 자바 코드로 디컴파일한 코드다.
**open 키워드를 표시한 a 프로퍼티만 getter, setter 앞에 final이 빠져있음을 확인할 수 있다.**
```java
public class Parent { //final이 빠짐
   private int a = 10;
   private int b = 1;

   public int getA() { //final이 빠짐
      return this.a;
   }

   public void setA(int var1) { //final이 빠짐
      this.a = var1;
   }

   public final int getB() {
      return this.b;
   }

   public final void setB(int var1) {
      this.b = var1;
   }

   public final void changeA() {
      this.setA(20);
   }

   public void shout() {
      String var1 = "나는 Parent!!";
      System.out.println(var1);
   }
}

public final class Child extends Parent {
   private int a = 30;

   public int getA() {
      return this.a;
   }

   public void setA(int var1) {
      this.a = var1;
   }

   public void shout() {
      String var1 = "나는 Child!!";
      System.out.println(var1);
   }

   public final void printSueprA() {
      int var1 = super.getA();
      System.out.println(var1);
   }
}

public final class KotlinInterfaceTestKt {
   public static final void main() {
      Child child = new Child();
      int var1 = child.getA();
      System.out.println(var1); //30
      
      child.changeA();
      var1 = child.getA();
      System.out.println(var1); //20
      child.shout();
      
      child.printSueprA(); //10
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```


## Reference
***

https://devstep.tistory.com/79
https://hevton.tistory.com/335