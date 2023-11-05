 ConfigurationChange가 발생하면 
 안드로이드 시스템이 Fragment의 새로운 인스턴스를 생성하는데도, 
  어떻게 상태를 저장하고, 그 상태를 어떻게 넘겨받는지 과정을 설명해보자.

FragmentManager.saveAllState()에
mStateSaved = true; 부분을 보자

Fragment.performSaveInstanceState()에서
mChildFragmentManager.saveAllState();를 호출한 값의 결과를
Parcelable로 받는다. (State가 저장된 내용)
그 내용을 outState라는 Bundle에 집어 넣는다.

FragmentStateManager.saveBasicState()가
Fragment.performSaveInstanceState()를 호출해서
멤버 fragment들의 state를 result라는 Bundle에 담아 온다.

FragmentStateManager.saveInstanceState()에서
FragmentStateManager.saveBasicState()가 반환한 Bundle을
Fragment.SavedState객체에 담아서 반환

FragmentManager.saveFragmentInstanceState()가
FragmentStateManager.saveInstanceState()가 반환한 
Fragment.SavedState객체를 다시 반환
이 함수에 설명에 새로운 Fragment 인스턴스에 
타입만 같다면 해당 State를 적용할 수 있다고 설명이 적혀있음

Fragment.setInitialSavedState()에서
FragmentManager.saveFragmentInstanceState()에서 저장한
상태를 불러와서 복원한다고 설명이 적혀있음

FragmentActivity.init()에서
Parcelable p = mFragments.saveAllState();
p를 getSavedStateRegistry().registerSavedStateProvider(FRAGMENTS_TAG,...
에 등록함

그리고 
Bundle savedInstanceState = getSavedStateRegistry()
					.consumeRestoredStateForKey(FRAGMENTS_TAG);

			if (savedInstanceState != null) {
				Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
				mFragments.restoreSaveState(p);//여기
			} 
로 새로운 Fragment들에게 이전에 저장했던 viewstate를 적용시킴

Activity의 configuration Change가 발생하면
Activity의 onSaveInstanceState를 정의하지 않더라도 superType에서 
onSaveInstanceState에 내용을 저장한다 위에서 저장한 내용과 함께,
그리고 Activity는 넘겨받은  savedInstanceState를 superType에서
뜯어 본 다음 상태를 복구한다

그 증거로 Activity의 onRestoreInstanceState를 아래와 같이 빈 Bundle로 구성하면 Configuration Change가 발생하면 스크롤 유지가 안된다.
그냥 RecyclerView마저 사라짐
override fun onRestoreInstanceState(savedInstanceState: Bundle) {  
    super.onRestoreInstanceState(bundleOf())  
})

그 내용은
FragmentStateManager.restoreState()에서
저장된 상태가 있는 Activity객체에 접근해서 Fragment에게 mSavedViewState를 넘겨준다.