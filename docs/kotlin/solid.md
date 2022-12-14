---
layout: default
title: 안드로이드(Kotlin)를 SOLID로 설계하기
parent: 코틀린
---

# SOLID 설계란 무엇인가?
* S : Single Responsibility Principle (SRP) = 단일 책임 원칙
* O : Open/Closed Principle (OCP)
* L : Liskov Substitution Principle (LSP) = 리스코프 치환 원칙
* I : Interface Segregation Principle (ISP) = 인터페이스 분리 원칙
* D : Dependency Inversion Principle (DIP) = 의존성 반전 원칙

SOLID 설계를 통해 이하기 쉽고 변경하기 쉬운 코드를 만들 수 있습니다.

하지만 반드시 **모든 앱을 만들 때 SOLID 설계를 적용할 필요는 없으며**
자신의 프로젝트에 적절하게 사용하면된다.

-------

# S - 단일 책임 원칙(SRP)
* SOLID 설계에서 S에 해당하는 부분이다
* 모듈은 하나의 일만 처리해야한다
   * 모듈은 하나의 소스 파일을 의미한다
* 아래와 같은 소스 파일이 있다고 생각한다

``` kotlin
class Order {
    
    fun sendOrderUpdateNotification() {
        // sends notification about order updates to the user.
    }
    
    fun generateInvoice() {
        // generates invoice
    }

    fun save() {
        // insert/update data in the db
    }
}
```

자세히 보면 위의 Order class는 복수의 책임을 갖는다
* Oder라는 소스 파일(모듈) 안에 다수의 역할을 하는 함수(fun)가 들어있다

그렇기 때문에 아래와 같이 바꿀 수 있다

``` kotlin
// 아래의 class들은 각각 다른 파일로 이루어져있다고 생각한다

data class Order(
    val id: Long,
    val name: String,
    // ... other properties. 
)


class OrderNotificationSender {

    fun sendNotification(order: Order) {
        // send order notifications
    }
}


class OrderInvoiceGenerator {

    fun generateInvoice(order: Order) {
        // generate invoice
    }
}


class OrderRepository {

    fun save(order: Order) {
        // insert/update data in the db.
    }
}
```

사용할 때는 다음과 같이 하나의 클래스에 책임을 위임하여 사용할 수 있다

<img width="693" alt="image" src="https://user-images.githubusercontent.com/69494230/207488973-fb2300af-d0b3-43aa-9b75-a3a73b4517ac.png">

``` kotlin
class OrderFacade(
    // 각각의 클래스 변수를 받는다 = 1개의 클래스가 1개의 책임을 가짐
    private val orderNotificationSender: OrderNotificationSender,
    private val orderInvoiceGenerator: OrderInvoiceGenerator,
    private val orderRepository: OrderRepository
) {
     fun sendNotification(order: Order) {
        // sends notification about order updates to the user.
        orderNotificationSender.sendNotification(order)
    }

    fun generateInvoice(order: Order) {
        // generates invoice
        orderInvoiceGenerator.generateInvoice(order)
    }

    fun save(order: Order) {
        // insert/update data in the db
        orderRepository.save(order)
    }
}
```

* 하지만 이렇게 모든 클래스에 단일 책임을 부여하면 클래스 파일이 기하급수로적으로 늘어나게 된다

* 그렇기 때문에 SOLID 설계를 완벽하게 적용하는 것 보다 적절하게 적용하는 편이 좋다...

-------

# O - OCP(Open/Closed Principle)
* SOLID 설계에서 O에 해당하는 부분이다
* Open/Closed Principle 정의를 하자면 다음과 같다

> 소프트웨어 아티팩트는 확장을 위해 열어야 하지만 수정을 위해 닫아야 한다.

> 즉, 소프트웨어 아티팩트의 동작은 아티팩트를 수정할 필요 없이 확장 가능해야 한다.

### OCP를 위반하는 예

``` kotlin
enum class Notification {
    PUSH_NOTIFICATION, EMAIL
}
```


``` kotlin
class NotificationService {

    fun sendNotification(notification: Notification) {
        // notificagion의 값에 따라 다른 작업을 실행하게 한다
        when (notification) {
            Notification.PUSH_NOTIFICATION -> {
                // send push notification
            }

            Notification.EMAIL -> {
                // send email notification
            }
        }
    }
}
```

딱히 실행하는데 문제가 되지도 않고 OCP를 위반하는거도 없어보입니다.

하지만, 여기서 새로운 Notification을 추가할 경우 어떻게 될까요?

`NotificationServiceSMS`를 추가해야한다고 가정하면 아래와 같이 될 것입니다

