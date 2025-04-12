# RxSwift 스터디 정리

> [!NOTE]
> RxSwift를 학습하며 스터디 중 논의한 내용을 정리한 문서입니다.

### RxSwift의 등장 배경

RxSwift는 비동기 처리를 더 쉽고 일관성 있게 구현하기 위해 등장했습니다.  
기존에는 비동기 작업을 완료 클로저(completion handler)를 통해 처리하는 방식이 일반적이었습니다.  
하지만 이 방식은 중첩 호출이 많아질수록 코드의 가독성과 유지보수성이 떨어지는 **콜백 지옥(callback hell)** 을 유발하게 됩니다.

특히 iOS 개발 환경에서는 이런 콜백 패턴이 중첩되면서 흐름을 파악하기 어려워지고, 에러 핸들링이나 데이터 흐름 제어가 복잡해지는 문제가 있었습니다. 이러한 문제를 해결하고자 **함수형 리액티브 프로그래밍(FRP)** 개념을 적용한 RxSwift가 등장하게 되었습니다.

### RxSwift는 왜 이렇게 디자인되었을까?

RxSwift는 초기 설계에 있어 **RxJava의 영향을 강하게 받은 구조**를 가지고 있습니다.  
RxJava는 Java 언어 기반의 리액티브 확장이며, 그 특성상 **클래스 기반 상속 구조**를 많이 사용합니다. 이 영향으로 RxSwift 역시 초창기에는 클래스 상속에 기반한 구조가 많았습니다.

Java는 모든 것이 객체이고 상속이 자연스러운 언어입니다. 반면 Swift는 **값 타입(value type)** 과 **프로토콜 지향 프로그래밍**을 강조하는 언어이므로, 시간이 지나면서 RxSwift도 **Swift스러운 구조**를 점점 도입하게 됩니다. 예를 들어, 일부 API나 내부 구조는 점차 `protocol`을 중심으로 리팩토링되고 있는 추세입니다.

또한 Rx의 설계는 **Push 기반 데이터 흐름**, 즉 Backpressure 문제를 고려하고 있으며, 다양한 오퍼레이터들을 체이닝 형태로 연결할 수 있게 유기적으로 구성되어 있습니다. 이는 복잡한 비동기 흐름을 선언적(declarative)으로 쉽게 표현할 수 있도록 하기 위한 의도입니다.

### Disposable, DisposeBag은 무엇이며 사용하는 이유가 무엇인가요?

Observer가 구독할 때 반환값으로 Disposable을 받게 됩니다. 이 Disposable 인스턴스를 통해 Observer는 dispose(unsubscribe)를 할 수 있습니다.

DisposeBag은 Disposable 객체들을 관리하는 클래스 입니다. DisposeBag 인스턴스가 deinit 될 때, 배열로 관리중인 Disposable들을 “스레드-세이프”하게 dispose하며 deinit됩니다.

Observer가 더 이상 구독을 유지할 필요가 없을 때 구독을 해지 하지 않으면 메모리 누수가 발생할 수 있습니다. 구독시 반환 받은 Disposable을 DisposeBag에 넣어 관리하면, Disposbag의 생명주기와 Disposable들의 생명주기를 함께 일괄적으로 관리할 수 있습니다.

+) Disposable 각각에 dispose를 해주지 않아도 되는 편리함이 있습니다(?)

### RxSwift에서의 Back Pressure는 무엇이며 어떻게 처리할 수 있나요?

백프레셔란, 데이터의 흐름이 downstream(소비자)에서 upstream(생산자)로 이어지는 것을 말합니다. 또는 데이터의 흐름을 생산자가 아닌 소비자 쪽에서 제어하는 것을 말합니다. RxSwift에서는 throttle, buffer, debounce를 통해 흐름을 제어할 수 있습니다.

### bind와 drive의 차이를 설명해 주세요.

bind와 drive 모두 subscribe를 래핑한 메서드입니다. bind는 onNext에 해당하는 이벤트만 발행할 수 있고, drive는 subscribe와 동일하지만 메인쓰레드에서 동작한다는 특징이 있습니다.

### merge, zip, combineLatest 차이, 데이터 타입이 상관없는 것은?

merge는 여러 스트림을 하나의 스트림으로 연결하는 오퍼레이터입니다. 각 스트림이 이벤트를 방출할 때마다 하나의 스트림으로 이벤트를 방출합니다. 따라서 동일한 데이터 타입이어야 합니다.

zip은 여러 스트림에서 방출하는 n번째 요소들을 튜플로 묶어 방출합니다. 모든 스트림에서 각각 하나씩 방출해야 하나의 데이터로 묶어 방출합니다. 서로 다른 데이터 타입을 함께 사용할 수 있습니다.

combineLatest는 가장 최근에 방출한 값들을 튜플로 묶어 방출합니다. 모든 스트림에서 1개 이상씩 방출한 이후, 어떤 스트림에서 하나라도 방출하면 데이터를 방출합니다. 서로 다른 데이터 타입을 함께 사용할 수 있습니다.

### 핫 옵저버블과 콜드 옵저버블의 차이에 대해 설명해주세요.

핫 옵저버블은 옵저버블이 만들어지는 즉시 이벤트를 발행하기 시작합니다. 그리고 핫 옵저버블을 구독하는 옵저버는 구독 시점 이후의 데이터부터 관찰합니다.

콜드 옵저버블은 옵저버가 구독하는 시점부터 이벤트를 발행하기 시작합니다. 따라서 콜드 옵저버블의 경우 시퀀스 전체의 데이터를 모두 관찰할 수 있음을 보장받습니다.

### Subject vs RxRelay 의 차이점에 대해 설명해주세요.

