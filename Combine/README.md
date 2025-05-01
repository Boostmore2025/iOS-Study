# Combine 스터디 정리

> [!NOTE]
> Combine을 학습하며 스터디 중 논의한 내용을 정리한 문서입니다.

### Combine의 등장배경?
애플은 비동기 이벤트 핸들링 퍼스트 파티는 클로저 기반의 API를 제공해왔습니다.

- Target/Action
- NotificationCenter
- URLSession
- KVO (class only)

클로저 기반의 비동기 처리는 코드의 가시성을 떨어트리며, 디버깅을 어렵게 만듭니다. 또한 안정적인 동시성 프로그래밍을 목표로 변화하는 애플은 class사용을 지향하고 있습니다. 
이러한 환경에서 Combine의 등장은 퍼스트 파티의 장점인 높은 성능, 값타입 기반으로 비동기적인 이벤트를 **안전히 처리할 수 있도록** 만들었습니다.
또한 **선언형 API**를 제공해 높은 코드 가시성과, 데이터 바인딩을 기본적으로 제공하는 SwiftUI의 활용성을 강화했습니다.

### 왜 이렇게(Back Pressure) 디자인 되었을까?

Subscriber와 메인 스레드가 처리할 수 있는 속도는 제한적입니다.
예를 들어 Publisher가 발행한 초당 1000개의 이벤트에 대해, 메인 스레드의 연산, 이벤트 처리와 UI갱신의 처리 속도는 이보다 늦어 **버벅임, 메모리 누수, 크래시** 등이 발생할 수 있습니다.
백프레셔 구조는 Subscriber가 요청한 만큼만 발행하므로 **리소스를 효율적으로 사용**할 수 있게 됩니다. Combine에서는 Subscription의 `request(_:)` 메서드로 Subscriber가 요구하는 이벤트의 발행량을 조절할 수 있도록 제공하고 있습니다.

### Q. Combine의 Publisher는 Hot Observable인가요? Cold Observable인가요?

A. Subject를 제외한 Publisher들은 Cold Observable로, 구독한 시점부터 값을 방출합니다.

Subject는 구독의 유무와 상관없이 값을 특정 스트림에 방출합니다. 따라서 구독하기 전에 send된 값들은 관찰할 수 없어 Hot Observable이라고 생각합니다.

### Q. CurrentValueSubject랑 PassthroughSubject의 차이가 무엇인가요?

A. CurrentValueSubject의 경우 초깃값이 존재하고, 직전에 방출된 값에 대한 버퍼를 가지고 있습니다. 이와 반대로 PassthroughSubject는 스트림에 값을 방출하고 그 값을 따로 저장해놓지 않기 때문에 한 번 방출된 값에 대한 이전 정보는 알 수 없다는 특징을 가지고 있습니다.
예컨대, PassthroughSubject는 일회성 이벤트 처리에 사용되고, CurrentValueSubject는 이전값에 대한 정보도 필요한 경우 사용할 수 있습니다.

### Q. AnyPublisher는 무엇이고 왜 사용되나요?

A. AnyPublisher는 Publisher를 래핑해서 타입을 지운 publisher입니다.

구체타입을 숨겨 내부 구현의 변경으로 인한 영향을 외부에 전파하지 않을 수 있기 때문에 사용합니다.

타입 이레이징을 하지 않은 경우, Publisher부터 오퍼레이터 연산과정이 타입으로 드러나게 되는데, 이는 외부에 내부 정보가 드러나게 되고, 타입이 구체적이기 때문에 Publisher나 연산과정이 변경될 경우 외부영향을 미칠 수 있습니다.

### Q. Combine과 RxSwift 차이는 무엇인가요?

A. Combine은 pub-sub 패턴, RxSwift는 Observer 패턴을 기반으로 하고있습니다.
Combine은 Backpressure를 Subscription.Demand로 제공하지만, RxSwift에는 해당 기능을 제공하지 않습니다.

RxSwift는 RxCocoa를 통해 UIKit 컴포넌트와의 연동성을 제공하지만, Combine은 그렇지 않고 제공되는 오퍼레이터의 갯수에서도 차이가 존재합니다. RxSwift의 경우 andThen이나 withUnretained 등 편리한 오퍼레이터들이 많지만, Combine의 경우 map, filter, reduce 와 같은 기본적인 오퍼레이터들만 제공합니다.

Combine의 경우 퍼블리셔 생성 시 에러타입을 지정해주어야 하는데, RxSwift에서 옵저버블은 에러타입을 요구하지 않습니다.