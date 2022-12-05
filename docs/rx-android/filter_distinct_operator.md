---
layout: default
title: Rx-java의 Filter, Distinct Operator 기초 설명
parent: Rx-Android
---

# 개요
Rx-java의 Filter Operator와 Distinct Operator의 기본적인 사용 방법을 알아본다

# Filter Operator 사용 방법
* 결과 값에 filter를 적용하여 특정 값만 얻을 수 있다

``` kotlin
private val mListNum = mutableListOf(1, 2, 3, 4, 5)

fun filterOperator(): Observable<Int> {
    // 보통 filter는 Iterable 데이터형과 많이 쓰기 때문에 fromIterable을 사용했다
    return Observable.fromIterable(mListNum)
}

// 실행 함수
Operators.filterOperator()
	    // subscribe 하기 전에 filter를 넣어서 조건을 지정할 수 있다
            .filter { it > 2 }
            .subscribe(
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

* Filter Operator 실행 결과

<img width="150" alt="image" src="https://user-images.githubusercontent.com/69494230/205600100-bade600f-4a92-4060-8147-c3fe530a7392.png">

------
# Distinct Operator
* 같은 요소는 2번 emit 하지 않는 Operator

``` kotlin
private val mListNum =mutableListOf(1, 2, 3, 4, 5, 3, 4, 4, 5)

fun distinctOperator(): Observable<Int> {
    return Observable.fromIterable(mListNum)
}

Operators.distinctOperator()
            .distinct()
            .subscribe(
                {
                    Timber.d("onNext: $it")
                },
                {
                    Timber.e(it)
                }
            )

// 출력 결과 - 결과가 연속으로 같은 값이 아니더라도 과거에 출력한 적이 있다면 emit하지 않음
onNext: 1
onNext: 2
onNext: 3
onNext: 4
onNext: 5

// distinct는 lamda로 쓸 수 도 있다
data class User(val name: String, val age: Int)

private val user = listOf(
        User("kim", 20), 
        User("aim", 22),
        User("eim", 25),
        User("lim", 20),
        User("zim", 22)
    )

fun distinctOperator(): Observable<User> {
        return Observable.fromIterable(user)
    }

// user의 age를 distinct 하도록 한다
Operators.distinctOperator()
            .distinct {
                it.age
            }
            .subscribe(
                {
                    Timber.d("onNext: $it")
                },
                {
                    Timber.e(it)
                }
            )


// 출력 결과 - 같은 나이를 가진 user는 distinct 처리됨
onNext: User(name=kim, age=20)
onNext: User(name=aim, age=22)
onNext: User(name=eim, age=25)
```
