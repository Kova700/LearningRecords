```java
public class Singleton {
    private static Singleton instance;

    private Singleton(){}
    
    public static Singleton getInstance(){
    	if(instance == null) {
    		instance = new Singleton();
    	}
        return instance;
    }
    
    ...
}
```
문제점 : 
multi - thread 환경에서 스레드들이 동시에 getInstance()메서드에 접근하게 되면, 여러 개의 인스턴스가 생성될 수 있음,  
- 스레드들이 동시에 instance변수를 보고 있다고 하더라고, CPU 캐시에 담긴 값이 다를 수 있음 (가시성 문제)  
- 스레드가 instance변수에 값을 할당하고 있다고 하더라도, 할당 연산이 원자적이지 않음으로, 할당 중인데도 불구하고, 다른 스레드는 할당되어있지 않다고 판단해서 할당을 시도하러 온다.  

1. 그래서 volatile 키워드로 스레드 간에 instance변수의 가시성을 해결해 주고, 
2. instance변수에 값 할당의 원자성을 보장해줘야 한다. (syncronized , Atomic같은) 
(synchronized를 쓰면 블럭 진입 전,후 자동으로 CPU캐시와 메모리의 동기화가 일어남으로 가시성 문제는 해결된다.)  
## 해결책 1. synchronized

```java
public class Singleton {
    private static Singleton instance;

    private Singleton(){}
    
    public static synchronized Singleton getInstance(){
    	if(instance == null) {
    		instance = new Singleton();
    	}
        return instance;
    }
    
    ...
}
```
원자성 문제와 가시성 문제가 해결되었지만,  
`synchronized` 키워드가 선언되어져 있으면 그 블록은 한 시점에 한 쓰레드만이 블록 안으로의 접근이 허용되기 때문에  
`getInstance` 메소드로 수많은 접근 요청이 들어올 경우, 
**병목(bottleneck)현상**이 생겨  **속도 저하의 성능 문제가 발생할 수 있다** (**이미 인스턴스가 만들어져있음에도 불구하고 lock에 걸려 병목현상이 발생할 수 있기 때문**)  

## 해결책 2. Double- checking Locking
```java
public class Singleton {
    private static Singleton instance;

    private Singleton(){}
    
    public static Singleton getInstance(){    	
    	if(instance == null) {
    	    synchronized (Singleton.class) {
				if(instance == null) {
				    instance = new Singleton();
				}
		    }
    	}
        return instance;
    }
    
    ...
}
```
DCL은 이름 그대로 인스턴스의 존재를 두 번 확인하는 방법  
**이미 인스턴스가 만들어져있다면 Lock이 걸리지 않음으로써, 병목 현상을 줄 일 수 있다.**  
  
하지만, new를 사용하여 객체를 생성할 때   
**메모리 공간의 확보-변수에 메모리 공간 링크-해당 메모리에 오브젝트 생성** 순으로 작업이 이루어진다.  
이 때 아래와 같은 문제가 발생할 수 있다.  

1. Thread1은 '변수에 메모리 공간 링크'만 된 상태  
2. Thread2가 첫번째 if문에 걸림. 오브젝트가 생성되었다고 착각  
3. Thread2가 getInstance()함수를 탈출, 해당 메모리 공간(오브젝트가 아직 미생성) 으로 작업을 하려고 함.  
4. 오브젝트가 생성되지 않았기 때문에 에러 발생  
```java
public class Singleton{
	private static Singleton uniqueInstance;
	private Singleton() {}

	public static Singleton getInstance(){
		if(uniqueInstance == null){
			synchronized (Singleton.class){
				if(uniqueInstance == null){
					some_memory_space = aloocate space for Singleton object; //메모리 공간 생성
					uniqueInstance = some_memory_space; //다른 스레드입장에서는 값이 할당되었다고 생각함(하지만 내용물이 없음)
					create a real object in some_memory_space;  // (내용물 생성)
				}
			}
		}
		return uniqueInstance;
	}
}
```
그래서 volatile 키워드로 원자성을 보장해주는것 이 좋다.  
Atomic 클래스를 사용할 때, 원자성을 보존해주는 내용과 비슷한 맥락이 있긴한데,  
volatile 키워드를 사용하면 해당 메모리 공간이 할당되고,   
그 공간에 내용물이 생성될 때까지 barrier가 생성된다고 생각하면 되고, ([참조](https://stackoverflow.com/questions/11639746/what-is-the-point-of-making-the-singleton-instance-volatile-while-using-double-l))  
Atomic 클래스는 i++같이 값을 가져와서 증가하고 다시 할당하는 과정에서 가져온 값의 불일치가 생기는 상황에,  
잘못된 값을 덮어쓰기 하는 상황에 발생하는 문제를 해결하기 위해서 사용하는 느낌이다.  

[volatile](volatile_키워드는_왜_쓰는걸까)이 그냥 가시성의 문제만 해결해준다고 생각하기보단   
위의 개념처럼 **메모리 할당에 있어서 원자성을 지켜준다고도** 생각하면 될 듯하다.  

```java
public class Singleton {
    private volatile static Singleton instance;

    private Singleton(){}
    
    public static Singleton getInstance(){    	
    	if(instance == null) {
    	    synchronized (Singleton.class) {
				if(instance == null) {
				    instance = new Singleton();
				}
		    }
    	}
        return instance;
    }
    
    ...
}
```


## 해결책 3. Lazy - holder
***
nested class가 호출될 때, 클래스가 로드되는 특성을 이용한 싱글톤 생성 방법으로   
가장 안전한 Lazy init singleton 생성 방법으로 여겨지고 있다.  

```java
public class Singleton {
    private Singleton(){}
    
    public static Singleton getInstance(){    	
    	return SingletonLazyHolder.INSTANCE;
    }
    
    private static class SingletonLazyHolder {
    	private static final Singleton INSTANCE = new Singleton();    	
    }
    
    ...
}
```

https://sinethneranjana.medium.com/5-ways-to-write-a-singleton-and-why-you-shouldnt-1cf078562376  

## Reference
***
https://bj25.tistory.com/21  
https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%8B%B1%EA%B8%80%ED%86%A4Singleton-%ED%8C%A8%ED%84%B4-%EA%BC%BC%EA%BC%BC%ED%95%98%EA%B2%8C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90  
https://yeahhappyday.tistory.com/entry/singleton-%ED%8C%A8%ED%84%B4%EA%B3%BC-volatileDCLDouble-Checking-Locking  
https://javaplant.tistory.com/21  
https://stackoverflow.com/questions/11639746/what-is-the-point-of-making-the-singleton-instance-volatile-while-using-double-l  