이번 글에서는 registerForActivityResult()로 인해 발생하는 두 가지 오류에 대한 내용과   
해당 오류를 발생하지 않게 하려면 언제 registerForActivityResult를 호출해야 하는가에 관한 글을 작성해 보려 한다.  

registerForActivityResult를 Activity 혹은 Fragment에서 사용하다 보면  
#### Activity의 경우
> IllegalStateException : LifecycleOwner is attempting to register while current state is STARTED. LifecycleOwners must call register before they are STARTED.
#### Fragment의 경우
> IllegalStateException :Fragments must call registerForActivityResult() before they are created (i.e. initialization, onAttach(), or onCreate()).

와 같이  registerForActivityResult()를 특정 생명 상태 이전에 호출해야 한다고 런타임 예외를 던지는 경우를 만날 수 있다.  

#### 추가로 Fragment의 경우

> IllegalStateException : Attempting to launch an unregistered ActivityResultLauncher 

와 같이 등록되지 않은 ActivityResultLauncher를 사용하고 있다는 예외도 추가로 마주할 수 있는데,  

위 예외들은 왜 발생하고, 어떻게 해야지 방지할 수 있을 지  알아보자.

</br>

***

</br>

우선, 안드로이드 [공식문서](https://developer.android.com/training/basics/intents/result)에 따르면  
> **Note:** You must call `registerForActivityResult()` before the fragment or activity is created,  
> but you can't launch the `ActivityResultLauncher` until the fragment or activity's [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) has reached [`CREATED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#CREATED).  

번역 : fragment 또는 activity가 created되기 전에 registerForActivityResult()를 호출해야 하지만,   
fragment 혹은 activity의 생명 주기가 CREATED에 도달할 때까지 ActivityResultLauncher를 시작할 수 없습니다.  

fragment와 activity가 created되기 전 이라는게 정확히 어떤 시점을 말하는 걸까?  

</br>

Fragment와 Activity는 LifecycleOwner.getLifecycle 메서드가 있는 LifecycleOwner 인터페이스를 구현한다.  
LifecycleOwner는 Lifecycle을 가지고, Lifecycle은 아래와 같이 5가지의 [State](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State)로 현재 Lifecycle의 상태를 나타내고,  
7가지의 [Event](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event)로 현재 어떤 Lifecycle 콜백 위에있는 지를 나타낸다.  
ex : CREATED state에서  STARTED state로 가는 ON_START lifecycle 콜백 위에 있다.  
(Lifecycle class 공식문서 : https://developer.android.com/reference/androidx/lifecycle/Lifecycle )  
![LifecycleEvent&State|500][Dev/Res/img-20231106.png]
(출처 : https://developer.android.com/topic/libraries/architecture/lifecycle)  
```java
public enum class Event {
	ON_CREATE, ON_START, ON_RESUME, ON_PAUSE, ON_STOP, ON_DESTROY, ON_ANY;
}
```
```java
public enum class State {  
    INITIALIZED, DESTROYED,  CREATED,  STARTED, RESUMED 
}
```
Fragment 내부에서 Fragment의 콜백 State를 나타내기 위한 상수들이 있는데 LifecycleOwner의 Lifecycle 상수들과 혼동하지 말자,    
(Fragment 클래스 내부에서 mState 프로퍼티는 Fragment의 콜백 State를 나타내기 위한 프로퍼티고,    
LifecycleOwner의 Lifecycle state는 mLifecycleRegistry.handleLifecycleEvent()함수에 Lifecycle.event를 전달함으로써 mLifecycleRegistry내부적으로 관리한다.  
(mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE); 식으로 호출하면, Event에 연결된 다음 State로 변환해서 저장함) ) 

Fragment클래스에 정의된 Fragment의 콜백 state 상수  
![fragmentStateConst](img-20231105-4.png)
Activity와 Fragment의 생명 주기에 대해서 이야기를 계속하면  
내용이 길어질듯 해, 각 생명 주기에 대한 자세한 이야기는 공식 문서를 참고하자.  

## Fragment 먼저 알아보자.
***

</br>

fragment에서 registerForActivityResult를 사용하기 위해서는  
fragment가 created되기 전에, registerForActivityResult를 호출해야 한다고 했다.

fragment의 생명주기는 아래의 그림과 같은데,  
fragment가 created되기 전은 어떤 시점을 말하는 걸까.  
FragmentLifcycle의 CREATED와 ViewLifecycle의 CREATED중 어느 것을 따라야 할지도 모르겠다.  

![Fragment생명주기|500](/Dev/Res/img-20231104.png)
(출처 : https://developer.android.com/guide/fragments/lifecycle)
잘 모르겠다면 하나하나 다 찍어보면 된다.  

</br>

#### onAttach( )
![[img-20231104-2.png]]  
결과 : 예외 터지지 않음  

#### onCreate( )
![[img-20231104-3.png]]
결과 : 예외 터지지 않음  

</br>

#### onCreateView( )
![[img-20231104-4.png]]
결과 : 예외 터지지 않음  

</br>

#### onViewCreated( )
![[img-20231104-5.png]]
결과 : 예외 터지지 않음  

</br>

#### onViewStateRestored( )
![[img-20231104-6.png]]
결과 :  
![[img-20231104-7.png]]
> java.lang.IllegalStateException: Fragment HomeFragment{297caeb} (ab26aa4e-445e-42c2-a4ff-2b294a70796d id=0x7f080080) is attempting to registerForActivityResult after being created. Fragments must call registerForActivityResult() before they are created (i.e. initialization, onAttach(), or onCreate()).

**IllegalStateException예외가 터지면서 registerForActivityResult 호출을     
Fragment가 created되기 전인 onAttach(), or onCreate())에서하라고 경고해준다.**  
![[img-20231105-3.png]]

그러니까 **Fragment가 create되기 전**이라고 한다면  
위에 Fragment lifeCycle 이미지에 나왔던 **Fragment의 ViewLifeCycle이 CREATED 상태가 되기 전**의 상태를 의미한다고 볼 수 있다.  
**고로 onViewStateRestored( )이전에 registerForActivityResult( )를 호출하라는 결론이 나온다.**  

약간 TMI지만 사실 내부적으로는  
아까 위에 말했듯이 mState라는 Fragment내부 프로퍼티가 Fragment의 lifecycle 콜백을 돌면서 mState 프로퍼티에 각 콜백에 맞는 값을 할당한다.  
onViewStateRestored()는 performActivityCreated() 메소드 안에서 호출되는데, 아래와 같이 mstate값을 CREATED상수보다 큰 숫자로 변경 시켜버린다.  
![600](Dev/Res/img-20231106-1.png)
![fragmentStateConst](img-20231105-4.png)

</br>

### **그냥 어쨌든, 내부적으로 어떻든 간에**   
### **onViewStateRestored( )이전에 registerForActivityResult( )를 호출하면 된다**  


## 멤버 변수에서 registerForActivityResult()호출
***

</br>

Fragment의 생명 주기와 별개로 Fragment 클래스의 멤버 변수로 registerForActivityResult( )를 호출한 결과 값인  
ActivityResultLauncher<Intent?>타입의 변수를 두면 Fragment가 생성될 때, 초기화 됨으로,  
Fragment의 View가 생성되기 전에, registerForActivityResult( )를 호출하라는 공식문서의 내용대로 사용이 가능하다.   
ex :   
```kotlin
//Fragment 내부부
val getContent = registerForActivityResult(GetContent()) { uri: Uri? ->
    // Handle the returned Uri
}

override fun onCreate(savedInstanceState: Bundle?) {
    // ...
    val selectButton = findViewById<Button>(R.id.select_button)
    selectButton.setOnClickListener {
        // Pass in the mime type you want to let the user select
        // as the input
        getContent.launch("image/*")
    }
}
```
출처 : [공식문서](https://developer.android.com/training/basics/intents/result) (공식 문서는 Activity를 예로 들고 있지만 Fragment에도 사용할 수 있는 구조라서 가져와봤다.)  

### **하지만, Fragment에서 위 구조를 가질 때, 주의할 점이 있다.**  

아래와 같이 매번 새로운 Fragment로 replace하면 매번 Fragment를 replace할 때마다,   
Fragment의 멤버 변수로 정의된 ActivityResultLauncher도 자동으로 Activity에 register되니까  
문제가 되지 않는데,  
```kotlin
supportFragmentManager.beginTransaction()  
    .setReorderingAllowed(true)  
    .replace(R.id.container_main, TargetFragment()) //매번 객체 생성성
    .commitNow()
```

</br>

아래와 같이 기존에 만들어놓은 Fragment를 가지고 replace 전환을 할 때는  
Fragment 인스턴스가 생성될 때, 멤버 변수도 초기화 됨으로 최초 1회에 한해서만 ActivityResultLauncher가 초기화되는데(register 됨),  
이 점이  문제가 된다.  

```kotlin
//Activity 내부 코드 중
val homeFragment: HomeFragment by lazy { HomeFragment() }  
val wishFragment: WishFragment by lazy { WishFragment() }

private fun initBottomNavigationView() {  
    bottomNavigationView.setOnItemSelectedListener { menuItem ->  
        when (menuItem.itemId) {  
            R.id.bottom_menu_home -> showFragment(homeFragment)  
            R.id.bottom_menu_wish -> showFragment(wishFragment)  
        }  
        true  
    }  
}

private fun showFragment(targetFragment: Fragment) {  
    supportFragmentManager.beginTransaction()  
        .setReorderingAllowed(true)  
        .replace(R.id.container_main, targetFragment) //만들어진 객체 재사용
        .commitNow()  
}
```

원래라면, Fragment를 replace하면 이전에 등록된 Fragment는 onDestroy() 콜백을 타고 메모리에서 사라진다.  
하지만 Activity에서 DESTROY된 Fragment를 변수로 가지고 있기 때문에,  
해당 Fragment에 대한 참조를 가지고 있어서, 메모리에서 소멸될 수 없다.  
해당 Fragment로 다시 replace를 하면, onAttach()부터 정상적으로 다시 Fragemnt가 inflate된다.  
하지만, 이미 생성되었던 인스턴스이기 때문에, 멤버 변수 초기화는 일어나지 않는다.  

</br>

### 이게 왜 문제가 될까?

registerForActivityResult로 전달해준 ActivityResultCallback은 내부적으로  ActivityResultRegisty.register()를 호출하는데,     
해당 함수에서 ActivityResultCallback을 등록하는 Fragment의 생명주기에  obsever를 추가해둔다.  
observer는 Fragment가 ON_DESTROY event를 수신하는 순간, ActivityResultRegistry에서 ActivityResultCallback을 지워버린다.  
![500](Dev/Res/img-20231106-3.png)

Fragment의 onDestroy()콜백을 호출해주는 Fragment.performDestroy()함수 내부에서   
mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY); 코드를 통해,  
Fragment의 생명 주기가 바뀐다는 Lifecycle.Event를 옵저버들에게 알린다.  
ON_DESTROY Event를 넘겨 받은 옵저버는  
미리 정의해둔 내용대로 unregister를 호출해서 ActivityResultRegistry에서 ActivityResultCallback을 지워버린다.  
![500](Dev/Res/img-20231106-2.png)

## **그래서, 아래와 같이 등록되지 않은  등록되지 않은 launcher를 사용했다고 런타임 예외가 터지던 것이었던 것이었다.**


>java.lang.IllegalStateException: 
	Attempting to launch an unregistered ActivityResultLauncher 
	with contract androidx.activity.result.contract.ActivityResultContracts$StartActivityForResult@466af5a 
	and input Intent { cmp=com.kova700.zerotomvvm/.DetailActivity (has extras) }. 
	You must ensure the ActivityResultLauncher is registered before calling launch().


Activity를 변수에 담아서 onDestroy되어도 사용할 일이 거의 없기 때문에  
Activity는 멤버 변수에서 registerForActivityResult()를 호출해도 문제가 되지 않을 것다.  
하지만 Fragment의 경우에, 매번 새로운 Fragment를 replace에 사용하거나,  
show & hide방식으로 사용하면 위 문제가 생기지 않으나,  
변수에 참조된 Fragment를 다시 replace에 사용되는 경우에는  
1회에 한해서만 등록되기 때문에 Fragment의 생명 주기 안에서 매번 등록을 해주는게 좋아보인다.  
( 이와 관련한 내용의 이슈 트래커 : https://issuetracker.google.com/issues/247221861  )  

</br>

### 그래서 Activity는 언제 호출하면 되는데?
***
위에서 말했듯,  Activity에서는 멤버 변수에서 registerForActivityResult()를 호출해서 사용하면 될듯하다.

</br>

## Reference
***
https://developer.android.com/training/basics/intents/result  
https://developer.android.com/guide/fragments/lifecycle  
https://developer.android.com/guide/components/activities/activity-lifecycle  
https://developer.android.com/training/basics/intents/result  
https://issuetracker.google.com/issues/247221861  
https://pluu.github.io/blog/android/2023/01/19/fragment_visible_lifecycleowner/  
https://medium.com/jaesung-dev/%EB%86%93%EC%B9%98%EA%B8%B0-%EC%89%AC%EC%9A%B4-lifecycle-daf5b293f5e  
https://developer.android.com/reference/androidx/lifecycle/Lifecycle   
https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State  
https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event    