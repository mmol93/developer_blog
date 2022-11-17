---
layout: default
title: 안드로이드 스튜디오 무선 디버깅 에러 안됨과 사용 방법
parent: Android(코틀린)
---

# 개요
1. 무선 디버깅 방법을 알아본다
2. 무선 디버깅 연결을 시도 했는데 연결이 되지 않을 때 해결 방법을 알아본다

안드로이드11 부터 무선 디버깅을 간편하게 할 수 있게 되면서 최근에는 USB 디버깅을 거의 사용하지 않은거 같네요

무선 디버깅의 장점이라고 하면 

**1. 중간에 선 접촉 문제로 연결이 끊길 일이 없다**
**2. 같은 와이파이가 연결되어있는 환경이라면 이동하면서도 디버깅을 할 수 있다**

정도 있겠네요

여러분들도 어서 무선 디버깅으로 갈아타시길 바라며...

# 무선 디버깅 방법
### 1. 먼저 안드로이드11 이상의 단말기가 필요하다
제가 소개하려는 방법은 안드로이드 11 이상의 단말기가 필요합니다.

### 2. 안드로이드 스튜디오에서 pair device using Wi-Fi를 클릭한다

![image](https://user-images.githubusercontent.com/69494230/202098721-9d3f5c0e-874c-4802-9e2a-527db866810d.png)

### 3. 이 화면을 띄운 상태에서 휴대폰으로 QR 코드를 읽게 한다

<img width="392" alt="image" src="https://user-images.githubusercontent.com/69494230/202101798-cae4f70a-b3e9-4a07-88ac-827a1ecbfa61.png">
* 대신 그냥 QR 코드 읽기 모드로 읽는 것이 아니라
   * USB 디버깅 모드 아래에 **무선 디버깅** 항목이 있으며
      * 여기에 있는 QR 코드로 디바이스 페어링을 통해 QR 코드를 읽어야 한다
      
<img width="392" alt="image" src="https://user-images.githubusercontent.com/69494230/202102506-023451d3-83d7-48f0-9689-ed6ea31ea7ad.png">

### 4. 여기까지 하면 페어링이 완료된다

# 무선 디버깅 연결이 안될 때

하다보면 _연결중_ 상태에서 넘어가지 않거나 실패하는 상황이 발생한다
그 때는 다시 페어링 해봐도 되지 않는데 이 때 이를 해결하는 방법은...

### 와이파이를 껏다가 다시 킨후 위 단계를 다시 반복하는 것이다

(필자는 이 방법을 몰랐을 때 그냥 다시 USB로 연결해서 디버깅을 했다 ㅠㅠ)

#debug