---
layout: default
title: Paging3를 사용하여 페이징 기능 구현(쌩기초&자세히)
parent: Android(코틀린)
---

# 서론
Paging3를 사용하여 페이징 기능을 사용하는 방법에 대해 예시로 알아본다
Paging3를 사용했을 때 어떠한 장점이 있고 어떤 단점이 있는지 알아본다

해당 프로젝트에선 Hilt를 사용했습니다.
Hilt에 대한 코드는 생략하겠습니다.
그 외에 관계있는 코드는 가능한한 올려보겠습니다!

* 해당 프로젝트는 코루틴 flow를 사용하여 비동기 처리를 했습니다.

# 개요
### Paging이란?
리스트를 스크롤할 때 데이터가 많을 경우 한 번에 모든 데이터를 로드 하지 않고
일정 양을 스크롤 했을 때 다음 데이터를 추가적으로 로드하는 기능이다.

### 왜 Paging3 라이브러리를 사용하는게 좋을까?
일반적으로 Paging 기능을 직접 구현해서 사용할 때 다음과 같은 문제점이 많이 발견된다
1. 중복으로 데이터를 취득할 가능성이 있다
2. 데이터를 로드하고 보여주는 과정에서 생기는 각종 문제(메모리 릭 등)

### Paging3는 다음과 같은 장점을 갖고 있기 때문에 편리합니다.
* 페이징 된 데이터의 메모리 내 캐싱. 이렇게 하면 앱이 페이징 데이터로 작업하는 동안 시스템 리소스를 효율적으로 사용할 수 있습니다.
* 요청 중복 제거 기능이 기본으로 제공되어 앱에서 네트워크 대역폭과 시스템 리소스를 효율적으로 사용할 수 있습니다.
* 사용자가 로드된 데이터의 끝까지 스크롤할 때 구성 가능한 [RecyclerView](https://developer.android.com/reference/kotlin/androidx/recyclerview/widget/RecyclerView?hl=ko) 어댑터가 자동으로 데이터를 요청합니다.
* Kotlin 코루틴 및 Flow뿐만 아니라 [LiveData](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData?hl=ko) 및 RxJava를 최고 수준으로 지원합니다.
* 새로고침 및 재시도 기능을 포함하여 오류 처리를 기본으로 지원합니다.

### Paging3는 Groupie 라이브러리와 같이 사용할 수 **없다**
* 현재 Groupie 라이브러리에서 paing 라이브러리를 지원하지 않음(2022.12.01)

------

# Paging3의 구성
Paging3는 다음과 같이 3개의 Layer로 구성되어 있다

<img width="805" alt="image" src="https://user-images.githubusercontent.com/69494230/204439372-c8993108-8495-4585-ae17-5a91786da0ca.png">

### 1. Repository Layer
해당 레이어는 [PagingSource](https://developer.android.com/reference/kotlin/androidx/paging/PagingSource?hl=ko)와 [RemoteMediator](https://developer.android.com/reference/kotlin/androidx/paging/RemoteMediator?hl=ko)로 이루어져 있으며 데이터를 검색 / 로드하는데 사용된다

### 2. ViewModel Layer
Repository Layer에서 구성된 객체를 바탕으로 [PagingData](https://developer.android.com/reference/kotlin/androidx/paging/PagingData?hl=ko)인스턴스를 구성하기 위한 API를 제공한다
PagingData는 해당 레이어를 UI와 연결하는 역할도 한다

### 3. UI Layer
 UI Layer의 기본 페이징 라이브러리는 RecyclerView의 어댑터인 [PagingDataAdapter](https://developer.android.com/reference/kotlin/androidx/paging/PagingDataAdapter?hl=ko)이다.
또는 포함된 [AsyncPagingDataDiffer](https://developer.android.com/reference/kotlin/androidx/paging/AsyncPagingDataDiffer?hl=ko) 구성요소를 사용하여 고유한 맞춤 어댑터를 빌드할 수 있다.

----

# 필요한 Dependency

``` kotlin
dependencies {
  def paging_version = "3.1.1"

  implementation "androidx.paging:paging-runtime-ktx:$paging_version"

  // alternatively - without Android dependencies for tests
  testImplementation "androidx.paging:paging-common:$paging_version"

  // optional - RxJava2 support
  implementation "androidx.paging:paging-rxjava2:$paging_version"

  // optional - RxJava3 support
  implementation "androidx.paging:paging-rxjava3:$paging_version"

  // optional - Guava ListenableFuture support
  implementation "androidx.paging:paging-guava:$paging_version"

  // optional - Jetpack Compose integration
  implementation "androidx.paging:paging-compose:1.0.0-alpha17"
}
```
# Paging3의 PaginSource 정의
PagingSource<Key, Value>에는 Key와 Value의 두 유형 매개변수가 있습니다. 
키는 데이터를 로드하는 데 사용되는 식별자를 정의하며, 값은 데이터 자체의 유형입니다. 
예를 들어 Int 페이지 번호를 [Retrofit](https://square.github.io/retrofit/)에 전달하여 네트워크에서 User 객체의 페이지를 로드하는 경우
Key 유형으로 Int를, Value 유형으로 User를 선택합니다.

``` kotlin
class ExamplePagingSource @Inject constructor(
    private val backend: ExampleBackendService
): PagingSource<Int, User>() {
    /**
     * load()는 params을 바탕으로 페이지의 데이터를 반환하면된다
     * 페이징 할 때 마다 성공 시 이 부분이 계속 호출된다
     *
     * @param params 객체에는 실행할 로드 작업에 관한 정보가 포함됩니다. 여기에는 로드할 키와 로드할 항목 수가 포함됩니다. page의 key 타입을 정의한다
     * @return 데이터 로드에 성공했을 때는 LoadResult 반환
     * */
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, User> {
        return try {
            Timber.d("PagingSource load called")
            // key 값이 없을 때는 1로 초기화
            val nextPageNumber = params.key ?: 1
            val response = backend.searchUsers(nextPageNumber)
            // LoadResult.Page는 데이터 로드에 성공했을 때 반환되는 값이다
            LoadResult.Page(
                data = response.users,
                prevKey = null, // null로 설정하면 반드시 다음 데이터만 로드한다
                nextKey = response.nextPageNumber
            )
        } catch (e: Exception) {
            // 데이터 로드에 실패했을 경우에는 LoadResult.Error를 반환하게 한다
            LoadResult.Error(e)
        }
    }

    /**
     * refresh 시 다시 시작할 key를 반환해주면 된다
     *
     * @param state 로드된 페이지 및 마지막으로 엑세스한 위치 등의 상태를 갖고 있음
     * @return 여기선 key가 Int이기 때문에 Int를 반환하면된다
     * */
    override fun getRefreshKey(state: PagingState<Int, User>): Int? {
        // Try to find the page key of the closest page to anchorPosition, from
        // either the prevKey or the nextKey, but you need to handle nullability
        // here:
        //  * prevKey == null -> anchorPage가 현재 첫 번째 페이지임
        //  * nextKey == null -> anchorPage가 현재 마지막 페이지임
        //  * both prevKey and nextKey null -> anchorPage가 최초의 상태이기 때문에 그냥 null을 반환한다
        Timber.d("refresh is called")
        return state.anchorPosition?.let { anchorPosition ->
            val anchorPage = state.closestPageToPosition(anchorPosition)
            anchorPage?.prevKey?.plus(1) ?: anchorPage?.nextKey?.minus(1)
        }
    }
}
```

# ExampleBackendService와 기타 다른 클래스 정의
### ExampleBackendService

``` kotlin
/**
 * Paging시 실시할 동작을 지정한다
 * */
class ExampleBackendService {
    /**
     * User 데이터를 Page에 넣어서 반환
     *
     * @return 로드한 결과(데이터)
     * */
    fun searchUsers(page: Int): Page {
        val users = mutableListOf<User>()
        // 가져온 페이지를 기록하게 한다
        users.add(User("name$page", page))
        return Page(users, page + 1)
    }
}
```
### User
* User라는 데이터를 받아와서 출력하게 한다

```kotlin
data class User(
    val name: String,
    val age: Int
)
```

### Page

``` kotlin
data class Page(
    val users: List<User>,
    val nextPageNumber: Int
)
```
# PaingRepository
* 데이터를 로드하고 받아올 Repository를 정의한다

``` kotlin
/**
 * Paging 처리에 대한 Repository
 *
 * @param service 데이터를 받아오기 위해 필요한 객체
 * */
class PagingRepository @Inject constructor(
    private val service: ExampleBackendService
) {
    /**
     * 페이징한 결과(데이터)를 받아온다
     * */
    fun getPagingData(): Flow<PagingData<User>>{
        // flow뿐 아니라 Observable, LiveData도 가능함
        // pageSize: RecyclerView에서 항상 갖고 있을 마지막 아이템 수를 의미한다
        // 예를 들어 현재 RecyclerView가 갖고 있는 item이 20개일 경우 이후 스크롤 할 때 마다 1개씩 늘어난다
        // 즉, 이미 스크롤 해서 지나간 item은 메모리에 갖고 있고 이후 계속 로드 하는 방식임
        // 공식 권장은 10~20
        return Pager(PagingConfig(pageSize = 20)){
            // 이 부분은 한 번만 호출된다
            Timber.d("getPagingData in Repository is called")
            ExamplePagingSource(service)
        }.flow
    }
}
```
# PagingViewModel
* UI와 Repository를 연결해줄 viewModel을 정의한다
* cachedIn()은 기본적으로 mainThread를 사용하지만 다른 Thread를 사용하도록 정의할 수 도 있다

``` kotlin
@HiltViewModel
class PagingViewModel @Inject constructor(
    repository: PagingRepository
) : ViewModel() {
    // 리포지토리에서 데이터를 가져온다
    // cachedIn은 연산자는 데이터 스트림을 공유 가능하게 만들고, 제공된 CoroutineScope를 사용하여 로드 된 데이터를 캐시한다.
    // 기본값으로 mainThread를 사용하여 데이터 스트림을 공유하게 한다
    val pagingData = repository.getPagingData().cachedIn(viewModelScope)

    // mainThread가 아닌 다른 방법으로 하고 싶다면 아래와 같은 방법으로도 가능하다
//    val pagingData = repository.getPagingData().cachedIn(CoroutineScope(Dispatchers.IO))
}
```

# MainActivity의 onCreate() 내부 정의

``` kotlin
// RecyclerView 정의
val userAdapter = UserAdapter()
binding.recyclerView.run {
    this.adapter = userAdapter
    layoutManager = LinearLayoutManager(requireContext())
}

lifecycleScope.launch {
    viewModel.pagingData.collectLatest {
        // 반드시 submitData로 데이터를 보내줘야 한다(submitData는 Paging 라이브러리의 일부임)
        (binding.recyclerView.adapter as UserAdapter).submitData(it)
    }
}
```

# UserAdapter
* Paging3를 사용하기 위해선 아래의 두 가지를 지켜야한다
1. RecyclerView의 adapter를 직접 정의해야함 = Groupie 사용불가
2. RecyclerView의 adapter에서 DiffUtil를 사용해야함

``` kotlin
/**
 * UserAdapter 정의
 *  Paing3 라이브러리는 Groupie로 구현할 수 없다
 * */
class UserAdapter: PagingDataAdapter<User, PagingViewHolder>(diffCallback){
    companion object{
        private val diffCallback = object : DiffUtil.ItemCallback<User>() {
            override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
                return oldItem == newItem
            }

            override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
                return oldItem == newItem
            }

        }
    }
    override fun onBindViewHolder(holder: PagingViewHolder, position: Int) {
        val item = getItem(position)
        if (item != null){
            holder.bind(item)
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PagingViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)
        return PagingViewHolder(ItemUserBinding.inflate(layoutInflater, parent, false))
    }
}

class PagingViewHolder(
    private val binding: ItemUserBinding
): RecyclerView.ViewHolder(binding.root){
    fun bind(user: User){
        binding.name.text = user.name
        binding.age.text = user.age.toString()
    }
}
```
------
# 실행 확인

<img src="https://user-images.githubusercontent.com/69494230/205206807-17728e69-9c3e-43bb-98b0-99b6c14e482c.gif" width="300">


-----
# 후기
Paging3가 참 좋긴한데 Groupie를 쓸 수 없다는 점이 좀 크네요 ㅠㅠ
Groupie를 포기하고 이걸 쓰기에는 Groupie를 쓰는 편이 더 편하기 때문에 지원하기 전까지는 안쓸거 같습니다...(아쉽)

-----
# 참조한 페이지
<a href="https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko" target="_blank">공식 블로그</a>

<a href="https://leveloper.tistory.com/202" target="_blank">leveloper 블로그</a>