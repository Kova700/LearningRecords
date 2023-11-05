## Q1. Activity의 Configuration change가 발생하면, Activity 내부에 사용되던 Fragment는 어떻게 되는가? (해결)

https://developer.android.com/reference/android/app/Fragment.html
에 의하면 Activity가 중지되면 Activity 내부의 어떤 Fragment도 시작할 수 없고, 모든 Fragment가 파괴된다 나와있음

새로운 Activity가 생성된다.
	그럼 이전 Activity의 FragmentManager가 가지고 있던 Fragment들은 어떻게 되는가?
	전부 Destroy된다.
	repalce를 이용해서 Fragment전환을 한다면 FragmentManager는 1개의 Fragment만 가지고있고
	나머지 Fragment는 전부 remove되기 때문에 Configuration Change가 발생하면
	띄워져있던 1개의 Fragment도 Destroy한다.
	(파괴되는 순서는 Activity Destroy -> Fragment Destroy 순이다)

하지만 FragmentManager는 ConfigurationChange 발생 시 Fragment에 속한 ViewState를 기억하고 있다가
새로운 Frgment를 만들어서 해당 Fragment의 상태를 주입시킨다. (스크롤 위치)

그래서 findFragmentByTag로 가져온 Fragment와 처음 사용했던 Fragment의 System.identityHashCode를 
찍어보면 다르게 나온다는걸로 증명할 수 있다.

FragmentActivity내부에 onDestroy()
```java
@Override  
protected void onDestroy() {  
    super.onDestroy();  
    mFragments.dispatchDestroy();  //Fragment전부 Distory호출
	mFragmentLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY);  
}
```

## Q2. Fragment를 replace해서 onDestroy()된다면 Fragment 객체는 소멸되는가? (해결)

Activity는 configuration Change가 발생하면 새로운 Activity객체를 생성하는 것은
hashcode를 찍어보고 확인했음 (그래서 Activity 멤버변수들도 다시 초기화 됨)

replace해서 onDestroy()된다면 Fragment 객체는 소멸되지만 참조를 가지고 있는 변수가 있다면 소멸되지 않고,
다시 해당 객체로 replace요청을 하면 처음부터 다시 Attach()콜백이 시작된다.
	

## Q3. Fragment를 replace하면 어떻게 스크롤 위치를 그대로 유지할 수 있는건가?

새로운 Frgment를 만들어서 해당 Fragment의 상태를 주입시킨다는  것은 알겠는데,
상태는 어떤게 포함되어있는가? 스크롤 위치만 가지고 있나?, 
데이터가 없다면 스크롤 위치도 의미가 없지 않나
만약 데이터를 로드할 수 없는 상황이라면 스크롤 위치도 유지되지 않겠네?

stateRestorationPolicy = RecyclerView.Adapter.StateRestorationPolicy.PREVENT를 설정하면 
RecyclerView의 스크롤이 유지되지 않음
기본 값은 StateRestorationPolicy.ALLOW임으로 스크롤 값이 유지됨
RecyclerView의 LayoutManager는 configurationChage가 발생하면 내부 구성값을,  LayoutManager객체에 주입 해줌
(RecyclerView 내부에 mPendingSavedState의 변수에 담아서 그걸 LayoutManger에 SaveSate로 담음)

Fragment의 LayoutManger는 Fragment가 종료되기 전에 RecyclerView의 LayoutManager의 정보를 저장함
Fragment가 onDestroy()되어도 변수로 참조를 가지고 있으면 메모리에서 사라지지 않음,
고로, 해당 Fragment로 replace요청시에 저장된 값을 가지고 있기 때문에 스크롤을 유지할 수 있음

다시 어댑터와 리사이클러뷰가 연결될 때, 저장된 State를 불러와서 스크롤을 유지하는 내용
RecyclerView 내부에 onRestoreInstanceState 와 onSaveInstanceState의 내용을 보고
StateRestorationPolicy.PREVENT에 연결된 코드와 RecyclerView에 setLayoutMager 코드를 보면 위 내용을 확인할 수 있다.
Fragment는 RecyclerView를 포함한 자식뷰에 저장된 내용을 함께 mSavedViewState에 저장한다.
(void moveToState(@NonNull Fragment f, int newState)함수를 따라가보자)
Fragment의 performActivityCreated 함수가 restoreViewState함수를 통해  Fragment 객체 내부에 저장된 state를
Fragment 내부의 View들에게 적용한다.

Fragment.restoreViewState() 내용 중

![[img-20231105.png]]
FragmentManager.moveToState() 내용 중 
![[img-20231105-1.png]]

![[img-20231105-2.png]]

참고 : https://medium.com/androiddevelopers/restore-recyclerview-scroll-position-a8fbdc9a9334
참고 : https://blog.seoft.co.kr/68

## Q4. Fragment replace는 내부적으로 FragmentManger에서 이전 Fragment를 remove하는건가? (해결)
ㅇㅇ 전부 remove되고 add로 새로운 Fragment추가함

addtoBackStack이 없다면
onDestroyView() - onDestroy() - onDetach()까지 완전히 제거하고 
ddtoBackStack이 있다면
nPause() - onStop() - onDestroyView()까지만 하고 대기하고 있는다.

그렇다면 이전 Fragemnt인스턴스는 사라진다는건데 
이전 Fragment인스턴스를 전달해서 replace요청을하면 어떻게 되는건가? 
	Destroy된 Fragment 인스턴스를 다시 repalce요청을 하면 
	Destroy된 Fragment다시 Attach부터 Reusme()까지 생명주기 콜백을 태워서 재활용이 가능하다.
	Destroy되어도 해당 Fragment 인스턴스의 참조를 변수가 가지고 있어서 GC가 수거해가지 않는 것같음


## Q5. Fragment.getInstance() 자체적으로 싱글톤 구현해서 사용하는 것과 findFragmentByTag로 FragmentManager가 만든 Fragment를 가져와서 사용하는 것 어느 것이 더 좋은 방법인가.  (해결)
딱히 argument를 넘겨줄 일이 없으면 굳이 하지 않아도 될 듯하다.

## Q6. 새로운 Fragment 인스턴스로 replace하면 왜 스크롤 위치가 유지되지 않는건가
그냥 완전히 새로운 Fragment를 inflate해서 그런건가? 
	-->Fragment 내부에 저장된 ViewState가 없어서

