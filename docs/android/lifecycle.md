---
layout: default
title: Navigation을 사용한 화면 이동 시 라이프사이클 변화
parent: Android(코틀린)
---

# 개요
Jetpack의 Navigation을 사용하여 화면 이동을 했을 때 Activity, Fragment가 정확히 어떤 타이밍에 어떻게 lifecycle이 순환하는지 확인한다

-----------------
# 필요한 종속 항목 선언
https://developer.android.com/jetpack/androidx/releases/lifecycle#kotlin

```
def lifecycle_version = "2.6.0-alpha01"
def arch_version = "2.1.0"

// ViewModel
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
// ViewModel utilities for Compose
implementation "androidx.lifecycle:lifecycle-viewmodel-compose:$lifecycle_version"
// LiveData
implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
// Lifecycles only (without ViewModel or LiveData)
implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"
```

-------------------
# Activity
Activity에서 화면 이동을 했을 때 Acticity에서 라이프사이클이 어떻게 변화하는지 알아본다

<img width="832" alt="image" src="https://user-images.githubusercontent.com/69494230/181199520-874daaa1-acdb-4fda-a685-54129c4eb0fa.png">

## [MainActivity의 전체 코드] 

``` kotlin
package com.example.selftest

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import com.example.selftest.databinding.ActivityMainBinding
import kotlinx.coroutines.launch
import timber.log.Timber

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        Timber.plant(MyTimber())
        Timber.d("Activity onCreate")
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.CREATED){
                Timber.d("lifecycle Created")
            }
        }
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED){
                Timber.d("lifecycle Started")
            }
        }
        lifecycleScope.launch{
            repeatOnLifecycle(Lifecycle.State.RESUMED){
                Timber.d("lifecycle Resumed")
            }
        }
    }

    override fun onStart() {
        super.onStart()
        Timber.d("Activity onStart")
    }

    override fun onDestroy() {
        super.onDestroy()
        Timber.d("Activity onDestroy")
    }

    override fun onResume() {
        super.onResume()
        Timber.d("Activity onResume")
    }

    override fun onPause() {
        super.onPause()
        Timber.d("Activity onPause")
    }
}
```

각 lifecycle이 시작될 때 로그를 남기도록 했다

### 1. 처음 실행했을 때 라이프 사이클 변화
* 기본적으로 각각의 override 함수가 먼저 호출되고 그 다음에 각각의 상태에 대한 lifecycleScope가 호출된다
* onCreate -> onStart -> onResume

![image](https://user-images.githubusercontent.com/69494230/181203131-c906d8ef-27ec-4cfb-bcf9-5f5b7aea75c5.png)

### 2. 현재 실행중인 앱 확인 화면(네모 버튼)으로 갔다가 돌아왔을 때 라이프 사이클 변화
* 이 때는 lifecycle에 아무런 영향을 끼치지 않는다

### 3. Home 화면으로 갔다가 돌아왔을 때 라이프 사이클 변화
* (홈으로 이동) -> onPause -> (돌아옴) -> onStart ->onResume

![image](https://user-images.githubusercontent.com/69494230/201503669-be0904e9-fa48-4829-bf1b-c5d759b15d92.png)

### 4. 완전히 다른 화면을 봤다가 돌아왔을 때 라이프 사이클 변화
* onPause -> onStart -> onResume

![image](https://user-images.githubusercontent.com/69494230/181203229-192fc0c1-a345-421a-83f1-437f66d03da6.png)

단, 지금 코드처럼 하나의 lifecycleScope에 각각 **repeatOnLifecycle**로 출력한 것이 아니라
`lifecycleScope.launchWhenCreated {  }`로 지정하면 딱 한 번만 출력한다

### 5. 결론
Activity의 lifecycle이 먼저 출력되고 그 다음에 lifecycleScope의 lifecycle이 출력된다

-------------------
# Fragment
Fragment에서 화면 이동을 했을 때 라이프 사이클의 변화를 알아본다

주로 Fragment에서 발생하는 문제는 다른 Fragment에서 돌아왔을 때 observe 하고 있던 데이터를
다시 불러와서 **같은 데이터를 2번 set 하는 문제가 많다**
lifecycle을 확인하고 이를 방지하기 위핸 대책을 생각해본다

<img width="517" alt="image" src="https://user-images.githubusercontent.com/69494230/181222121-67598492-b563-4875-b642-4aa39207339e.png">

### 테스트 과정
1. FirstFragment, SecondFragment 이렇게 2개의 프래그먼트를 만든다
2. FirstFragment에만 라이프 사이클을 알 수 있게 로그를 달아 놓는다
3. FirstFragment에서 SecondFragment로 이동했다가 다시 FirstFragment로 돌아온다
4. 돌아오는 과정에서 라이프 사이클이 어떻게 변화하는지 확인한다
5. 프래그먼트 이동은 navigation을 사용한다

[FirstFragment의 전체 코드]

``` kotlin
class FirstFragment : Fragment() {
    private lateinit var binding: FragmentFirstBinding
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentFirstBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Timber.d("FirstFragment onViewCreated")
        binding.nextFragment.setOnClickListener {
            val action = FirstFragmentDirections.actionFirstFragmentToSecondFragment()
            findNavController().navigate(action)
        }
    }

    override fun onStart() {
        super.onStart()
        Timber.d("FirstFragment onStart")
    }

    override fun onResume() {
        super.onResume()
        Timber.d("FirstFragment onResume")
    }

    override fun onPause() {
        super.onPause()
        Timber.d("FirstFragment onPause")
    }

    override fun onStop() {
        super.onStop()
        Timber.d("FirstFragment onStop")
    }

    override fun onDestroy() {
        super.onDestroy()
        Timber.d("FirstFragment onDestroy")
    }
}
```

### 1. 첫 번째 프래그먼트를 열었을 때 라이프 사이클 변화
* onViewCreated -> onStart -> onResume

<img width="558" alt="image" src="https://user-images.githubusercontent.com/69494230/181679309-8cf22ab5-1c52-45f7-8515-dad3ffc63df0.png">

### 2. Navigation을 사용하여 첫 번째 프래그먼트에서 두 번째 프래그먼트로 이동했을 때 라이프 사이클 변화
* graph로 이동했을 때는 이전 fragment가 destroy되지 않고 stop 상태에 머물러 있음
* onPause -> onStop

<img width="432" alt="image" src="https://user-images.githubusercontent.com/69494230/181679511-6f7c043e-bd13-4cb8-8f44-ecc8f780ed7c.png">

### 3. Navigation을 사용하여 두 번째 프래그먼트에서 첫 번째 프래그먼트로 돌아왔을 때 라이프 사이클 변화
* destroy는 되지 않았지만 onViewCreated부터 다시 실행된다
* onViewCreated -> onStart -> onResume

<img width="536" alt="image" src="https://user-images.githubusercontent.com/69494230/181679661-ca6a87fd-6647-4602-9820-71579560a1c4.png">

### 4. 액티비티를 완전히 닫았을 때 라이프 사이클 변화
* Activity를 완전히 닫아야 Fragment가 Destroy된다
* onPause -> onStop -> onDestroy

<img width="471" alt="image" src="https://user-images.githubusercontent.com/69494230/181679800-be7253e0-08ec-4837-b898-c240f8ce38d7.png">

