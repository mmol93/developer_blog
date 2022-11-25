---
layout: default
title: just, fromArray, FromIterable Operator
parent: Rx-Android
---

원하고는 operator를 찾고 싶다면 검색을 하면됩니다!

# 개요
1. just Operator를 알아본다
2. fromArray Operator를 알아본다
3. FromIterable Operator를 알아본다

---------

# just Operator
받은 값을 가공없이 그대로 emit해주는 operator

### 예시 코드

``` kotlin
// just는 subscribe를 했을 때 1이라는 값을 emit 하도록한다
val observable = Observable.just(1)

// Observe할 값이 어떠한 값으로 들어오는 지정해줘야한다
val observer = object: Observer<Int>{
    // subscribe되었을 때 호출
    override fun onSubscribe(d: Disposable) {
        Timber.d("onSubscribe")
    }
    // 성공적으로 끝났을 때 호출
    override fun onNext(t: Int) {
        Timber.d("onNext: $t")
    }
    // 에러가 발생했을 때 호출
    override fun onError(e: Throwable) {
        Timber.d("onError")
    }
    // 성공 유무 상관없이 해당 subscribe에 대한 작업이 다 끝나면 호출
    override fun onComplete() {
        Timber.d("onComplete")
    }
}
// subscribe 하기
observable.subscribe(observer)
```
### 실행 결과

<img width="291" alt="image" src="https://user-images.githubusercontent.com/69494230/203692773-b358bd65-c4fd-44c5-9db1-c42c97f76117.png">

결과를 보면 onSubscribe → onNext → onComplete 순서로 출력된다

--------

# FromArray Operator
Array 형태의 데이터를 받아서 emit할 수 있다

받은 Array 형태의 데이터를 여러 방법으로 가공한 다음 emit할 때 많이 쓰인다

``` kotlin
private val arrayNum =arrayOf(1, 2, 3, 4)
private val arrayNum2 =arrayOf(10, 20, 30, 40)
fun fromOperator() {
    val observable = Observable.fromArray(arrayNum, arrayNum2)

    val observer = object: Observer<Array<Int>> {
        override fun onSubscribe(d: Disposable) {
            Timber.d("onSubscribe")
        }

        override fun onNext(t: Array<Int>) {
            Timber.d(t.toString())
        }

        override fun onError(e: Throwable) {
            Timber.d("onError")
        }

        override fun onComplete() {
            Timber.d("onComplete")
        }
    }
    observable.subscribe(observer)
}
```
----

# FromIterable Operator

동적 Iterable 데이터를 observe할 수 있다

Iterable을 받지만 onNext에선 별도 처리를 하지 않아도 값을 1개씩 확인할 수 있다

private val mListNum =mutableListOf(1, 2, 3, 4, 5)

``` kotlin
fun fromIterableOperator() {
    val observable = Observable.fromIterable(mListNum)
    val observer = object: Observer<Int>{
        override fun onSubscribe(d: Disposable) {
            Timber.d("onSubscribe")
        }
				// forEach를 사용하지 않아도 요소를 1개씩 출력한다
        override fun onNext(t: Int) {
            Timber.d("onNext: $t")
        }

        override fun onError(e: Throwable) {
            Timber.d(e)
        }

        override fun onComplete() {
            Timber.d("onComplete")
        }
    }
    observable.subscribe(observer)
}
```
### 출력 결과

<img width="287" alt="image" src="https://user-images.githubusercontent.com/69494230/203744797-8b746278-841e-4eab-80c8-eee066485c4d.png">
