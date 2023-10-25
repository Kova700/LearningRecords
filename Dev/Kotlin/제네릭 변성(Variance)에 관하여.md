## 변성(Variance)이란?
***
List\<String> 과 List\<Any>와 같이  **기저 타입**이 같고 **타입 인자**가 다른 여러 타입이  
서로 어떤 관계가 있는지 설명하는 개념이다.  (kotlin in action에 정의된 설명이다.)  
> 위 예시에서 **기저타입**은 **제네릭을 품고 있는 클래스(List)** 를 의미하고,   
> 타입 인자란 **제네릭 안에 있는 타입 파라미터(String, Any)** 를 의미한다.    

위 설명이 잘 와닿지 않는다면 아래의 예시들을 보자.  

Any는 String의 상위 클래스다.  
그래서 아래와 같이 Any타입으로 선언된 변수에 String 타입 변수의 데이터를 집어넣을 수 있다.  
```kotlin
val string = "asd"  
val any :Any = string
println(any)  // asd (가능)
```

Any타입으로 선언된 변수에 String 타입의 변수 데이터를 집어넣을 수 있다면,  
List\<Any>타입 변수에  List\<String>타입 변수의 데이터도 집어넣을 수 있지 않을까?  
```kotlin
val stringList = listOf("a")  
val anyList :List<Any> = stringList
println(anyList)  // [a] (가능)
```

오... 그렇다면   
MutableList\<Any>타입 변수에  MutableList\<String>타입 변수의 데이터도 집어넣을 수 있지 않을까?   
```kotlin
val stringMutableList = mutableListOf("n")  
val anyMutableList :MutableList<Any> = stringMutableList //Type mismatch. (불가능)

//IDE가 stringMutableList 부분에 빨간줄을 그어주면서 경고문을 날린다.
//Type mismatch.
//Required: MutableList<Any>
//Found: MutableList<String>
```
 List\<Any>타입 변수에  List\<String>타입 변수의 데이터를 넣는 것은 가능한데,  
 MutableList\<Any>타입 변수에  MutableList\<String>타입 변수의 데이터는 왜 넣을 수 없는 걸까  

이유는 뒤에서 설명을 하겠지만,   
간단하게 설명하면 코틀린에서 **List**는 **공변성**을 가지고, **MutableList**는 **불(무)공변성**을 가지기 때문이다.  
변성의 종류로는 **공변성**, **불공변성**, **반공변성** 이 있는데 아래에서 각 **변성**에 대해서 알아보자.  

</br>

## 변성의 종류
***
- **공변** (**co(함께)** variant(변한다)) = 같이 변한다.(타입 파라미터의 상속 관계를 제네릭 클래스에서도 가져간다.)   
- **불(무)공변** (**in(X)** variant(변한다)) = 같이 변하지 않는다. (타입 파라미터의 상속 관계와 제네릭 클래스의 상속 관계는 무관하다)   
- **반공변** (**contra(반대로)** variant(변한다)) = 반대로 변한다.  (타입 파라미터의 상속 관계와 정반대의 상속 관계를 제네릭 클래스가 가진다.)   

간단하게 설명하면 변성은 위 세 가지로 구분된다.  
하나씩 각 변성에 대해 알아보자.  
(편의상 타입 간 상하 관계를 상속 관계(포함 관계)라고 표현했다. )
### **공변성(covariant)**
***
위의 예시에서 List는 타입 파라미터의 상속 관계를 그대로 따라간다는 걸 코드를 통해서 봤다.  
```kotlin
val stringList = listOf("a")  
val anyList :List<Any> = stringList
println(anyList)  // [a] (가능)
```
List와 같이 **타입 파라미터의 상속 관계를 제네릭 클래스에서도 그대로 가져갈 때**, **공변성을 가진다**라고 한다.  
이러한 공변성을 가지는 제네릭 클래스를 정의하려면 **out 키워드**를 타입 파라미터에 붙여줘야 한다.  

아래는 List 인터페이스의 선언부다.   
```kotlin
public interface List<out E> : Collection<E> {
			...
}
```
out키워드가 타입 파라미터 왼쪽에 적혀있는 걸 볼 수 있다.  

</br>

