# Fragment Lifecycle

프래그먼트는 FragmentManager를 통해 관리되며, 앱이 실행되는 동안 여러 수명 주기 상태를 거친다. 프래그먼트의 수명 주기를 이해하고 적절하게 관리하면 앱의 안정성과 사용자 경험을 향상시킬 수 있다.

![Untitled](/images/fragment-view-lifecycle.png)

## 주요 수명 주기 상태

1. INITIALIZED
    - 프래그먼트가 인스턴스화된 초기 상태
2. CREATED
    - 프래그먼트가 생성되고, onCreate() 콜백이 호출된다.
    - 프래그먼트의 상태를 복원할 수 있다.
3. STARTED
    - 프래그먼트가 화면에 보이기 시작하고, onStart() 콜백이 호출된다.
4. RESUMED
    - 프래그먼트가 포그라운드에 나타나고, 사용자와 상호작용할 준비가 되었음을 나타낸다.
    - onResume() 콜백이 호출된다.
5. PAUSED
    - 사용자가 프래그먼트를 벗어나기 시작할 때 호출되며, onPause() 콜백이 호출된다.
6. STOPPED
    - 프래그먼트가 더 이상 화면에 보이지 않을 때 호출되며, onStop() 콜백이 호출된다
7. DESTROYED
    - 프래그먼트가 소멸되기 직전에 호출되며, onDestroy() 콜백이 호출된다.

## 주요 콜백 메서드

1. onAttach()
    - 프래그먼트가 호스트 액티비티에 연결될 때 호출된다.
    - 이 시점에 프래그먼트가 활성화되고, FragmentManager는 프래그먼트의 생명주기를 관리한다.
2. onCreate()
    - 프래그먼트가 생성될 때 호출되며, 초기화 작업을 수행한다.
    - 이 시점에는 아직 Fragment View가 생성되지 않았기 때문에 Fragment의 View와 관련된 작업을 수행하지 않도록 해야한다.
3. onCreateView()
    - 프래그먼트의 뷰가 생성될 때 호출된다. 즉, Layout을 Inflate하는 작업을 수행하는 부분이다.
4. onViewCreated()
    - 뷰가 생성된 직후에 호출되며, 뷰 초기화 작업을 수행한다.
    - View의 초기값을 설정해주거나 LiveData Observing, RecyclerView나 ViewPager2에 사용할 Adapter 초기화 등을 이곳에서 수행한다.
5. onStart()
    - 프래그먼트가 시작될 때 호출되며, 프래그먼트가 화면에 나타난다.
    - 이 시점부터는 Fragment의 child FragmentManager 통해 FragmentTransaction 을 안전하게 수행할 수 있다.
6. onResume()
    - 프래그먼트가 사용자와 상호작용할 준비가 되었을 때 호출된다.
7. onPause()
    - 프래그먼트가 일시 중지될 때 호출된다.
8. onStop()
    - 프래그먼트가 중지욀 때 호출된다.
9. onDestroyView()
    - 프래그먼트의 뷰가 소멸될 때 호출된다.
    - 가비지 컬렉터에 의해 수거될 수 있도록 Fragment View에 대한 모든 참조가 제거되어야 한다.
10. onDestroy()
    - 프래그먼트가 소멸될  때 호출된다.

## 수명 주기 상태 전환

- 상향 상태 전환
    - 프래그먼트가 초기화되고 생성된 후 시작되고 재개되는 상태로 전환된다.
    - onAttach() -> onCreate() -> onCreateView() -> onViewCreated() -> onStart() -> onResume()
- 하향 상태 전환
    - 프래그먼트가 일시 중지되고 중지되며 소멸되는 상태로 전환된다.
    - onPause() -> onStop() -> onDestroyView() -> onDestroy() -> onDetach()

## 프래그먼트의 뷰 수명 주기

- 프래그먼트의 뷰는 프래그먼트와 독립적으로 관리되는 별도의 수명 주기를 가진다.
- getViewLifecycleOwner()를 통해 뷰의 수명 주기에 접근할 수 있다.
- 뷰의 수명 주기는 프래그먼트의 뷰가 존재하는 동안에만 유효하다.
