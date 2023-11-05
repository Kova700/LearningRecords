이번 글에서는 registerForActivityResult()로 인해 발생하는 두 가지 오류에 대한 내용과   
해당 오류를 발생하지 않게 하려면 언제 registerForActivityResult를 호출해야 하는 가에 관한 글을 작성해 보려 한다.  

registerForActivityResult를 Activity 혹은 Fragment에서 사용하다 보면  
#### Activity
> IllegalStateException : LifecycleOwner is attempting to register while current state is STARTED. LifecycleOwners must call register before they are STARTED.
#### Fragment
> IllegalStateException :Fragments must call registerForActivityResult() before they are created (i.e. initialization, onAttach(), or onCreate()).

와 같이  registerForActivityResult()를 특정 시점 이전에 호출해야 한다고 경고하며 런타임 예외를 던지는 경우와  

> IllegalStateException : Attempting to launch an unregistered ActivityResultLauncher 

와 같이 등록되지 않은 ActivityResultLauncher를 사용하고 있다는 예외를 만날 수 있다.  

위 오류들은 왜 발생하고, 어떻게 해야지 방지할 수 있을 지에 관한 내용을 알아보자.  

이벤트와 status 관한 내용 참고
https://medium.com/jaesung-dev/%EB%86%93%EC%B9%98%EA%B8%B0-%EC%89%AC%EC%9A%B4-lifecycle-daf5b293f5e  


</br>

***

</br>

우선, 안드로이드 [공식문서](https://developer.android.com/training/basics/intents/result)에 따르면  
> **Note:** You must call `registerForActivityResult()` before the fragment or activity is created,  
> but you can't launch the `ActivityResultLauncher` until the fragment or activity's [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) has reached [`CREATED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#CREATED).  

번역 : fragment 또는 activity가 생성되기 전에 registerForActivityResult()를 호출해야 하지만,   
fragment 혹은 activity의 생명 주기가 CREATED에 도달할 때까지 ActivityResultLauncher를 시작할 수 없습니다.  

"before the fragment or activity is created"에서  
fragment와 activity가 create되기 전이라는게 정확히 어떤 시점을 말하는 걸까?  

</br>

## Fragment 먼저 알아보자.
***

</br>

fragment에서 registerForActivityResult를 사용하기 위해서는  
fragment가 create되기 전에, registerForActivityResult를 호출해야한다.  

fragment의 생명주기는 아래의 그림과 같은데,  
fragment가 create되기 전은 어떤 시점을 말하는 걸까.  
FragmentLifcycle의 CREATED와 ViewLifecycle의 CREATED중 어느 것을 따라야 할지도 모르겠다.  

![[img-20231104.png|500]]

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
Fragment가 create되기 전인 onAttach(), or onCreate())에서하라고 경고해준다.**  
![[img-20231105-3.png]]

![[img-20231105-4.png]]

![[img-20231105-5.png]]
![[img-20231105-6.png]]


그러니까 **Fragment가 create되기 전**이라고 한다면  
위에 Fragment lifeCycle 이미지에 나왔던 **Fragment의 ViewLifeCycle이 CREATED 상태가 되기 전**의 상태를 의미한다고 볼 수 있다.  
**고로 onViewStateRestored( )이전에 registerForActivityResult( )를 호출하라는 결론이 나온다.**  

</br>

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
Fragment 인스턴스가 생성될 때, 멤버 변수도 초기화 됨으로  최초 1회에 한해서만 ActivityResultLauncher가 초기화되는데(register 됨),  
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
(멤버 변수 초기화부분은 일어나지 않는다.)

Fragment가 onDestroy()콜백을 타면,
Fragment.performDestroy()함수 내부에,
mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY); 코드를 통해,
Fragment의 생명주기가 바뀐다는 Event를 옵저버들에게 알린다.
Fragment의 생명주기가 ON_DESTROY되었다는 내용을 넘겨 받은 옵저버는
미리 정의해둔 내용대로 unregister를 호출해서 ActivityResultRegistry에서 ActivityResultCallback을 지워버린다.

ActivityResultRegisty.register()에 의해 등록되고,
ActivityResultRegisty.unregister()에 의해 제거된다.




FragmentManager 내부적으로 캐싱해서 사용함으로
onDestroy가 되어도 객체가 사라진게 아니다. 같은 객체로 다시 onCreate를 함으로
unregist되었다고 나오는 것

Fragment를 replace 방식으로 전환하는 구조를 가지게 되면
Fragment의 lifecycle은 아래와 같이 1번 Fragment를 지우고 2번 Fragment를 사용한다.
> 1번 Fragment add 혹은 repalce
> 1 - onAttach() 
> 1 -  onCreate()
> 1 -  onCreateView 
> 1 - onViewCreated() 
> 1 - onViewStateRestored 
> 1 - onStart() 
> 1 - onResume()
> 2번 Fragment replace
> 1 - onPause()
> 1 - onStop()
> 2 - onAttach() 
> 2 -  onCreate()
> 2 -  onCreateView 
> 2 - onViewCreated() 
> 2 - onViewStateRestored 
> 2 - onStart() 
> 1 - onDestroyView
> 1 - onDestroy
> 1 - onDetach
> 2 - onResume()




>java.lang.IllegalStateException: 
	Attempting to launch an unregistered ActivityResultLauncher 
	with contract androidx.activity.result.contract.ActivityResultContracts$StartActivityForResult@466af5a 
	and input Intent { cmp=com.kova700.zerotomvvm/.DetailActivity (has extras) }. 
	You must ensure the ActivityResultLauncher is registered before calling launch().

replac시에 onDestroyView까지 가게 된다.
여기서 unregister가 되어서 등록되지 않은 launcher를 썻다고 에러가 나온다
그래서 매번 등록을 해주는게 좋다.


registerForActivityResult는 어떤 것인지 알려주고
Fragment에서 registerForActivityResult를 호출하면 상위 Activity의 register에 등록된다는 내용도 작성

### 그래서 Activity는 언제 호출하면 되는데?
***


## Reference
***
https://developer.android.com/training/basics/intents/result  
https://developer.android.com/guide/fragments/lifecycle  
https://developer.android.com/guide/components/activities/activity-lifecycle  
https://developer.android.com/training/basics/intents/result  
https://issuetracker.google.com/issues/247221861  
https://pluu.github.io/blog/android/2023/01/19/fragment_visible_lifecycleowner/  
https://medium.com/jaesung-dev/%EB%86%93%EC%B9%98%EA%B8%B0-%EC%89%AC%EC%9A%B4-lifecycle-daf5b293f5e  