### **불공변성(invariant)**
***
위의 예시에서 MutableList는 타입 파라미터의 상속 관계를 따르지 않았다.  
그래서 오류가 IDE가 "Type mismatch."라고 경고를 날려주지 않았나.  
```kotlin
val stringMutableList = mutableListOf("n")  
val anyMutableList :MutableList<Any> = stringMutableList //Type mismatch. (불가능)
```
MutableList와 같이 **타입 파라미터의 상속 관계와 제네릭 클래스의 상속 관계가 무관**할 때, **불(무)공변성을 가진다**고 한다.  
out키워드가 공변성을 나타내는 키워드인 것과는 달리, 불공변성을 가질 때는 타입 파라미터 앞에 아무 키워드도 적지 않는다.  
(== 제네릭 클래스는 기본적으로 불공변성을 가진다.)

아래는 MutableList 인터페이스의 선언부다.    
```kotlin
public interface MutableList<E> : List<E>, MutableCollection<E> {
			...
}
```
List인터페이스와 달리 타입 파라미터 E 앞에 어떠한 키워드도 없는 걸 확인할 수 있다.  

</br>

### **반공변성(contravariant)**
***
반공변성은 파라미터의 상속 관계와 제네릭 클래스의 상속 관계가 정반대로 정의된다.  
(A가 상위 타입이고 B가 하위 타입이라면, Class\<A> 는 Class\<B>의 하위 타입으로 정의된다.)  
반공변성을 가지는 제네릭 클래스를 정의하려면 **in 키워드**를 타입 파라미터에 붙여줘야 한다.  

### 솔직히 어떤 의미인지는 잘 알겠는데, 상속 관계를 뒤집는 반공변성이 왜 필요한가 잘 와닿지 않는다.  

예제를 보면서 왜 필요한지 알아보자.  
아래는 Collection을 Comparator내용을 기준으로 정렬한 List를 반환해주는 sortedWith 확장 함수의 선언부다.  
```kotlin
public fun <T> Iterable<T>.sortedWith(comparator: Comparator<in T>): List<T> { 
	//클래스가 아닌 함수 파라미터에 반공변성 키워드인 in이 적혀있다. (comparator: Comparator<in T>) <- 여기
	//이런 경우를 "사용 지점 변성" 이라 한다. 
	//(클래스 전체에 사용되는 타입 파라미터에 변성을 적용하는 게 아닌 해당 함수 파라미터에서만 반공변성이 적용된다)
	//(List처럼 클래스 선언부 타입 파라미터에 변성을 지정하는 것은 "선언 지점 변성")
    ...
}
```

Comparator는 객체 간 대소 비교(>,<, >=, <=)를 할 때, 비교 기준을 정의 하는 인터페이스로써,  
Collection을 정렬할 때, 정렬 기준을 제시할 때 자주 사용된다.  
인터페이스 내부 compare함수에 어떤 프로퍼티 값으로 객체 간 대소를 비교하면 되는지 명시해서 override 해주면 된다.   
(단일 추상 클래스라서 SAM 변환이 가능해 람다표현식으로 인스턴스 생성이 가능하다.)  
```kotlin
sealed interface Car{
    val productName: String
    val price: Int
}

data class Tesla(
    override val productName: String,
    override val price: Int
) : Car

data class Hyundai(
    override val productName: String,
    override val price: Int
) : Car
```

위와 같이 Car 인터페이스를 상속한 Tesla와 Hyundai 클래스가 있다고 할 때,   

```kotlin
fun main() {
    val HyundaiCarList = listOf(
        Hyundai("a",3),
        Hyundai("b",2),
        Hyundai("c",1),
    )
    
    val TeslaCarList = listOf(
        Tesla("a",3),
        Tesla("b",2),
        Tesla("c",1),
    )

    val carComparator = Comparator<Car> { car, otherCar ->
        car.price - otherCar.price
    }

    val HyundaiCarComparator = Comparator<Hyundai> { car, otherCar ->
        car.price - otherCar.price
    }

    val TeslaCarComparator = Comparator<Tesla> { car, otherCar ->
        car.price - otherCar.price
    }

    println(TeslaCarList.sortedWith(carComparator)) // sortedWith는 Comparator<Tesla>를 원하지만, Comparator<Car>로도 사용이 가능
    println(TeslaCarList.sortedWith(TeslaCarComparator))
    println(HyundaiCarList.sortedWith(carComparator)) // sortedWith는 Comparator<Hyundai>를 원하지만, Comparator<Car>로도 사용이 가능
    println(HyundaiCarList.sortedWith(HyundaiCarComparator))
}
```

