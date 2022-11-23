---
layout: default
title: Rx-Android란?
parent: Rx-Android
---

# Rx-Java (Rx-Android)란?

일반적인 프로그래밍은 Imperative Programming(명령형 프로그래밍)으로

우리가 입력한 코드 순서대로 프로그램을 실행한다

이에 비해 Rx-Reactive(통칭 Rx-Java 등)은 반응형으로 기본적으로 똑같이 순서대로 흐르지만

이벤트 등에 원하는 타이밍에 실행되거나 할 수 있다

**즉, 특정한 반응이 있을 때 실행된다는 것이 가장 큰 차이점이다**

기존의 프로그래밍 방식을 **Pull 방식**, Reactive 프로그래밍 방식을 **Push 방식**이라고도 한다.

----------------

- RxJava의 특징
    - ReactiveX는 관찰가능한 절차를 통해 비동기, 이벤트 기반 프로그램을 구성하기 위한 라이브러리이다.
    - Observer Pattern을 확장하며, Sequence를 조합할 수 있는 연산자를 지원한다.
    - low-level Thread, 동기화, Thread 안전성, non-blocking I/O에 관한 우려를 줄인다.

-----------------
## 1. RxJava의 구성 요소

### 1. Observable

데이터의 흐름에 맞게 Comsumer에 이벤트를 보내는(emit 하는) Class

### 2. Operator

특정 데이터를 다른 폼으로 변환 후 방출(emit)하게 한다

### 3. Observer

특정 값을 감시하다가 값이 바뀌면 해당 값을 가져온다 = Comsume 한다

-----------------

## 2. Dependecy 추가

Rx-Android를 사용하기 위해선 다음과 같은 dependency를 추가해야한다

```kotlin
// RX-java - https://github.com/ReactiveX/RxJava
implementation "io.reactivex.rxjava3:rxjava:3.1.5"

// RX-android - https://github.com/ReactiveX/RxAndroid
implementation "io.reactivex.rxjava3:rxandroid:3.0.0"
```

-----------------

## 3. RxJava Operator

공식 홈페이지에 각 Operator들에 대한 설명과 종류가 나와있다

[https://reactivex.io/documentation/operators.html](https://reactivex.io/documentation/operators.html)