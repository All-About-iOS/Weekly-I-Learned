### Data?
- Data는 UI를 구동하는 모든 정보
- 다양한 형태와 구조를 가질 수 있음

### SwiftUI의 데이터 흐름 원칙 두 가지
- 뷰가 데이터를 읽을 때 마다 해당 뷰에 대한 의존성을 생성한다.
- 뷰 계층에서 읽어들이는 모든 데이터들은 Source of Truth를 가진다.
    - SoT는 뷰 계층구조 내부에 위치할 수도 있고 외부에 존재할 수도 있다.
    - SoT의 위치에 상관없이 항상 하나의 SoT만을 유지해야 한다. 
<img width="736" alt="Screenshot 2024-06-15 at 8 48 52 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/9db43b68-a721-495c-b458-39bc44e7bf38">
<img width="725" alt="Screenshot 2024-06-15 at 8 49 06 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/38fd59ee-e71d-4a63-98c8-300495379d2d">

### 일반적인 프로퍼티
- SoT로부터 파생된 데이터를 읽는 뷰를 만들 때 좋은 도구이다.
```swift
struct PlayerView: View {
	let episode: Episode // 요놈이 regular property

	var body: some View {
		VStack {
			Text(episode.title)
			Text(episode.showTitle).font(.caption).foregroundColor(.gray)
		}
	}
}

PlayerView(episode: episode) // 이런식으로 상위 뷰로부터 파생 데이터를 내려받아 사용
```

### @State
- 시간이 지남에 따라 변경될 수 있고, 뷰가 그것에 의존하여 값의 변경에 따라 뷰도 재렌더링되는 값
- 모든 @State는 Source of Truth 이다.
- 일반적인 프로퍼티를 사용해 직접적으로 뷰 계층구조를 변경시키면 안 됨. (컴파일 에러 발생)
- private으로 선언해 해당 뷰에 의해 소유되고 관리됨을 명시적으로 표현하는 것이 좋음

### @State의 동작 방식
- SwiftUI 프레임워크는 @State로 선언된 프로퍼티를 위한 지속적인 저장소를 할당하고, 이를 의존성으로 추적
- 시스템이 저장소를 대신 생성해주기 때문에 항상 초기 상수 값을 지정해야 한다.
```swift
struct PlayerView: View {
	let episode: Episode
	@State private var isPlaying: Bool = false

	var body: some View {
		VStack {
			Text(episode.title)
			Text(episode.showTitle).font(.caption).foregroundColor(.gray)

			Button(action: {
				self.isPlaying.toggle()
			}) {
				Image(systemName: isPlaying ? "pause.circle" : "play.circle")
			}
		}
	}
}
```
<img width="720" alt="Screenshot 2024-06-15 at 9 13 52 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/0d81b906-e95c-463e-aae5-0e60d4737a70">
<img width="723" alt="Screenshot 2024-06-15 at 9 14 10 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/52699b65-3688-462d-97b0-ab73d9e290bc">
<img width="716" alt="Screenshot 2024-06-15 at 9 14 25 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/bc9e1b41-7418-4355-8a92-8c073c835899">



- @State 프로퍼티는 SwiftUI 프레임워크가 관리하는 지속적인 저장소에 저장된다.
- SwiftUI는 @State 프로퍼티의 변경을 관찰(observe)할 수 있다.
- SwiftUI는 @State 변수가 body를 작성할 때 사용된다는 것을 알기 때문에, 뷰 렌더링이 해당 @State 변수에 의존하고 있다는 것을 알 수 있다.
- 만약 유저가 버튼과 상호작용하면 프레임워크는 action을 실행하고 상태의 변경이 일어난다.
- 런타임에 데이터, 즉 @State가 변경되면 해당 @State를 소유하는 뷰를 검증(validate)하기 시작한다. 이는 해당 뷰와 그 하위 뷰의 body를 다시 계산한다는 것을 의미한다. 
	- 따라서 모든 변경 사항은 항상 뷰 계층구조를 통해 아래로 흐른다.
	- 프레임워크는 뷰를 비교하고 변경된 뷰만 재렌더링한다. 