26번째, TeslaCarList.sortedWith()에 Comparator\<Car>타입 carComparator가 전달되었다.  
sortedWith() 함수 선언부를 다시 보자.  

```kotlin
public fun <T> Iterable<T>.sortedWith(comparator: Comparator<in T>): List<T> { 
    ...
}
```
분명 Iterable의 타입 파라미터 T에 Tesla 타입이 전달 되었을텐데, 
comparator위치에 Comparator\<Tesla>가 아닌 Comparator\<Car>가 와도 컴파일러가 오류를 뱉지 않는다.  

**반공변성**이 정의되면,   
Car     **<-**     Tesla, Hyundai 관계였던 상속 관계가   
Comparator\<Car>     **->**     Comparator\<Tesla>, Comparator\<Hyundai>로 바뀌기 때문에   
Comparator\<Tesla>, Comparator\<Hyundai> 타입 파라미터 자리에 Comparator\<Car>타입의 변수를 집어넣을 수 있어진다.   

고로, **상위 타입에 정의된 내용(프로퍼티 등)으로 하위 타입에도 똑같은 로직**을 적용할 수 있게 하려면 **반공변성**이 방법이 될 수 있다.  
(여기선 Car라는 부모 타입에 정의된 프로퍼티를 이용한 로직을 하위 타입들의 비교에 사용 했다.)  

</br>

## 그래서 List는 공변성을 가지고, MutableList는 불공변성을 가지는 이유는 뭘까?
***
맨 위에서 MutableList로 예들 들었던 상황에서  
컴파일러가 오류를 뱉지 않는다면 런타임에서 어떤 상황이 벌어질지 생각해보자.  
(MutableList가 공변성을 가진다면 벌어지는 상황)  

```kotlin
val stringMutableList = mutableListOf("n")  
val anyMutableList :MutableList<Any> = stringMutableList  //Type mismatch.
anyMutableList.add(26)
```
MutableList\<String> 타입으로 정의된 stringMutableList타입에 Int타입인 26이 들어가는 상황이 생겼다.  
이제 MutableList\<String>처럼 제네릭으로 타입 파라미터를 명시했다 한들,   
MutableList\<String>타입의 인스턴스에 String타입만 들어있다고 보장할 수 없어졌다.  
즉, 제네릭 클래스의 **타입 안정성이 사라졌다.**  
그래서 코틀린 컴파일러는 위와 같은 상황에 MutableList\<String> -> MutableList\<Any> 으로 타입 캐스팅을 허용하지 않는다.  

왜 저런 상황이 벌어진 걸까 생각해보자.  
MutableList의 add()함수의 선언부를 보면 타입 파라미터가 함수의 파라미터에 적혀있는 걸 볼 수 있다.  
```kotlin
public interface MutableList<E> : List<E>, MutableCollection<E> {
	override fun add(element: E): Boolean //함수의 파라미터에 타입 파라미터가 있다.
	...
}
```
stringMutableList의 타입은 MutableList\<String>이고, anyMutableList의 타입은 MutableList\<Any>이다.  
anyMutableList의 타입은 MutableList\<Any>임으로 anyMutableList.add() 함수의 파라미터 타입은 Any다. (T에 Any가 들어간 것)  
그래서 Int 타입 데이터를 넣을 수 있게 stringMutableList가 업케스팅된 것이다.  
이렇게 되면, 위에서 말한 것처럼 **타입 안정성이 사라지게 된다.**  

그래서 코틀린 컴파일러는 이러한 상황을 막기 위해 **변성 규칙**을 준수해서 제네릭 클래스를 정의하게 강제한다. 
(함부로 업케스팅, 다운 캐스팅을 못하게 제네릭 클래스를 정의할 때부터 미연에 방지하기 위해)  
변성 규칙을 알면 **공변성을 가질 때** 왜 **out키워드**를 쓰는지, **반공변성을 가질 때** 왜 **in키워드**를 쓰는지 알 수 있다.  
아래에서 변성 규칙에 대해 알아보자.   

</br>

## **변성 규칙**
***
1. 제네릭 클래스 내부에서 타입 파라미터가 **out위치**에만 쓰이는 경우에만 **공변성**을 가질 수 있다.
2. 제네릭 클래스 내부에서 타입 파라미터가 **in위치**에만 쓰이는 경우에만 **반공변성**을 가질 수 있다.
3. 제네릭 클래스 내부에서 타입 파라미터가 **out위치, in위치 둘 다** 쓰이는 경우는 **공변성, 반공변성을 가질 수 없다. == 불공변성을 가진다.**
(생성자 파라미터는 in위치 out위치 둘 중 어느 곳에도 해당하지 않는다. == 생성자는 어디에 T가 있던 상관 없다는 뜻)

