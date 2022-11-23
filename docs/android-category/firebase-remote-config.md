---
layout: default
title: firebase의 remote config 사용해보기(기초)
parent: Android(코틀린)
---

앱을 사용하다보면 내가 업데이트도 안했는데 layout이 바뀌어 있고

보던 내용이 바뀌어있고 이런 것들은 어떻게 구현했을까?

생각하는 것들이 있습니다.

그럴 때 사용한 것이 remote config입니다.

---------------------
# 개요
remote config를 사용해서 데이터를 받아서 출력해본다

-------------------------
# remote config 셋팅

## firebase 콘솔 셋업
1. firebase의 오른쪽에 있는 remote config를 클릭한다
2. 처음 클릭했다면 아래와 같은 그림이 나온다

<img src="https://user-images.githubusercontent.com/69494230/202616974-8bd7023e-a53e-4195-8958-25c694c6e4d4.png" width="300">

3. 여기서 다음과 같은 값을 설정해야한다
   * key: 앱 쪽에서 데이터를 받을 때 지정할 key
   * Data type: 어떤 형태의 데이터를 넘길지(이거도 앱 쪽에서 지정해야할 필요가 있음)
   * default value: 어떤 데이터를 넘겨줄지 적는다

4. 테스트로 2개 정도 적어서 넣어봤다

<img src="https://user-images.githubusercontent.com/69494230/202617329-fb47dcfc-538d-4317-811d-12fba4c0dab5.png" width="500">

### 코드 작성
* Dependency 셋업
`implementation 'com.google.firebase:firebase-config-ktx:21.2.0'`

* 앱을 처음 켰을 때 remote config로 데이터를 가져와서 해당 데이터를 출력하게 한다

* MainActivity

``` kotlin
// MainActivity의 내부
private fun initRemoteConfigData(){
    val remoteConfig: FirebaseRemoteConfig = Firebase.remoteConfig
     // 비동기로 데이터를 가져온다
    remoteConfig.setConfigSettingsAsync(   
        remoteConfigSettings {
            // 1시간 간격으로 데이터를 fetch 하게 한다
            minimumFetchIntervalInSeconds = 3600       
        }
    )

    remoteConfig.fetchAndActivate()
        // fetch가 됐을 경우 listener를 사용하여 결과를 알 수 있다
        .addOnCompleteListener { task -> 
            if (task.isSuccessful) {
                // 받아온 데이터를 Log로 출력
                Timber.d("remote config test message test: ${remoteConfig.getBoolean("test")}")
                Timber.d("remote config test message long_text: ${remoteConfig.getString("long_text")}")
            }
        }
}
```

### 출력 결과

<img src="https://user-images.githubusercontent.com/69494230/202617767-4223e950-3696-49e3-bf62-0556123c371f.png" width="400">

### 음? 이거 클라우드 Storage 같은거로도 할 수 있는거 아님?

네, 분명 비슷한 점도 있습니다
하지만, 다른 점이 더 많습니다!

여기까지 firebase의 remote config의 가장 기본적인 사용 방법을 알아봤으며
사실 이것 말고도 무궁무진한 방법이 존재한다

특히 조건을 넣어서 특정 유저나 일부 사람에게만 적용하거나 하는 기능을 사용할 수도 있다
또한 그 결과를 비교할 수 도 있다
(이 부분이 FCM와의 큰 차이점임)

또한 매 번 업데이트(fetch)를 하는 것이 아니라 1시간 마다 하는 등 주기를 설정할 수 도 있음