### 모든 @State는 SoT!
모든 @State는 뷰가 소유하는 Source of Truth이다.
- SwiftUI의 뷰는 이벤트의 연속이 아닌, @State의 함수이다. 
	- 옛날엔(UIKit) alpha값 변경하기, subview 추가하거나 제거하기 등의 뷰 계층을 직접적으로 변경하는 이벤트들을 사용해 뷰를 관리했다.
	- 그러나 SwiftUI는 @State를 변경하고, 그 @State가 SoT로 작동하며, 이를 바탕으로 뷰를 계산해낸다. 


### 뷰 업데이트 흐름
- 앱은 유저와 디바이스간의 지속적인 피드백 루프로 생각할 수 있다.
- 사용자가 앱과 상호작용하여 어떤 액션을 생성한다.
- 이 액션은 프레임워크에 의해 실행되며, 어떤 @State를 변경한다.
- 시스템은 @State의 변경을 감지하고, 해당 @State에 의존하는 뷰를 업데이트한다.
- 데이터가 항상 한 방향으로 흐르고, @State가 모든 변경의 최종 지점이 되어 뷰 업데이트를 예측 가능하고 이해하기 쉽게 만든다. 
<img width="732" alt="Screenshot 2024-06-15 at 9 28 49 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/6d907ff1-a457-4c31-8e1d-ccf6cb6e15bd">

### @Binding
- 소유권 없이(Source of Truth를 가지지 않고) 값을 읽거나 쓸 수 있다.
- @State로부터 파생(derive)될 수 있다.

### @Binding 예시
```swift
struct PlayerView: View {
	let episode: Episode
	@State private var isPlaying: Bool = false // Source of Truth

	var body: some View {
		VStack {
			Text(episode.title)
			Text(episode.showTitle).font(.caption).foregroundColor(.gray)

			// $ prefix로 binding 값 파생시키기
			// $도 프로퍼티 래퍼의 기능 중 하나이다.
			PlayButton(isPalying: $isPlaying)
		}
	}
}

struct PlayButton: View {
	@Binding var isPlaying: Bool // Derived from @State

	var body: some View {
		Button(action: {
				self.isPlaying.toggle()
			}) {
				Image(systemName: isPlaying ? "pause.circle" : "play.circle")
		}
	}
}
```
- PlayButton을 독립된 뷰로 분리하였다.
- isPlaying 프로퍼티를 @State로 선언하게 되면 PalyerView, PalyerButton 두 개의 뷰에서 각자의 SoT를 가지고 관리하게 된다. 그러나 그럴 필요는 없음
- @Binding을 사용해 Source of Truth를 소유하지 않고, SoT에 대한 명시적인 의존성을 정의할 수 있다. 
-  @Binding은 @State로부터 파생될 수 있기 때문에 초기값이 필요없다.

### @State와 @Binding 구조가 가져오는 이점
<img width="573" alt="Screenshot 2024-06-15 at 9 42 30 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/61064242-24f2-4413-963f-3878a657aea8">


- 기존 UIKit에서 위와 같은 PlayerView 뷰를 작성하기 위해서는 뷰 컨트롤러가 데이터 동기화 작업 및 뷰 관리 작업을 모두 담당하고, delegate와 target action 패턴을 사용해 유저 액션에 대응해야 했다.
	- 담당하는 작업이 많고 상태 관리가 어려움
- 또한 뷰 컨트롤러가 모델 변경을 관찰하고 변경 이벤트에 대응하고, 값이 변경될 때 마다 값을 읽고 필요한 곳에 할당해줘야 한다. 
- 앱이 커질수록 이러한 복잡도가 증가한다.

<img width="388" alt="Screenshot 2024-06-15 at 9 46 09 PM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/4e5315a5-1440-4fe7-8224-c5a9891b139e">

- 그러나 SwiftUI에서는 @State와 @Binding을 사용해 데이터 의존성을 정의하기만 해주면 나머지는 프레임워크가 알아서 다 해준다. 
- 프레임워크 전체에 적용되는 강력한 아이디어!
	- Toggle, TextField, Slider 등을 살펴보면 @Binding을 파라미터로 받는다.
	- 값을 복사하거나 동기화할 필요 없이 Source of Truth에 대한 참조만을 전달하면 데이터 관리가 끝나는 것