out 위치는 함수의 반환 타입 위치를 지칭하고,  
(나가는 위치라서 Out 키워드 사용)
```kotlin
interface List<out T> :Collection <T> {  
	operator fun get(index :Int) : T //반환위치에 T가 존재
	fun subList(fromIndex :Int, toIndex: Int) :List<T> //반환위치에 T가 존재 
	//...  
}
```

in 위치는 함수의 파라미터 위치를 지칭한다.  
(들어오는 위치라서 In 키워드 사용)
```kotlin
  
interface Comparator<in T> {  
	fun compare(e1: T, e2: T): Int {  //함수 파라미터 위치에 T가 존재
		...  
	}  
}
```

### 변성 규칙을 가지지 않는다면 벌어질 수 있는 상황
***
#### 공변성을 가지는 제네릭 클래스가 in위치에 타입 파라미터가 존재할 경우
출처 : [stackoverflow](https://stackoverflow.com/questions/65776218/why-it-is-forbidden-to-use-out-keyword-in-generics-if-a-method-excepts-the-typ)
```kotlin
open class Animal

class Cat : Animal() {
    fun meow() = println("meow")
}

class Foo<out T : Animal> {
    private var animal: T? = null

    fun consumeValue(x: T) { // NOT ALLOWED
        animal = x
    }

    fun produceValue(): T? {
        return animal
    }
}

private fun main() {
    val catConsumer = Foo<Cat>()
    val animalConsumer: Foo<Animal> = catConsumer // upcasting is valid for covariant type
    animalConsumer.consumeValue(Animal())
    catConsumer.produceValue()?.meow() // can't call `meow` on plain Animal
}
```

#### 반공변성을 가지는 제네릭 클래스가 in위치에 타입 파라미터가 존재할 경우
출처 : [stackoverflow](https://stackoverflow.com/questions/65776218/why-it-is-forbidden-to-use-out-keyword-in-generics-if-a-method-excepts-the-typ)
```kotlin

class Bar<in T: Animal>(private val library: List<T>) {
    fun produceValue(): T  { // NOT ALLOWED
        return library.random()
    }
}

private fun main(){
    val animalProducer: Bar<Animal> = Bar(List(5) { Animal() })
    val catProducer: Bar<Cat> = animalProducer // downcasting is valid for contravariant type
    catProducer.produceValue().meow() // can't call `meow` on plain Animal
}
```

## 추가로 (정리 중)
***
자바에서 배열은 공변성을 가지지만, 코틀린에서는 배열은 불공변성을 가진다.

이펙티브 자바 내용
자바에서는 배열은 공변 (상위타입으로 캐스팅 가능해서, 이상한 거 넣을 수 있음),
리스트는 불공변을 가진다.
그래서 배열은 런타임에 에러가 터지고, 리스트는 컴파일에 에러가 터진다.
그래서 자바에서는 아래와 같은 상황에 Array는 런타임에 에러가 터질 수 있다.
```java
Object[] objectArray = new Long[1];
objectArray[0] = "asd" // runtime exception
```
코틀린에서는
배열은 불공변성을 가지고,
List는 공변성, MutableList는 불공변성을 가진다.
그래서 코틀린에서는 아래와 같은 상황에 List는 런타임 에러가 터질 수 있다.
```kotlin
val stringLists = Array<List<String>>(1) { emptyList() }  
val objects = stringLists as Array<List<Any>> // unchecked cast  
objects[0] = listOf(42)  
val s = stringLists[0][0] // runtime exception here
```

</br>

## Reference
***
[kotlin in action 9장 - 제네릭스](https://www.yes24.com/Product/Goods/55148593)   
[Effective Java - Item 28 : 배열보다는 리스트를 사용하라](https://www.yes24.com/Product/Goods/65551284)   
https://appmattus.medium.com/effective-kotlin-item-28-prefer-lists-to-arrays-c597d8dfa335   
https://medium.com/kotlin-thursdays/introduction-to-kotlin-generics-9d18d3719e1d  
https://kotlinlang.org/docs/generics.html#generics  
https://stackoverflow.com/questions/65776218/why-it-is-forbidden-to-use-out-keyword-in-generics-if-a-method-excepts-the-typ  