``` kotlin
enum class Notification {
    PUSH_NOTIFICATION, EMAIL, SMS
}
```

``` kotlin
class NotificationService {

    fun sendNotification(notification: Notification) {
        when (notification) {
            Notification.PUSH_NOTIFICATION -> {
                // send push notification
            }

            Notification.EMAIL -> {
                // send email notification
            }

            Notification.SMS -> {
                // send sms notification
            }
        }
    }
}
```

여기서 눈치 챘을지 모르겠지만

새로운 값을 추가할 때 마다 `NotificationService`를 수정해야합니다.

그렇기 때문에 **OCP를 위반한다**라고 볼 수 있습니다.

* 다음과 같은 방법으로 해결할 수 있습니다.

``` kotlin
// 클래스가 아닌 인터페이스를 만든다
interface Notification {
    fun sendNotification()
}
```

그리고 각각의 Notification에 대한 클래스 파일을 만든다

``` kotlin
// Notification interface를 상속 받으며
class PushNotification : Notification {
    // 각 함수를 재정의 하는 형태로 구현한다
    override fun sendNotification() {
        // send push notification
    }
}
```


``` kotlin
class EmailNotification : Notification {
    override fun sendNotification() {
        // send email notification
    }
}
```

이를 통합해줄 `NotificationService`라는 클래스 파일을 만든다

``` kotlin
class NotificationService {

    fun sendNotification(notification: Notification) {
        notification.sendNotification()
    }
}
```

이렇게 하면 관련 Notification 파일을 수정하지 않고 다음과 같이 파일을 추가하기만 해서
새로운 기능을 추가할 수 있게 된다

``` kotlin
class SMSNotification : Notification {
    override fun sendNotification() {
        // send sms notification
    }
}
```

* 이렇게 interface를 활용하면 공통된 부분을 활용함과 동시에 필요에 따라 재정의 해서 사용할 수 도 있다
* **OCP 또한 100% 지키기 매우 어렵기 때문에** 코드를 짤 때 항상 의식하며 적절히 사용하는 것이 중요하다

-------

# L - Liskov Substitution Principle(LSP)
* SOLID 설계에서 L에 해당하는 부분이다
* 정의하자면 다음과 같다

> 자료형 S가 자료형 T의 하위형이라면 필요한 프로그램의 속성의 변경 없이
> 자료형 T의 객체를 자료형 S의 객체로 치환할 수 있어야 한다

좀 더 간단히 말하자면 

> 똑같이 사용했을 때 자식 객체는 부모객체의 능력을 그대로 사용할 수 있어야한다.

(이거에 대한 자세한 설명은 다른 포스팅에서 진행하겠습니다)

바로 예시로 넘어가면 다음과 같은 코드를 만들 예정입니다.

* 쓰레기를 받아서 분류를 하는 서비스

<img width="695" alt="image" src="https://user-images.githubusercontent.com/69494230/207506968-1f72cd11-aadb-45c3-8d76-e89ac4d2ad8c.png">

``` kotlin
interface Waste {
    fun process()
}
```

Waste interface를 상속받는 클래스 구현

``` kotlin
class OrganicWaste : Waste {
    override fun process() {
        println("Processing Organic Waste")
    }
}
```

``` kotlin
class PlasticWaste : Waste {
    override fun process() {
        println("Processing Plastic Waste")
    }
}
```

이 둘을 사용하는 클래스 구현

``` kotlin
class WasteManagementService {

    fun processWaste(waste: Waste) {
        waste.process()
    }
}
```
이를 사용하는 함수 정의
* 똑같이 `wasteManagementService`의 `processWaste`를 호출하지만 서로 영향을 주지 않고 같은 처리를 하고 있기 때문에 **리스코프 치환 원칙**을 만족한다

``` kotlin
fun main() {
    val wasteManagementService = WasteManagementService()

    var waste: Waste

    waste = OrganicWaste()
    wasteManagementService.processWaste(waste) // Output: Processing Organic Waste

    waste = PlasticWaste()
    wasteManagementService.processWaste(waste) // Output: Processing Plastic Waste
}
```

-------

# I - 인터페이스 분리 원칙(ISP)
* SOLID 설계에서 I에 해당하는 부분이다
* 정의는 다음과 같다
   * 사용하지 않는 인터페이스를 상속하는 클래스에서 불필요한 메서드를 구현하도록 강요해선 안된다

* 나쁜 예시

``` kotlin
interface OnClickListener {
    fun onClick()
    fun onLongClick()
}
```

다음과 같이 항상 onClick()과 onLongClick()을 둘 다 정의해야한다

