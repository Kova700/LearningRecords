Compose가 composable함수를 실행할 때 빌드한 UI에 대한 설명을 **컴포지션**이라고 합니다.  
  
상태 변경이 발생하면,  Compose는 영향을 받은 composable 함수를 새 상태와 함께 다시 실행하여 업데이트된 UI를 만듭니다.  
새로운 상태를 가지고 업데이트된 UI를 만드는것을 **리컴포지션**이라고 한다.  

If a state change happens, Compose re-executes the affected composable functions with the new state, creating an updated UI—this 
is called **_recomposition_**.  

또한 Compose는 개별 composable 함수에 필요한 데이터가 무엇인지 확인하여 
데이터가 변경된 구성요소만 재구성하고 영향을 받지 않은 구성요소는 건너뜁니다.  

Compose는 업데이트를 수신할 때 리컴포지션을 예약할 수 있도록 추적할 상태를 알아야 합니다.
Compose에는 특정 상태를 읽는 모든 컴포저블의 리컴포지션을 예약하는 특수 상태 추적 시스템이 있습니다.
이를 통해 Compose를 세분화하고 전체 UI가 아닌 변경이 필요한 composable 함수만 재구성할 수 있습니다. 

Compose의 State 및 MutableState 유형을 사용하여 Compose에서 상태를 관찰할 수 있도록 만듭니다.
Compose는 상태 값 속성을 읽고 값이 변경되면 리컴포지션을 트리거하는 각 컴포저블을 추적합니다. (그리고 각 컴포저블들을 리컴포지션)

하지만, 리컴포지션이 발생해도 값이 기본값으로 다시 초기화 되므로 상태가 변경되지 않는 현상을 마주하게된다.
그래서, 리컴포지션에서 기존에 변경된 상태 값을 보존할 수 있는 방법이 필요합니다.
Remember 내부에 정의된 상태 값은 초기 컴포지션 중에 컴포지션에 저장되며,
**저장된 값은 리컴포지션 전체에 걸쳐 유지됩니다.**★★★

일반적으로 Remember와 mutableStateOf는 composable 함수에서 함께 사용됩니다.

기존 앱에서
상태를 저장하기 위해 이미 LiveData, StateFlow, Flow 및 RxJava의 Observable과 같은 다른 관찰 가능 유형을 사용하고 있을 수 있습니다. Compose가 이 상태를 사용하고 상태가 변경될 때 자동으로 재구성하도록 허용하려면 상태를 State\<T>에 매핑해야 합니다.
아래와같이 State\<T>로 매핑할 수 있는 함수를 활용해보자.
```kotlin
// import androidx.lifecycle.viewmodel.compose.viewModel
@Composable
fun MyScreen(
    viewModel: MyViewModel = viewModel()
    //viewModel() = 컴포저블이 활동에서 사용되는 경우, 활동이 완료되거나 프로세스가 종료될 때까지 동일한 인스턴스를 반환
) {
    val dataExample = viewModel.exampleLiveData.observeAsState() //LiveData를 State타입으로 매핑

    // 이제 dataExample의 데이터가 바뀔 때마다,
    // MyScreen이 리컴포즈 된다.
    dataExample.value?.let {
        ShowData(dataExample)
    }
}
```
추가로 
viewModel()를 통해서 전달 받은
ViewModel 인스턴스를 다른 컴포저블에 전달해서는 안 되며, 필요한 데이터와 필수 로직을 수행하는 함수만 매개변수로 전달해야 합니다.
(메모리 누수가 발생할 수 있음)

### Reference
***
https://developer.android.com/codelabs/jetpack-compose-state#4    
https://developer.android.com/jetpack/compose/libraries#streams  
https://developer.android.com/jetpack/compose/libraries  