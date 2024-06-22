
[![WWDC23 - Keep up with the keyboard](https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/621ef853-0733-4ebd-915b-1f92ec951790)](https://developer.apple.com/kr/videos/play/wwdc2023/10281/)

## Out of Process Keyboard

앱 프로세스에서 분리된 키보드 프로세스 사용

- Security 개선
- 사용자 입력 내용의 Privacy 보장
- 시스템 전체의 메모리 관리 효율 증가
    - 여러 앱에서 키보드 인스턴스가 생성되지 않고, 단 하나의 키보드만 사용하기 때문이다.! (싱글톤 같은 것인듯?)

## iOS 17 이전

- 키보드의 뷰와 로직이 앱 내에서 실행됨.
    
<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/40cc8224-6a46-43ca-b336-2f168daf2dea" width=300>
    

### 키보드 내부 동작 ( 동기적 실행 과정 )

1. 키보드 실행 시, 앱이 키보드를 요청한다
    - 사용자 터치에 응답하여 becomeFirstResponder를 수행하는 것과 같음.
2. UI Initialization
3. 애니메이션을 수행하여 키보드를 화면에 등장시킨다.
4. 화면 터치 또는 텍스트 입력을 할 때까지 별도의 동작을 하지 않음.

<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/482ed461-1477-4313-a800-75fb3020b5e7" width=500>

## iOS 17 이후

- 키보드가 자체 프로세스로 이동.
    - 기존에는 App 내에서 동작하기에, 해당 앱의 Main Thread에서 동작해야 했다면,
    - 별도의 프로세스로 동작하기 때문에, 키보드가 등장하는 동안 앱에서 별도의 동작 가능.
- 키보드가 앱 외부에서 실행되는 것과 유사.

<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/def81f04-7de5-4aba-ac2c-f13d66316f0b" width=400>


### 키보드 내부 동작

1. 키보드 실행 시, 앱이 키보드를 요청한다.
    - 키보드 프로세스는 비동기적으로 UI를 생성한다.
    - 그 동안, 앱은 가만히 있거나 사이드 작업을 수행한다.
- 준비가 완료되면, 키보드를 불러오고(bring up) 프로세스 간 애니메이션을 조정한다.
- 키보드가 나타나면, 키보드 범위 내에서 일어나는 터치 이벤트를 기다린다.
    - 터치 이벤트가 발생하면, 텍스트 삽입으로 전환한다.
    
<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/257d8a6c-7bf9-4852-8e36-0d04f140e27b" width=400>


## 새로운 Text입력 API

### Inline predictions ( 인라인 예측 ) - 영어만 지원.


<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/bb7d3b9c-a926-431a-80a7-887a12152c41" width=500>

- 텍스트 필드의 문맥 정보를 이용해 생성
- 적용 코드

```swift
// 내부 코드
@MainActor public protocol UITextInputTraits : NSObjectProtocol {
    // Controls whether inline text prediction is enabled or disabled during typing
    @available(iOS, introduced: 17.0)
    optional var inlinePredictionType: UITextInlinePredictionType { get set }
}

public enum UITextInlinePredictionType : Int, @unchecked Sendable {
    case `default` = 0
    case no = 1
    case yes = 2
}

// 실제 사용 코드
let textView = UITextView(frame: frame)
textView.inlinePredictionType = .yes
```
