MVC패턴이 기존에 View와 Controller가 통합된 채로  
Controller(View) 와 Model 체제로 구성된다고 했다.  

View와 Controller가 결합된 채로 코드를 구성하게 되면,  
코드의 양이 작을 때는 크게 불편함을 느끼지 못하고,  
하나의 클래스 안에 모든 내용이 담겨있어서 오히려 가독성이 높다고 생각된다.  

하지만, View에 새로운 데이터로 UI를 갱신하는 코드,  
View의 visibility를 설정하는 코드,  
Model로부터 데이터를 가져오는  코드,  
Model에게 갱신 요청하는 코드,  
다른 Activity를 호출하는 코드 등  
코드의 양이 많아지면 하나의 Activity(Fragment)에 너무 많은 코드가 담겨서   
**가독성이 떨어지고, 코드의 수정이 어려워진다.** (전부 엉켜있는 느낌)  
위 상황에서 RecyclerView의 Adapter가 여러 개이거나, Repository가 여러 개인 경우,  
상황은 더욱 심화된다.  

그래서 이 상황에 MVP 패턴을 사용하면 위 상황을 어느 정도 해결할 수 있다.     
**MVP에서는 MVC에서 View와 Controller가 결합된 상태를 완전히 분리 시킨다.**   
View와 Controller역할을 동시에 했던 **Activity(Fragment)를 오로지 View의 역할로써만 사용**한다.  
**View와 Model만 남기고, View에서 필요한 이벤트 처리(onClick과 같은)는 
Presenter를 통해서 처리하고, Model과 상호 작용 또한, Presenter를 통해서 작동하는 구조다.**    
그래서 View에서는 Model을 직접적으로 사용할 일이 없기 때문에, View와 Model의 결합도가 줄어드는 장점이 있다.  

MVP 구현시 참고하면 매우 좋은 글  (MVVM 시에도 좋은 글)  
https://medium.com/@cervonefrancesco/model-view-presenter-android-guidelines-94970b430ddf  

Presenter에서 데이터와 상태는 가지고 있지 않는 것이 좋음
어차피 Activity의 생명주기를 따르기 때문에, Configuration change 혹은 
Fragment의 경우 System으로 부터 memory kill을 당해서 다시 생성되는 경우 
Presenter는 데이터와 상태를 보존하지 못함으로 Repository같은 Application의 생명주기를 가지는 싱글톤 클래스에 데이터를 가지고 있는 게 좋다고 함

### Reference
***
https://thdev.tech/androiddev/2016/06/14/Android-TODO-MVP-Example/#google_vignette  
https://thdev.tech/androiddev/2017/02/18/Android-MVP-Presentation/  
https://thdev.tech/androiddev/2016/10/12/Android-MVP-Intro/  
https://thdev.tech/androiddev/2016/10/23/Android-MVC-Architecture/  