# Alamofire 스터디 정리

> [!NOTE]
> Alamofire를 학습하며 스터디 중 논의한 내용을 정리한 문서입니다.

### Alamofire의 등장배경

Swift에서는 기본적으로 URLSession이라는 HTTP 통신을 위한 클래스를 제공합니다.
하지만 URLSession은 아래와 같은 단점들이 존재합니다.

- 요청(Request)을 구성하는 코드가 장황하고, 가독성이 떨어진다.
- 반복되는 코드가 발생한다.(e.g. 헤더 설정, 에러 핸들링, JSON 디코딩)
- 네트워크 요청을 디버깅하거나 로그를 남기려면 기존 코드 수정이 많이 필요하다.

이처럼 URLSession은 `URLRequest`를 직접 만들고 `URLSessionDataTask`를 관리하는 등 번거로운 작업을 필요로하는 low-level API입니다.

예를 들어 간단한 GET 요청을 한다고 했을때, URLSession을 사용하면 통상적으로 아래와 같은 코드를 작성하게 됩니다.

```swift
guard let url = URL(string: "<https://api.example.com/user>") else { return }

var request = URLRequest(url: url)
request.httpMethod = "GET"
request.addValue("application/json", forHTTPHeaderField: "Content-Type")

let task = URLSession.shared.dataTask(with: request) { data, response, error in
    if let error = error {
        print("요청 실패: \\(error)")
        return
    }

    guard let data = data else { return }

    do {
        let user = try JSONDecoder().decode(User.self, from: data)
        print("유저 정보: \\(user)")
    } catch {
        print("디코딩 실패: \\(error)")
    }
}

task.resume()
```

하지만 Alamofire를 사용하게 되면 아래와 같이 간결하고 가독성이 좋아진 코드를 작성할 수 있게 됩니다.

```swift
import Alamofire

AF.request("<https://api.example.com/user>", method: .get)
    .validate() // default: 200 ~ 299
    .responseDecodable(of: User.self) { response in
        switch response.result {
        case .success(let user):
            print("유저 정보: \\(user)")
        case .failure(let error):
            print("요청 실패: \\(error)")
        }
    }
```

이렇게 Alamofire는 반복적이고 복잡하며 가독성이 좋지 않은 URLSession을 통한 네트워킹을 가독성이 보다 향상된 코드를 작성할 수 있도록 합니다.

가독성 측면 뿐만아니라, 인증, 재시도 등등 URLSession으로 직접 구현해야하는 기능들을 고수준으로 추상화하여 편리하게 여러 기능들을 활용할 수 있도록 하기위함도 있습니다.

### Alamofire는 왜 이렇게 디자인되었을까?

#### Swift 언어 철학을 반영한 설계

Swift는 안정성, 명확성, 간결함을 추구합니다. Alamofire는 이를 반영하여 다음과 같은 방식으로 디자인 되어있습니다.

- 클로저 기반 비동기 처리 → delegate보다 간결하고 읽기 쉬움
- enum과 struct 사용 → 타입 안정성, 명확한 상태 표현
- 체이닝 방식의 API → 요청 흐름이 한 눈에 들어옴

#### 현실적인 사용자의 요구를 충족

네트워크 통신은 단순 요청을 넘어 인증 토큰 자동 주입, 에러 핸들링 큐칙 통일, JSON 디코딩 등의 상황들이 요구됩니다. Alamofire는 이러한 사용자들의 니즈를 고려해 미리 모듈화된 컴포넌트를 설계했습니다.

e.g. `RequestInterceptor`, `EventMonitor`, `ParameterEncoding` 등

#### 유지보수성과 테스트 용이성 고려

Alamofire 내부의 모든 객체들은 요청은 `Request`, 세션은 `Session`, 인코딩은 `ParameterEncoding`, 응답 처리는 `ResponseSerializer`들과 같이 각자 명확한 역할과 책임 단위로 분리되있습니다.

이러한 구조 덕분에 확장에 유리하며 테스트 가능한 코드가 되었습니다.