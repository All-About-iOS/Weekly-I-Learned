### 개요
- Library는 무엇이고 Framework는 무엇일까요? 둘의 차이점과 쓰임새를 알아봅시다

### 들어가기 전 알아두면 좋을 용어들
- Bundle 
    - 디스크의 번들 디렉터리에 저장된 코드 및 리소스의 표현.
    - 애플은 번들을 사용해 앱, 프레임워크, 플러그인 등의 특정한 콘텐트를 표현한다.
    - info.plist, assets, string 파일 등을 가진다. 
    - 번들은 그것이 가지는 리소스를 잘 정의된 하위 디렉토리로 구성하며, 이 구조는 플랫폼과 번들 타입에 따라 다르다.
    - 번들은 대부분 이러한 기본 구조를 숨긴다. 번들 객체를 사용하면 번들의 구조를 몰라도 번들의 리소스에 접근할 수 있다. 
    - 따라서 코드와 리소스를 캡슐화한 후 우리가 사용하기 편하게 제공하는 역할을 한다. 

- 모듈
    - 모듈은 Swift가 코드를 조직화하는 방법
    - 코드 배포의 단일 단위 (배포 가능한 가장 작은 독립적인 요소)
    - 단일 단위(unit)로 빌드 및 제공되는 프레임워크 또는 애플리케이션
    - Swift의 import 키워드를 사용해 다른 모듈을 import 가능
    - 각 모듈은 네임스페이스를 가지며 접근 제어자(access control)를 통해 코드의 어떤 부분이 모듈의 외부에서 사용될 수 있는지 제어한다. 
    - Xcode의 각 빌드 타겟(앱 번들이나 프레임워크 등)은 분리된 모듈로 취급된다. 

- link
    - 라이브러리 / 프레임워크와 앱의 소스 코드 파일을 병합하는 것

- Mach-O (Mach Object File Format)
    - Mach 기반의 커널 OS에서 사용하는 실행 파일, object 파일, 라이브러리 파일 포맷

- Xcode 빌드 과정
    - 전처리기
        - 컴파일할 소스 코드 결정
    - 컴파일러
        - 소스코드를 어셈블리어로 변경
    - 어셈블러
        - 어셈블리어를 object 파일로 변경 (기계어로 변경)
        - 애플에서는 Mach-O 파일 포맷을 사용
        - .o, .dylib, .a, .bundle 등의 파일 포맷들이 있음
    - 링커
        - 위에서 생성한 오브젝트 파일과 라이브러리들을 연결시켜 실행파일로 만드는 것

### Library vs Framework - 일반적인 의미
- 라이브러리
    - 호출할 수 있는 함수들의 모음
    - 각 호출은 작업을 수행하고 제어권을 클라이언트에게 넘겨준다.
    - ex: KingFisher, Alamofire 등
- 프레임워크
    - 더 많은 동작을 가지는 추상적인 디자인을 구현
    - 이를 사용하기 위해서는 서브클래싱 또는 자체 클래스를 연결하여 프레임워크의 여러 위치에 동작을 삽입하는 것이 필요
    - 프레임워크의 코드는 필요한 지점에서 우리의 코드를 호출
    - ex: UIKit, SwiftUI 등

### Library vs Framework - iOS 개발에서의 의미
- 라이브러리
    - 특정 기능을 수행하는 코드의 집합

- static library / static archive libraries / static linked shared libraries
    - 대부분의 앱 기능은 실행 코드 라이브러리들에서 구현된다. 앱이 static linker를 사용해서 라이브러리와 연결되면 앱이 사용하는 코드는 생성된 실행 파일에 복사된다.
    - static linker는 컴파일된 소스 코드와 라이브러리 코드를 하나의 실행 파일로 모아, 런타임에 전체 코드들이 메모리에 올라가도록 한다. 
    - 이렇게 앱의 실행 파일의 일부분이 되는 라이브러리가 static library이다.
    - 이들은 객체 파일의 모음 또는 아카이브이다. 
    - *.a 확장자를 가진다. 
![Address Space 1](https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/de810f2a-7e2c-44cf-93ee-cf8a94bf88cb)


