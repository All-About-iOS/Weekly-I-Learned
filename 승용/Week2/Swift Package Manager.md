### 개요
- Swift Package Manager는 무엇이고, 어떻게 이루어져 있는지 알아봅시다.

### SPM이란?
- Swift 코드 배포를 관리하는 도구
- Swift 빌드 시스템과 통합되어 다운로드, 컴파일, 의존성 연결 프로세스를 자동화한다.
- Swift 3.0 이상부터 지원

### Modules
- 모듈은 Swift가 코드를 조직화하는 방법
- 코드 배포의 단일 단위 (배포 가능한 가장 작은 독립적인 요소)
- 단일 단위(unit)로 빌드 및 제공되는 프레임워크 또는 애플리케이션
- Swift의 import 키워드를 사용해 다른 모듈을 import 가능
- 각 모듈은 네임스페이스를 가지며 접근 제어자(access control)를 통해 코드의 어떤 부분이 모듈의 외부에서 사용될 수 있는지 제어한다. 
- Xcode의 각 빌드 타겟(앱 번들이나 프레임워크 등)은 분리된 모듈로 취급된다. 
- 특정 문제를 해결하는 코드를 모듈화하면 해당 코드를 다른 상황에서 재사용할 수 있다.

### Packages
- Package는 Swift 소스 파일과 매니페스트 파일(manifest file)로 구성된다.
- Package.swift 라는 매니페스트 파일은 PackageDescription 모듈을 사용해 패키지의 이름과 내용을 정의한다.
- Package는 하나 이상의 타깃을 가지며, 각 타깃은 product를 지정하고 하나 이상의 종속선을 선언할 수 있다. 

> [!Note]
> 메니페스트 파일이란 소프트웨어 시스템 내에서 필수적인 정보를 담고 있는 메타데이터 파일을 의미한다.

여기까지 확인하고, 실제 Swift Packge의 파일 구조와 Package.swift의 구조를 살펴보자. 

<img width="316" alt="Screenshot 2024-06-08 at 6 40 03 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/cdc184c5-7239-4923-a2c7-cbe64e935900">

```swift
// swift-tools-version: 5.10
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "Network",
    platforms: [.iOS(.v14)],
    products: [
        // Products define the executables and libraries a package produces, making them visible to other packages.
        .library(
            name: "Network",
            targets: ["Network"]),
    ],
    dependencies: [
        .package(path: "../Core")
    ],
    targets: [
        // Targets are the basic building blocks of a package, defining a module or a test suite.
        // Targets can depend on other targets in this package and products from dependencies.
        .target(
            name: "Network",
            dependencies: [
                .product(name: "Core", package: "Core")
            ]
        ),
        .testTarget(
            name: "NetworkTests",
            dependencies: [
                "Network",
                .product(name: "Core", package: "Core")
            ]
        ),
    ]
```

- Network라는 Package는 Package.swift라는 메니페스트 파일을 가지며, 소스 코드와 테스트를 가진다.
- Package.swift의 내용을 뜯어보자.

### swift-tooles-version
- 패키지를 빌드하기 위한 최소 Swift 버전

### import PackageDescription
- 위에서 살펴봤듯, Package.swift 메니페스트 파일은 PackageDescription 모듈을 import해 패키지의 이름과 내용을 정의한다.

### name
- 패키지 이름

### platforms
- 지원 플랫폼과 최소 OS 버전

### products
- 타깃은 라이브러리 또는 실행 파일(executable file)을 product로 빌드할 수 있다.
- 라이브러리에는 다른 Swift 코드에서 import할 수 있는 모듈이 포함되어 있다.
- 실행 파일은 운영 체제에서 실행할 수 있는 파일이다. ex) main.swift를 포함시키면 macOS에서의 실행 파일로 빌드 가능
- 위 예시는 다른 Swift 코드에서 import할 수 있는 라이브러리를 제공하고 있다.

### dependencies
- 패키지의 코드에 필요한 모듈.
- dependency는 패키지 소스에 대한 상대 또는 절대 URL 및 사용할 수 있는 패키지 버전에 대한 요구사항 집합으로 구성된다.
- 위 예시는 같은 계층의 폴더 안에 존재하는 로컬 모듈에 대한 URL을 작성하고 있다.
- 외부 모듈을 사용하고 싶다면 아래와 같이 작성할 수 있다.
```swift
   .package(url: "https://github.com/apple/swift-argument-parser.git",
                 from: "0.4.4"),
```
- Swift Package Manager의 역할은 프로젝트의 모든 의존성을 다운로드하고 빌드하는 프로세스를 자동화하여 이러한 조정 비용을 줄이는 것이다.
- 이는 재귀적인 프로세스로, A->B->C->D의 의존 관계가 있다면 이러한 의존 그래프를 충족하기 위한 모든 모듈을 다운로드 및 빌드한다.

### targets
- 모듈 또는 테스트 스위트(test suite)가 되는 패키지의 기본적인 빌딩 블록
- 하나의 패키지가 여러 타겟을 가질 수 있다.
<img width="321" alt="Screenshot 2024-06-08 at 6 52 56 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/7629c62c-5c53-4b2e-b088-d81047df6e4c">

- Sources, Tests 바로 아래에 위치한 폴더가 타겟 단위이다.
- 위 예시를 확인하면 타겟 name과 각 폴더 이름이 일치하는 것을 확인할 수 있다.
- 이러한 폴더를 추가해 다른 타겟을 생성할 수 있다.
- 이러한 타겟은 이 패키지의 다른 타겟을 의존할 수 있고, 패키지가 의존하는 product에도 의존할 수 있다.
- 위에서 보았듯이 각 타깃은 product로 빌드 가능하다. 따라서 여러 개의 product가 존재할 수 있다. 

### target dependencies
- 위의 dependencies는 Package 전체에서 사용하는 의존성이고, 타겟의 dependencies는 타겟이 사용하는 의존성이다.

### package 접근제어자
- Package의 외부에서 Package의 코드를 접근하기 위해서는 Package 내부의 접근제어자를 를 public으로 설정한다.
- 그러나 패키지 외부에는 코드를 공유하지 않고, 패키지 내의 모듈 간에만 코드를 공유하고 싶을 경우 package 접근제어자를 사용한다.
- Swift 5.9에서 새로 나온 접근제어자!
- 예시
```swift
// Module Engine in GamePackage
public struct MainEngine {
    public init() { ...  }
    // Intended to be public
    public var stats: String { ...  }
    // A helper function made public only to be accessed by Game
    public func run() { ...  }
}

// Module Game in GamePackage
import Engine

public func play() {
    MainEngine().run() // Can access `run` as intended since it's within the same package
}

// client App in AppPackage
import Game
import Engine

let engine = MainEngine()
engine.run() // Can access `run` from App even if it's not an intended behavior
Game.play()
print(engine.stats) // Can access `stats` as intended
```
- 위 예시에서 Game 모듈에서 사용할 수 있게 하기 위해 public 접근제어자로 엔진의 run 함수를 선언했지만, 엔진의 run 함수에 대해서는 몰라야 하는 앱에서도 엔진의 run 함수를 실행할 수 있게 되었다.
- 이를 package 접근제어자로 변경하면 패키지 내부에서만 사용 가능하고, GamePackage 바깥인 앱에서는 사용할 수 없게 할 수 있다.
