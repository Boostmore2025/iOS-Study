# SwiftUI 스터디 정리

> [!NOTE]
> SwiftUI를 학습하며 스터디 중 논의한 내용을 정리한 문서입니다.

### 📌 SwiftUI의 View가 구성되는 방식

```swift
struct ContentView: View {
	var body: some View {
		Text("Hello, world!")
			.padding()
	}
}
```

- `some` 키워드
    - some 키워드가 반환 타입 앞에 붙을 경우, 해당 반환 타입이 불투명한 타입(Opaque Type)이라는 것을 나타낸다.
    - 제네릭은 함수 외부에서 해당 타입을 알 수 있는 반면, 
    불투명한 타입은 외부에서 합수의 반환 값 유형을 정확하게 알 수 없지만, 함수 내부에서 어떤 타입을 다루는지 정확히 알고 있다.
- View 프로토콜
    - body라는 프로퍼티를 갖고 있으며 body는 contentView의 최상위 View 역할을 함 (연산 프로퍼티)
    - body는 **1개의 View만 리턴해야 함**
- View Modifier
    - ViewModifier 프로토콜
    - 모든 뷰가 View 프로토콜을 따르고, Modifier는 그 View를 감싸서 새로운 View를 리턴
    - `.font()`, `.padding()`, 등 체이닝 가능

<br>

### 📌 SwiftUI가 뷰를 구분하는 방법

![1](https://github.com/user-attachments/assets/2055eae1-f9be-436a-ac24-15500f661bfd)

SwiftUI는 정체성(Identity)을 통해 뷰를 구분한다.

UIView, NSView를 클래스로 모델링한 UIKit, AppKit과 달리 SwiftUI에선 View를 Struct로 취급하기 때문에 View를 식별하기 위해선 별도의 ID가 필요하다.

정체성은 명시적 정체성, 구조적 정체성 두 가지 방식으로 표현될 수 있으며, 명시적 정체성을 부여하지 않더라도 뷰 계층 구조를 활용해 뷰에 암시적인 ID를 부여한다.

<br>

1️⃣**명시적 정체성 (Explicit Identity)**

개발자가 ID 나 Identifiable 프로토콜을 사용하여 뷰나 데이터에 직접 고유한 식별자를 제공하는 방식이다.

ForEach와 같은 데이터 기반 컴포넌트에서 특히 중요하며, SwiftUI가 컬렉션의 항목을 효율적으로 식별하고 변경 사항에 따라 애니메이션을 적용할 수 있도록 한다.

UIKit의 포인터 정체성과 달리 SwiftUI 뷰는 값 타입이므로 별도의 명시적 식별자가 필요하다.

<br>

2️⃣**구조적 정체성 (Structural Identity)**

구조적 정체성(Structural identity)은 SwiftUI가 뷰의 유형(type)과 뷰 계층 구조 내에서의 위치(position) 를 기반으로 암시적인 정체성을 생성하는 방식이다.

```swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension ViewBuilder {

    public static func buildIf<Content>(_ content: Content?) -> Content? where Content : View

    public static func buildEither<TrueContent, FalseContent>(first: TrueContent) -> _ConditionalContent<TrueContent, FalseContent> where TrueContent : View, FalseContent : View

    public static func buildEither<TrueContent, FalseContent>(second: FalseContent) -> _ConditionalContent<TrueContent, FalseContent> where TrueContent : View, FalseContent : View
}
```

예를 들어, if 문과 같은 조건부 로직은 _ConditionalContent 라는 제네릭 뷰로 변환되며, 이 뷰는 참(true) 및 거짓(false) 내용에 대한 제네릭을 가진다.

이러한 변환은 ViewBuilder에 의해 이루어지며, 이를 통해 개발자가 명시적으로 식별자를 지정하지 않아도 SwiftUI가 뷰를 구별할 수 있도록 한다.

<br>

### **📌 Identify 를 사용할 때 주의할 점**

**1. 상태 초기화**

시간이 지남에 따라 뷰의 상태(state)가 변경되어도 뷰의 정체성(Identify)이 동일하다면, SwiftUI는 이를 동일한 뷰로 간주한다.

또한 뷰의 상태가 변경되더라도, 정체성은 유지된다.

하지만 View의 수명(Lifetime)은 정체성과 일치한다. 정체성이 변경되면 다른 View로 간주되고, 초기화 된다.

이 때 해당 View가 갖고있던 상태 또한 초기화되기 때문에 주의해야 한다.

**2. 애니메이션이 예상과 다를 수 있음**

동일한 뷰로 인식되어 부드러운 애니메이션 전환이 일어나야 할 곳에서 갑자기 사라지고 나타나는(fade in/out) 현상이 발생할 수 있다.

<br>

**💥 다른 View인데 identity가 같을 수 있을까?**

같을 수 있다.

두 View가 struct로 다르더라도 런타임에서는 "같은 위치에서 같은 View type이며 identity도 동일"하게 판단하면 기존 instance를 재사용한다.

예시: `Text("A")` → `Text("B")`로 내용은 바뀌었지만 *Text라는 View는 같은 위치에서 존재하므로 재사용* 가능

<br>

### 📌 뷰 연속성

SwiftUI의 핵심 컨셉 중 하나는 `"View는 값이고 body는 함수"`지만, *실제 화면을 그리는 데에는 상태와 identity를 통해 **continuity**(연속성)를 유지하려고 하려고 함.*

- SwiftUI는 매 변경마다 body를 다시 호출해 **새로운 View tree (값)를 생성**
- 하지만 실제로는 기존 화면을 **최소 변경**(diffing + reconciliation)하여 사용자 경험을 부드럽게 유지
- 이때 **identity가 같으면 상태/애니메이션/layout continuity 유지**, identity가 다르면 새로 만듦

이 **연속성**으로 identity를 사용해 뷰의 상태를 연속적으로 가져가서, 뷰를 재생성이 아닌 부분 변경해 사용자 경험을 높이면서도 성능상의 이점을 가져감

- List, NavigationStack, Animation, ForEach 등에서는 identity가 필수로 명시되는 경우가 많고
- 다른 View에서도 identity를 명시적으로 `.id(_:)`로 부여하면 **명확하게 어떤 View가 유지될지** 우리가 컨트롤할 수 있음

<br>

### 📌 뷰 재랜더링

identity가 달라졌다면 무조건 다시 그리고, 같다면 내용을 비교해 필요한 부분만 다시 그린다.

그러나 body함수는 계속 다시 호출되고, subview struct value는 바뀐다.

view의 dependency(view에 변경을 줄 수 있는 input 값)가 변경되면 SwiftUI는 렌더링을 위해 view body 프로퍼티를 호출하기 때문에 body 내부 모든 view가 재랜더링된다.

**예) Dynamic Property (프로토콜임)**