- static library 예시와 문제점
    - 앱이 실행되면 앱의 코드(연결된 정적 라이브러리의 코드가 포함되어있는)는 앱의 메모리 주소 공간으로 불러와진다. 많은 정적 라이브러리를 앱에 연결하면 실행 파일의 크기를 너무 크게 만든다. 
    - 큰 실행 파일을 가진 앱은 실행이 느리고 메모리를 많이 차지한다. 
    - 또한 정적 라이브러리가 업데이트되었을 때 앱도 함께 업데이트해 주어야 업데이트된 라이브러리를 사용 가능하다는 문제도 있다.
    - 앱이 코드를 메모리의 주소 공간으로 불러오는 더 좋은 방법은, 앱 실행 순간이나 런타임 중 실제로 필요한 순간에 코드를 불러오는 것이다.
    - 이러한 유연성을 제공하는 라이브러리가 바로 dynamic library이다. 

- dynamic library / dynamic shared libraries / shared objects / dynamically linked libraries
    - 동적 라이브러리는 실행 파일의 일부분이 되지 않는다. 앱 실행 또는 런타임에 앱으로 불러와질 수 있다.
    - 동적 라이브러리를 사용하면 라이브러리에 대한 업데이트를 자동적으로 앱에 반영할 수 있다.
    - 정적 라이브러리에 비해 실행 파일을 작게 유지할 수 있다.
    - 동적 라이브러리의 업데이트가 앱에 자동적으로 반영되기 때문에 그에 관한 대비가 필요하다. 
    - *.dylib 확장자를 가진다.
![Address Space 2x](https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/8c12e914-8fdc-454e-a8da-ca9d8b18aeec)

- 프레임워크
    - 프레임워크는 동적 라이브러리, nib 파일, 이미지 파일, localized 문자열, 헤더 파일 및 참조 문서와 같은 공유 리소스를 단일 패키지로 캡슐화하는 계층적 디렉터리이다.
    - 여러 앱이 이 모든 리소스를 동시에 사용 가능하다. 시스템은 필요에 따라 리소스를 메모리에 로드하고, 가능할 경우 하나의 리소스 사본을 공유해 사용한다. 
    - 프레임워크는 번들이다.
    - 따라서 Core Foundation Bundle Service 또는 Cocoa NSBundle 클래스를 통해 접근 가능하다.
    - 다른 번들과는 달리 finder에서 opaque file로 존재하지는 않는다. 
    - 프레임워크 번들은 사용자가 탐색할 수 있는 표준 디렉토리이며, 개발자는 프레임워크 내용을 쉽게 브라우징하며 문서와 헤더파일을 볼 수 있다.

- 라이브러리와 비교한 프레임워크의 장점
    - 프레임워크는 앱이 특정 작업을 수행하기 위해 호출할 수있는 루틴 라이브러리(library of routines)를 제공한다는, 정적 및 동적 라이브러리와 동일한 기능을 제공한다. 
    - 그러나 프레임워크는 정적 및 동적 라이브러리에 비해 다음과 같은 이점을 제공한다.
        - 프레임워크는 서로 관련이 있지만 분리되어있는 리소스를 그룹화하여 해당 리소스를 더 쉽게 설치, 제거, 및 탐색하도록 한다.
        - 프레임워크는 라이브러리보다 더 다양한 리소스 타입들이 포함될 수 있다. 예를 들어 모든 헤더 관련 파일과 문서가 포함될 수 있다.
        - 프레임워크의 여러 버전을 동일한 번들에 포함하여 이전 프로그램과의 backword 호환이 가능하다. 
        - 프레임워크의 읽기 전용 리소스는 해당 리소스를 사용하는 프로세스 수에 관계없이 한 번에 하나의 복사본만 물리적으로 메모리에 상주한다. 이러한 리소스 공유는 시스템의 메모리 사용 공간을 줄이고 성능을 개선하는 데 도움이 된다. 


- static framework vs dynamic framework
    - static framework는 메인 바이너리가 static archive인 프레임워크.
    - 여기서 메인 바이너리란 앱 실행을 위한 executable (Mach-O 포맷)을 의미한다. 
    - 여기서부터는 추측
        - static archive란 static library가 컴파일되어 하나의 파일로 패키징된 것. ex) .a 확장자 파일
        - 따라서 static framework는 소스 코드를 static library처럼 정적으로 패키징하여 제공하는 프레임워크이며
        - dynamic framework는 그 반대이다.
    - static framework와 dynamic framework도 정적 및 동적 라이브러리와 같은 특성과 장단점을 가진다.

### 참고
- [Documentation Archive - Dynamic Library](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)
- [Documentation Archive - Framework](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html#//apple_ref/doc/uid/20002303-BBCEIJFI)
