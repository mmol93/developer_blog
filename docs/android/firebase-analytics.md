---
layout: default
title: firebase의 analytics 사용해보기(기초)
parent: Android(코틀린)
---

firebase의 analytics는 안드로이드 앱을 개발하는 사람이라면 거의 100% 쓰는 기능이라고 할 수 있습니다.

사용자가 언제 앱을 사용했고, 어느 화면을 보고, 어떤 버튼을 클릭했는지

심지어는 언제 앱을 삭제했는지도 알려줍니다.

그렇기 때문에 firebase의 애널리틱스는 앱 개발을 할 때 필수로 들어가야하는 기능 중 하나입니다.

# 개요
Firebase Analytics를 셋업하고 사용해본다
Firebase 프로젝트 셋업 방법은 설명하지 않는다

------------------
# 애널리틱스 셋업 방법
1. 상단의 [Tool] -> [Firebase] -> 메뉴에서 [Analytics] 선택 -> Get started with Google Analytics[Kotlin]을 선택한다
2. 그 이후에는 설명서를 보고 따라하면 된다

-------------------
# firebase의 analytics에 필요한 dependency
`implementation 'com.google.firebase:firebase-analytics:21.2.0'`

------------------
# 코드 작성
MainActivity가 있고 그 안에 FirstFragment 라는 Fragment가 있는 레이아웃 구조의 앱임
* MainActivity
   * 화면을 열면 screen_view 이벤트를 출력하게 한다

``` kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var analytics: FirebaseAnalytics

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        Timber.plant(MyTimber())

        // Analytics 초기화
        analytics = Firebase.analytics

        // screen_view 이벤트 출력
        analytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW) {
            param(FirebaseAnalytics.Param.SCREEN_NAME, this@MainActivity.javaClass.simpleName)
            param("test", "test")
        }
}
```

* FirstFragment
   * 화면 안에 있는 버튼을 누르면 select_content 이벤트를 출력하게 한다

``` kotlin
class FirstFragment : Fragment() {
    private var binding: FragmentFirstBinding by autoCleared()
    private lateinit var analytics: FirebaseAnalytics

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentFirstBinding.inflate(inflater, container, false)

        // Analytics 초기화
        analytics = Firebase.analytics

        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // select_content 이벤트 출력
        binding.analyticsButton.setOnClickListener {
            analytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT) {
                param(FirebaseAnalytics.Param.SCREEN_NAME, this@FirstFragment.javaClass.simpleName)
                param("test", "test")
                param(FirebaseAnalytics.Param.ITEM_NAME, "test item name");
            }
        }
    }
}
```

* firebase의 analytics 테스트에서 사용한 화면 모습

<img src="https://user-images.githubusercontent.com/69494230/202603241-0d83af65-7d63-4031-814a-84502ae328d8.jpg" width="200">

--------------------
# 테스트 방법
* 테스트 방법으로는 DebugView를 사용한다
   * adb를 사용한 테스트도 있지만 DebugView가 직관적이고 확실하기 때문에 이 방법을 사용한다
   * DebugView 사용 방법에 대해서는 아래의 링크 참조
* 아래와 같이 이벤트가 출력되는 것을 확인할 수 있다

<img width="300" alt="image" src="https://user-images.githubusercontent.com/69494230/202603442-a28f7529-3dba-4f5c-b84b-96e45dc6bb12.png">
