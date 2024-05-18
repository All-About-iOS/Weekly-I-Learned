## 들어가며
- UIKit 앱 개발을 하며 UIScreen.main.bounds를 통해 디바이스 크기를 가져오고, window의 루트 뷰 컨트롤러를 가져오고, SceneDelegate에서 window의 루트 뷰컨을 넣어주는 등의 행위를 해 보았을 것이다.
- 그러나 사용한 컴포넌트들의 정확한 개념을 알지 못해 한 번 정리해 보고자 한다!

## iOS의 개념적 UI 구조

![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/a1b0c9af-8fd0-47c6-8edb-149f239297ee)

- UIScreen은 하드웨어 기반 디스플레이와 연관된 프로퍼티들을 정의하는 객체이다.
<img width="600" alt="Screenshot 2024-05-19 at 12 53 28 AM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/5f20ef3b-3650-427a-bb89-e7a7bf4054a2">

- UIWindowScene은 하나 이상의 window들을 관리하는 씬이다. 씬은 앱 UI의 인스턴스를 관리하는 객체이다. 
<img width="600" alt="Screenshot 2024-05-19 at 12 54 12 AM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/3e9034fd-9b53-45b0-b183-cc745c656167">

- UIWindow는 앱 UI의 배경이자 뷰에 이벤트를 전송하는 객체이다.
<img width="600" alt="Screenshot 2024-05-19 at 12 57 00 AM" src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/362d78bd-4a76-4488-b84c-b3b715f912a1">

- UIWindowScene이 씬들의 생명주기를 관리하고, 그 위에 UIWindow가 존재하며 사용자와의 상호작용을 관리하는 구조이다.
- 개념적으로는 UIScreen 위에 UIWindowScene이 존재하지만, 실제로는 UIWindowScene과 UIWindow가 UIScreen 객체를 가진다.
- 그러나 UIScreen(하드웨어) -> UIScene -> UIWindow -> UIViewController (contentView) 순으로 관리된다는 개념을 이해

### UIScreen
- UIScreen은 하드웨어 기반 정보를 가져올 수 있다는 사실을 알았다.
- iOS, iPadOS, tvOS 디바이스에서 디바이스에 통합된 디스플레이 장치 / 외부 디스플레이 장치에 대한 정보를 제공한다.
- visionOS와 호환되는 iPhone, iPad 앱에서는 UIScreen 기반 프로퍼티를 사용하면 안 된다고 한다. (신기)
- 우리가 UIScreen.main.bounds 등의 코드를 통해 접근하던 건 실제 하드웨어 정보에 기반한 데이터였던 것.
- screen 객체를 직접적으로 생성해선 안 되고, UIWindowScene의 window들로부터 screen 객체 가져와 사용하기.

### UIWindowScene, UIScene
- UIWindowScene은 UIScene을 상속하는 Scene이고, UIKit에서는 직접 씬을 생성하지 않는 이상 주로 UIWindowScene을 사용하게 된다.
- UIWindowSceneDelegate를 통해 씬 생명주기에 따른 이벤트를 전달한다.
- 그래서 새로 생성한 Xcode UIKit 프로젝트의 SceneDelegate에 가보면 UISceneDelegate가 아닌 UIWindowSceneDelegate를 상속하고 있는 것을 확인할 수 있다. (UIWindowSceneDelegate는 UISceneDelegate를 준수한다.)
- 이러한 씬은 하나 이상의 window들을 가질 수 있다.

### UIWindow
- UIWindow는 앱 UI의 배경, 가장 뒤에 위치하며 이벤트들을 뷰에 전송하는 객체임을 알았다.
- 아까 씬이 여러 개의 window를 가질 수 있다고 했는데, 대다수의 앱들은 하나의 window만을 가진다.
    - 일반적으로 추가적인 window는 추가로 연결된 외부 디스플레이에 콘텐츠를 표시하는 데 사용된다.
- window의 content view를 제공하는 rootViewController 프로퍼티를 가진다.
    - 보통 이 프로퍼티를 사용해 우리 앱의 root 뷰 컨트롤러를 설정 및 해당 뷰 컨트롤러로부터 네비게이션 분기 시작
- UIView를 상속하며, 따라서 UIResponder도 상속한다.
    - 따라서 사용자 이벤트를 전달하는 Responder chain의 일부분에 속함
<img src="https://github.com/All-About-iOS/Weekly-I-Learned/assets/22342277/4c0db5d3-32bc-4a25-9fa5-8fafe57fb733" width=500>

## 정리
- scene은 앱 UI 인스턴스를 관리하는 객체, 앱 UI 인스턴스 안에 window가 있음.
- window는 앱 UI의 가장 뒤에 위치하며 이벤트를 뷰에 전송하는 객체.
- window가 가지는 rootViewController 프로퍼티가 우리가 보는 첫 뷰 컨트롤러.
- Screen - Scene - Window - rootViewController로 이어지는 개념적 뷰 구조를 이해할 수 있었다. 
