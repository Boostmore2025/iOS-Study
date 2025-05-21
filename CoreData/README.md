# Core Data 스터디 정리

> [!NOTE]
> Core Data를 학습하며 스터디 중 논의한 내용을 정리한 문서입니다.

### Core Data의 등장배경
iOS 앱에서 데이터를 저장하는 방식으로 `UserDefaults`, `FileManager`, `Keychain`, `SQLite` 등이 있었지만, 
- 복잡한 데이터 모델링
- 객체 간의 관계 관리
- 조건을 이용한 검색    
등의 요구사항을 만족시키기에는 한계가 있었습니다.

Core Data는 이러한 문제를 해결하기 위해 등장했습니다.
- 객체 지향 방식으로 데이터를 다루고
- 관계형 데이터를 추상화하여 더 쉽게 관리할 수 있으며
- 변경 사항을 자동으로 추적하고
- 성능 최적화를 고려한 쿼리 실행이 가능합니다.

Core Data는 복잡한 데이터 구조를 가진 앱에서도 **일관된 방식으로 데이터를 관리**할 수 있게 해줍니다.


### Core Data는 왜 이렇게 디자인 되었을까?

Core Data는 단순 데이터베이스 래퍼가 아니라, 앱 내부의 객테 상태와 생명 주기를 관리하는 **객체 상태 관리(Object Graph Management) 프레임워크**입니다.

- 객체를 메모리에서 효율적으로 관리하고
- 필요한 시점에만 디스크에 저장해 **성능을 최적화** 하며
- MVC 패턴과의 통합을 고려해 **모델 계층의 복잡도를 줄이고 재사용성을 높이기 위해 설계**되었습니다.

SQLite 같은 기본 저장소를 내부에서 사용하지만, 개발자는 이를 직접 다루지 않고 고수준의 API를 통해 **객체 단위로 데이터를 다루는 추상화된 방식**으로 접근할 수 있습니다.


### 멀티스레드 환경에서의 Core Data

Core Data의 데이터 작업은 `NSManagedObjectContext`를 통해 수행됩니다. 

이 컨텍스트는 기본적으로 메인 스레드에서 실행되므로, 대량의 데이터 작업은 백그라운드에서 처리하지 않으면 성능 저하가 발생할 수 있습니다.

#### `NSManagedObjectContextConcurrencyType`
Core Data는 스레드 별 작업을 위해 컨텍스트 생성 시 다음의 동시성 타입을 제공합니다.
- `.mainQueue`: 메인 스레드에서 실행
- `.privateQueue`: 백그라운드 스레드에서 실행
  
#### `perform`, `performAndWait`
- `perform`: 비동기 실행
- `performAndWait`: 동기 실행

#### MergePolicy
여러 컨텍스트가 동일 객체를 수정할 경우 **충돌(NSMergeConflict)**이 발생할 수 있습니다.     

이를 해결하기 위해 mergePolicy를 설정할 수 있습니다.
- `.error`: 기본값, 충돌 시 저장 실패
- `.mergeByPropertyObjectTrump`: 현재 컨텍스트 값 우선 (충돌된 속성만)
- `.mergeByPropertyStoreTrump`: DB 저장소 값 우선 
- `.overwrite`: 현재 컨텍스트 값으로 전체 덮어쓰기
- `.rollback`: 현재 변경사항 모두 되돌리기

--------

### Q. NSManagedObjectContext는 어떤 역할을 하며, 왜 중요한가요?

A. `NSManagedObjectContext`는 Core Data에서 **데이터의 변경 사항을 추적**하고 실제 저장소와의 트랜잭션을 처리하는 핵심 객체입니다.    

데이터 추가, 수정, 삭제 등과 같은 작업은 이 컨텍스트를 통해서만 수행되며 `save()`를 호출하여 변경 내용을 저장합니다.

NSManagedObjectContext가 중요한 이유는, **멀티스레딩 환경에서 데이터 일관성을 유지하면서 안전하게 작업을 처리할 수 있도록 도와주기 때문**입니다.

다수의 컨텍스트를 통해 변경사항을 처리할 때, 변경사항을 반영할 컨텍스트를 정할 수 있으므로 일관성 있는 정책으로 데이터 충돌을 방지하고 안정적인 데이터 저장을 도와줍니다.


### Q. Core Data에서 FetchRequest와 Predicate의 역할은 무엇인가요?

A. `FetchRequest`는 Entity 조회를 요청하는 객체이고, `Predicate`는 Entity 조회 시 조건을 추가하는 필터링 객체입니다.

즉, `FetchRequest`는 "무엇을 가져올지"를 정의하고, `Predicate`는 "어떤 조건으로 필터링 할지"를 지정합니다.


### Q. Core Data와 SQLite의 차이는 무엇인가요?

A. SQLite는 데이터베이스로, SQL 기반의 CRUD 및 트랜잭션 기능을 제공하지만 멀티스레드 환경에서 안전하지 않습니다.

Core Data는 SQLite를 기반으로, 로컬 데이터베이스 역할 뿐만 아니라 멀티스레딩 기능, 객체 상태 추적, iCloud 연동 등 다양한 기능을 제공하는 프레임워크로, 어플리케이션의 모델 계층으로 활용할 수 있습니다.

