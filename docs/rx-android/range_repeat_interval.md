---
layout: default
title: Range, Repeat, Interval 사용방법
parent: Rx-Android
---

# Range Operator
* Range Operator는 범위를 정해서 원하는 횟수 만큼 값을 넣어서 반복할 수 있다
* 또한 Observable을 반환하는 형태로 다음과 같이 람다식으로 쓸수도 있다

(람다식 형태가 더 많이 쓰인다. 단, onSubscribe 함수는 사라진다)

``` kotlin
fun rangeOperator(): Observable<Int> {
    return Observable.range(1, 10)
}

// 실행 함수
Operators.rangeOperator().subscribe(
            // 람다식으로 표현
	    // onNext
            {
                Timber.d("onNext: $it")
            },
	    // onError
            {
                Timber.e(it)
            },
	    // onComplete
            {
                Timber.d("onComplete")
            }
        )
```
### Range Operator 사용 결과

<img width="355" alt="image" src="https://user-images.githubusercontent.com/69494230/204238599-c8d479a9-f8fa-45e5-9420-3fbf22b21d97.png">

------
# Repeat Operator
* Repeat Operator를 사용하면 Observe 자체를 반복할 수 있게 해준다

``` kotlin
fun repeatOperator(): Observable<Int> {
    return Observable.range(1, 5).repeat(2)
}

// 위에서 사용한 rangeOperator를 2번 반복하게 한다
Operators.repeatOperator().subscribe(
            {
                Timber.d("onNext: $it")
            },
            {
                Timber.e(it)
            },
            {
                Timber.d("onComplete")
            }
        )
```
### Repeat Operator 사용 결과

<img width="337" alt="image" src="https://user-images.githubusercontent.com/69494230/204238955-f31f98b1-bdd4-42f0-9837-a997b12e6073.png">

------
# Interval Operator
* Interval Operator은 정해진 시간 만큼 반복하여 정해진 횟수를 반복했을 때 subscribe할 수 있다
* 이러한 기능은 정해진 시간마다 위치를 갱신해야하거나 페이지를 갱신하는 작업에 유용함

``` kotlin
fun intervalOperator(): Observable<Long> {
    return Observable.interval(2, TimeUnit.SECONDS)
}

// 매초 반복하면 2번 반복했을 때 onNext를 실시한다
Operators.intervalOperator().subscribe(
            {
                Timber.d("onNext: $it")
            },
            {
                Timber.e(it)
            },
            {
                Timber.d("onComplete")
            }
        )
```

### Interval Operator 사용 결과
<img width="312" alt="image" src="https://user-images.githubusercontent.com/69494230/204239271-32bfa85b-fa56-4443-bc40-5398e67b5ed1.png">
