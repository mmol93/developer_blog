---
layout: default
title: LicenseToolsPlugin을 사용해서 자동으로 라이브러리의 라이센스 공개하기
parent: Android(코틀린)
---

앱을 개발하고 배포할 때 반드시 해야하는 작업이 있습니다!

바로... 앱에 사용한 라이브러리들의 라이센스를 공개하는 일이죠

하지만 이 작업이 영 까다롭습니다 ㅠㅠ

어떤 라이브러리인지, 어디서 가져왔는지를 다 공개해야하니까요....

그럴 때!! 이 라이브러리를 사용하면 간단하게 현재 프로젝트에서 사용중인

라이브러리들의 라이센스를 공개할 수 있습니다.


# 관련 링크

[https://github.com/cookpad/LicenseToolsPlugin](https://github.com/cookpad/LicenseToolsPlugin)

# Dependency 선언

```kotlin
// project 수준의 gradle에서 선언
buildscript {
  repositories {
    maven {
      url "https://plugins.gradle.org/m2/"
    }
  }
  dependencies {
    classpath "gradle.plugin.com.cookpad.android.plugin:plugin:1.2.6"
  }
}

// app 수준의 gradle에서 선언
apply plugin: "com.cookpad.android.plugin.license-tools"
```

# 사용 방법

## 필요한 license 파일들을 생성하고 확인한다

- `licenses.yml` 파일 생성하기
    1. 안드로이드 스튜디오 터미널에 다음과 같이 입력한다
        - `./gradlew updateLicenses`
    2. 성공했다면 `./app/licenses.yml` 파일이 만들어진 것을 확인할 수 있다
        - 보기 설정을 `Android`가 아닌 `Project`로 해야 보인다
        - 실패 했다면 licenseCheck를 먼저 해보고 다시 실시해본다
    
- 생성된 파일을 확인하기
    1. 안드로이드 스튜디오 터미널에 다음과 같이 입력한다
        - `./gradlew checkLicenses`
    2. 성공했다면 `BUILD SUCCESSFUL` 메시지를 볼 수 있다

- `licenses.html` 파일 생성하기
    1. 안드로이드 스튜디오 터미널에 다음과 같이 입력한다
        - `./gradlew generateLicensePage`
    2. 성공 했다면 `./app/src/main/assets/licenses.html` 파일이 만들어진 것을 화인할 수 있다
        - 보기 설정을 `Android`가 아닌 `Project`로 해야 보인다

## 위 과정에서 발생할 수 있는 문제나 주의 사항

- licenses 파일들이 생성되는 과정
    1. `updateLicenses`를 통해 `licenses.yml` 파일을 생성한다
    2.   `./gradlew generateLicensePage`는  `licenses.yml`을 참조하여 `licenses.html` 파일을 생성한다
    3. 그렇기 때문에 대부분의 문제는 `licenses.yml`에 있는 내용 때문에 발생한다

- **Could not generate the copyright statement** 라는 에러가 발생했을 때
    - 해당 `artifact` 안에 `skip: true` 항목을 추가 해준다
        - 이는 generate할 때 해당 라이브러리의 라이센스 생성을 하지 않겠다는 의미다
    - 또는 `copyrightHolder`에 값을 넣어준다
        - 처음 생성되었다면 `copyrightHolder` 에는 *`#COPYRIGHT_HOLDER#`* 같은 무의미한 값이 들어있다
        - 하지만 이를 삭제하고 Google 같이 실제 해당 라이브러리를 소유하고 있는 or 제작한 회사 or 개인의 이름 등을 넣어두면된다

## 생성된 html 파일을 보여주기

- 생성된 html 파일을 보여주는 것 만으로도 끝이 난다
- 아래와 같은 모습의 html이 됩니다

<img src="https://user-images.githubusercontent.com/69494230/202360051-fbda831c-14e4-4569-94bb-17a4f91555a2.jpg" width="300">
