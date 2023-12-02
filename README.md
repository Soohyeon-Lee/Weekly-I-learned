# Weekly-I-learned
매일 쓰긴 귀찮아서 주말(매주...? 격주...?)마다 끄적이는 노트

## 2023

### 2023.12

<details>
  <summary>2023.12.04 ~ 2023.12.08</summary>
  
  - TODO
    
</details>

### 2023.11

<details>
  <summary>2023.11.27 ~ 2023.12.01</summary>
    
  - CI/CD 로컬 빌드머신 이사 준비
    - Intel 기반 Mac mini에서 Apple Silicon 기반 Mac mini로 이사 예정, CI/CD 정상 작동을 위한 작업 및 테스트
    - Xcode 12 이후로 x86_64와 arm64 모두를 지원하게 되면서 Apple Silicon 기반 맥북에서 빌드 과정에서 오류가 생겼었다.
    - 이를 해결하기 위해 시뮬레이터 빌드 과정에서 arm64를 제외하였었다.
      - `Excluded Architectures` -> `Build Configuration` -> `Any iOS Simulator` -> `arm64`
    - 이제는 반대로 개발과정(로컬 빌드, CI 빌드)에서는 Apple Silicon 기반 시뮬레이터 빌드만 검증하기 때문에 아래와 같이 변경하였다.
      - `Excluded Architectures` -> `Build Configuration` -> `Any iOS Simulator` -> `x86_64`
    - [reference](https://ios-development.tistory.com/1212)
  
  - `Task`와 `[weak self]`
    - `[weak self]`
      - `@escaping`이 명시된 closure에서 self에 접근하려면 capture가 필요하다.
      - self의 프로퍼티에 저장된 `@escaping` closure 안에서 self를 캡쳐할 경우, retain cycle이 일어날 가능성이 있다. -> `[weak self]` 필요
      - closure를 소유하는 대상이 메모리에 올라가지 않는 경우(자기 자신에 대한 reference count가 증가하지 않는 경우)에는 reference count가 증가하지 않기 때문에 retain cycle이 일어나지 않는다. -> `[weak self]` 필요 없음
        - ex) UIView.animate(withDuration:animations:completion:)
        - ex) GCD 호출 (DispatchQueue)
        - 다만, closure의 실행이 무조건 담보되어야하는 경우 (ViewModel 안에서의 로직을 통한 데이터 동기화, 무결성 확보 등), closure 실행 시점 이전에 self가 deinit되는 것을 막기 위해서 `[weak self]` + `guard let self ~` 구문을 통해 self의 deinit 시점을 closure 실행 이후로 미룰 수 있다.
    - `Task`
      - `Task`는 기본적으로 init 함수에서 받는 파라미터인 operation이 @escaping이다.
      - 하지만 `@_implicitSelfCapture`가 명시되어있어 별도의 캡처 없이 self에 접근할 수 있다.
      - 또한 `Task`는 실행 직후 메모리에서 해제되기 때문에 retain cycle 위험이 없다.
      - ```swift
        // reference : https://github.com/apple/swift/blob/main/stdlib/public/Concurrency/Task.swift
        
        @discardableResult
        @_alwaysEmitIntoClient
        public init(
          priority: TaskPriority? = nil,
          @_inheritActorContext @_implicitSelfCapture operation: __owned @Sendable @escaping () async -> Success
        )
  
        Task {
          self.function()
          self.canAccessProperty = true
        }
        ```
      - 하지만 장시간이 소요되는, 중간에 cancel해도 되는 Task의 누적으로 인한 메모리 누수를 막기 위해서 `[weak self]`를 통해 self가 deinit되는 시점에 Task를 릴리즈해버린다.
        - ex) cell에 image를 async하게 로드하는 경우

</details>