``` kotlin
class CustomUIComponent : OnClickListener {
    override fun onClick() {
        // handles onClick event.
    }

    // left empty as I don't want the [CustomUIComponent] to have long-click behavior.
    override fun onLongClick() {

    }
}
```

* 이를 해결하기 위해선 그냥 interface를 나누면된다

``` kotlin
interface OnClickListener {
    fun onClick()
}
```


``` kotlin
interface OnLongClickListener {
    fun onLongClick()
}
```

그리고 필요한 인터페이스만 상속해서 사용한다

``` kotlin
class CustomUIComponent : OnClickListener {
    override fun onClick() {
        // handle single-click event
    }
}
```
-------

# D - Dependency Inversion Principle(DSP; 의존성 반전 원칙)
* SOLID 설계에서 I에 해당하는 부분이다
* 정의는 다음과 같다
   * 프로젝트는 서로 독립된 모듈을 갖고 있다
   * 하위 모듈은 상위 모듈을 참조(의존)하고
   * 상위 모듈은 하위 모듈을 참조(의존)할 수 없다
      * 이를 "의존성 반전 원칙" 이라고 부른다
* 간단히 말하면 "쉽게 변하지 않는 요소에만 의존하되 상위 모듈은 하위 모듈에 의존할 수 없다"이다
* 예시 코드

``` kotlin
class ClassA {
    fun doSomething() {
        println("Doing something")
    }
}

class ClassB {
    fun doIt() {
        // ClassB 는 ClassA가 반드시 있어야만 사용할 수 있다 = 즉, ClassA에 대해 의존성을 가진다
        val classA = ClassA() 
        classA.doSomething()
    }
}
```
* 다음과 같이 의존성을 가지는 프로그램이 있다고 가정

<img width="457" alt="image" src="https://user-images.githubusercontent.com/69494230/207514564-86619dd7-ebe8-433f-a1ce-c441ecebbb7d.png">

이메일을 보내는 Notification

``` kotlin
class EmailNotification {
    fun sendNotification(message: String) {
        println("Sending email notification with message \"$message\"")
    }
}
```

실질적으로 Notification을 보내는 부분

``` kotlin
class NotificationService {

    fun sendNotification(message: String) {
        // 이 부분에서 반드시 EmailNotification에 의존함
        val emailNotification = EmailNotification()
        emailNotification.sendNotification(message)
    }
}
```
실행 함수

``` kotlin
fun main() {
    val notificationService = NotificationService()
    
    // Output: Sending email notification with message "Happy Coding"
    notificationService.sendNotification("Happy Coding") 
}
```

* 여기까지만 하면 의존성에는 문제가 없지만 다른 유형의 Notification을 보낼 수 없다는 문제가 있다
* 다음과 같은 방법으로 의존성을 제거하고 `NotificationService`가 Email뿐 아니라 다른 유형의 Notification을 보내게 할 수 있다

<img width="692" alt="image" src="https://user-images.githubusercontent.com/69494230/207515357-2e75dcb7-c54a-4f11-aa00-1d3305406d58.png">

``` kotlin
interface Notification {
    fun sendNotification(message: String)
}
```

``` kotlin
class EmailNotification : Notification {
    override fun sendNotification(message: String) {
        println("Sending email notification with message \"$message\"")
    }
}
```

``` kotlin
class SmsNotification : Notification {
    override fun sendNotification(message: String) {
        println("Sending sms notification with message \"$message\"")
    }
}
```

``` kotlin
fun main() {
    val message = "Happy Coding"
    val notificationService = NotificationService()
    var notification: Notification

    notification = EmailNotification()
    notificationService.notification = notification
    notificationService.sendNotification(message)
    // Output: Sending email notification with message "Happy Coding"

    notification = SmsNotification()
    notificationService.notification = notification
    notificationService.sendNotification(message)
    // Output: Sending sms notification with message "Happy Coding"
}
```

이렇게 독립된 의존성과 OCP를 동시에 만족할 수 있다

-------

# SOLID 설계의 한 줄 정의
* SRP: 각 소프트웨어 모듈은 하나의 역할만 담당해야한다

* OCP: 소프트웨어 시스템은 변경이 용이해야 한다. 기존 코드를 변경하는 것이 아니라 새로운 코드를 추가하여 시스템 동작을 변경할 수 있도록 설계해야 한다.

* LSP: 교환 가능한 부품으로 소프트웨어 시스템을 구축하려면 해당 부품을 **서로** 교환할 수 있는 계약에 따라야 합니다.

* ISP: 소프트웨어 설계자는 사용하지 않는 항목에 의존하지 않도록 해야 한다.

* DIP: 상위 수준의 정책을 구현하는 코드는 하위 수준의 세부 정보에 의존해서는 안 됩니다.