- 뷰의 외부 속성을 업데이트 하는 stored property의 인터페이스
- State, Binding, StateObject, ObservedObject 등이 채택하고 있음
- body를 그리기 전에 속성 값을 최신으로 유지하도록 도와줌
- 내부적으로 update() 함수가 자동 호출되어 최신 값으로 초기화 됨

<br>

### 🤔 배경색이 랜덤인 뷰

```swift
struct myView { ...

var body: some View {
    VStack {
		    MySecondView(name: $myModel.name) <- 얘의 바디가 재호출
		    
        Text("One")
            .background(Color.random)

        Text("Two")
            .background(Color.random)

        Text("Three \(counter)") // @State 의존
            .background(Color.random)

        Button("Increment") {
            counter += 1
        }
    }
}
```

`background(Color.random)` → 매번 body 실행 시 **Color.random 값이 새로 생성됨**

SwiftUI 입장에서는 background modifier가 이전과 다른 것으로 간주됨 → 해당 View는 **변화된 View**로 판단

→ 이로 인해 해당 subview에 대해:

- 기존 View instance reuse 불가
- background modifier가 달라졌으므로 새로운 View instance 적용
→ 배경색이 매번 바뀌는 것처럼 보임
- `background(Color.random)`는 *modifier chain 자체가 바뀌는 원인* → identity 유지 X
- 따라서 State가 하나만 바뀌어도 **세 subview 전부 다른 View로 판단됨 → 배경색 전부 갱신**

**따라서 identity는 같지만 diffing 결과가 다르고, 뷰를 다시 그림**

---

### 📌 SwiftUI의 레이아웃

뷰의 레이아웃을 만들려면 먼저 자식 뷰들의 계층을 구성한다.

그런 다음, 자식 뷰의 크기와 간격을 설정 파라미터와 frame, padding 같은 뷰 수정자를 통해 조정할 수 있다.

SwiftUI가 뷰 계층을 렌더링할 때, 각 자식 뷰를 재귀적으로 계산(evaluates)한다.

부모 뷰는 포함된 자식 뷰에 크기를 **제안(propose)** , 자식은 그에 따른 크기를 반환한다.

<br>

![2](https://github.com/user-attachments/assets/971a1d7e-6585-4169-96ae-2d08ec8f3fec)

<br>

### 📌 뷰 크기 제한하기

SwiftUI 내장 뷰들은 각기 다른 방식으로 크기를 처리한다.

- **부모가 제공하는 공간을 가득 채우는 뷰**(Expand to fill the space offered by their parent)
    - ex) Color, LinearGradient, Circle
- **콘텐츠에 따라 크기를 갖는 뷰**(ideal size that varies according to their contents)
    - ex) Text, HStack 그리고 container views
- **항상 고정된 크기를 갖는 뷰**(ideal size that never varies)
    - ex) Toggle, DatePicker

<br>

frame 수정자를 사용하면 뷰의 크기를 명시적으로 제한할 수 있다.

예를 들어 원의 너비를 40pt로 제한하려면 다음과 같이 작성한다.

```swift
struct MessageRow: View {
    let message: Message

    var body: some View {
        HStack {
            ZStack {
                Circle()
                    .fill(Color.yellow)
                Text(message.initials)
            }
            .frame(width: 40)

            Text(message.content)
        }
    }
}
```

SwiftUI는 frame 수정자를 적용하면 해당 뷰를 감싸는 **새로운 뷰**로 View Hierarchy에 추가한다.

![3](https://github.com/user-attachments/assets/2fe07572-8bfc-487c-8c7b-95758cf63dd2)