Subject는 옵저버블과 옵저버 역할을 동시에 할 수 있는 클래스입니다. RxRelay는 Subject를 래핑한 클래스로 옵저버에게 onNext를 통한 값의 전달만 가능하고, onError나 onComplete는 방출하지 않는다는 특징이 있습니다. 

Subject는 onError 혹은 onComplete가 발생하면 스트림이 종료될 수 있지만, RxRelay는 onNext를 통한 값 방출만 가능하기 때문에 스트림 종료되지 않습니다. 따라서 UI 바인딩과 같은 스트림의 유지가 필요할 경우 용이합니다.

### throttle, debounce 차이를 설명해주세요.

throttle은 이벤트 방출 이후 일정 시간 동안 다른 이벤트를 방출하지 않습니다. debounce는 일정시간 동안 방출된 이벤트 중 마지막 이벤트만 방출합니다.

throttle은 시퀀스 중 마지막 이벤트를 방출할 지 하지 않을 지 선택할 수 있습니다.

### Observable의 생명주기에 대해 설명해주세요.

observable은 생성되면 Next 이벤트를 방출하고 .completed 또는 .error를 방출하여 처리하고 나서 dispose로 메모리에서 해제시킵니다.
Observable은 구독(subscribe())이 발생하면 이벤트 흐름이 시작되며, .next 이벤트들을 방출하다가 .completed 또는 .error 이벤트로 종료됩니다. 이후에는 dispose()를 통해 구독을 해제하고 리소스를 정리합니다.

### AtomicInt

`AtomicInt`는 정수(Int)의 래퍼(Wrapper) 타입으로 멀티스레드 환경에서도 안전하게 연산할 수 있습니다.

RxSwift 내에서 **구독 상태 관리**, **레퍼런스 카운팅(ex: Disposable이 몇번 호출 되었는지)** 등에 사용됩니다.

### DisposeBase 사용 이유

과거에는 동기화 관련 로직을 수동으로 구현했지만, 리팩토링 과정에서 `DisposeBag` 등 동기화 관련 로직이 생겨나게 되었습니다.

이후 `DisposeBase`는 **Rx 내부 리소스를 추적**하기 위한 용도로 활용되고 있습니다.

개발자는 이를 통해 **디버깅 시 리소스 누수를 추적**하는 데 사용할 수 있습니다.


### Swift에서 Unmanaged란?

Swift의 `Unmanaged`는 **포인터를 직접 제어하는 기능**입니다.

ARC에 의존하지 않고 객체의 **retain/release를 수동으로 처리**해야 하며, 메모리 관리를 명확히 알고 있어야 안전하게 사용할 수 있습니다.


### SynchronizationTracker

Rx 내부에서 동기화 상황을 추적하기 위한 **내부 디버깅 도구**입니다.

사용자보다는 **Rx 라이브러리 개발자들을 위한 기능**으로 보입니다.

### Reactive 확장을 통한 커스텀 이벤트 예시

Reactive를 확장해 다양한 커스텀 이벤트를 만들 수 있습니다.

```swift
public extension Reactive where Base: UIGestureRecognizer {
  var tap: Observable<Base> {
    event.filter { $0.state == .recognized }
  }
}

public extension Reactive where Base: UIViewController {
  var viewDidLoad: Observable<Void> {
    self.methodInvoked(#selector(Base.viewDidLoad))
      .map { _ in }
  }
}
```

### Subject와 RxRelay

`RxRelay`는 내부적으로 `Subject`를 래핑한 클래스입니다. 기능적으로는 유사하지만, 목적과 안정성 면에서 서로 다른 특성을 갖고 있어 사용 상황에 따라 적절한 선택이 필요합니다.

UI 바인딩을 할 때는 일반적으로 `RxRelay`를 사용하는 것이 더 적절합니다. 그 이유는 `Relay`는 에러를 방출하지 않으며 스트림이 끊기지 않고 지속적으로 데이터를 전달할 수 있기 때문입니다. 반면 `Subject`는 `.onError()` 호출 시 스트림이 종료되므로, UI 바인딩과 같이 지속적이고 안정적인 흐름이 필요한 곳에서는 문제가 발생할 수 있습니다.

UI 바인딩이 아닌 경우에도 스트림의 연결이 끊기지 않기를 원한다면, `Relay`를 사용하는 것이 적절할 수 있습니다. 물론 기술적으로는 `Subject` 하나만으로도 대부분의 상황을 처리할 수 있습니다. 그러나 실제 적용에서는 상황에 따라 선택이 달라져야 합니다. 예를 들어, 오류 처리가 필요한 경우이거나 구독이 끊겨도 괜찮은 경우라면 `Subject`를, 반대로 스트림이 유지되어야 한다면 `Relay`를 선택하는 것이 좋습니다.

Rx는 출시된 지 15년이 넘은 만큼 다양한 상황에 맞춰 사용할 수 있도록 여러 커스텀 Observable 타입이 도입되었습니다. `Relay`, `Driver` 등이 그 예입니다. 이들은 초보자나 일반 사용자도 Rx의 복잡한 내부 동작을 몰라도 쉽게 사용할 수 있도록 하기 위한 목적을 가지고 있습니다.

예를 들어, `Subject`에 대한 개념이 부족한 상태에서 UI 바인딩을 `Subject`로 처리할 경우, 오류 발생 시 스트림이 끊기며 큰 문제가 발생할 수 있습니다. 이러한 사고를 예방하기 위해 별도의 안전한 타입인 `Relay`가 도입되었을 가능성이 높습니다. 다시 말해, 개발자가 세부 동작을 몰라도 안전하게 사용할 수 있도록 만들어진 것이 `Relay`라고 볼 수 있습니다.


