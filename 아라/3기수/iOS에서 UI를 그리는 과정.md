## **Main Run Loop**
![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/b5d5cbb1-506f-4c07-83aa-75665598b4f2)
run loop는 키보드 마우스와 같은 input 소스나 timer에 대해 처리하는 루프로, thread당 하나의 run loop가 존재합니다. 기본적으로 자동으로 실행되지 않지만 main thread에서는 UIApplication으로 인해 자동으로 실행되며 반복 실행은 하지 않습니다.
event queue에 event를 쌓고, run loop가 실행되는 동안 해당 이벤트를 핸들링하게 되는데요, 이때 해당 run loop에서의 event 핸들링이 모두 끝나게 되면 update cycle로 넘어가게 됩니다.

## **Update Cycle**
![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/2dfec569-2cff-463a-95e9-fdf80bb3d62a)
update cycle은  layout, display, constraints 등 view의 모든 변경 사항을 갱신해주는 과정입니다. 대략 60초에 1번 꼴로 실행되기 때문에 사용자가 체감할 수 없을 정도로 매우 짧은 시간이지만, 이벤트 발생 시점과 update cycle 시작 시간 사이 차이는 분명히 존재합니다.
그래서 view가 원하지 않는 동작을 하기도 하는데요, 이를 해결하기 위해 update cycle을 제어하는 여러 메서드를 활용할 수 있습니다.

### **Layout**
layout은 view의 크기 및 위치를 의미합니다. layoutSubviews()를 호출하면 subview의 layoutSubviews()를 재귀적으로 호출하여
자기 자신 뿐 아니라 자신의 모든 subview의 크기 및 위치를 재계산하여 변경해줍니다. 변경 사항이 없는 subview도 다시 그려줘야하기 때문에 그만큼 많인 비용을 요구하는데요, 그래서 애플은 layoutSubviews()를 직접 호출하는 것을 지양하고 있습니다.

그럼 어떻게 view를 업데이트 해줄 수 있을까요?

> You shouldn’t call this method directly. If you want to force a layout update, call the [setNeedsLayout()](https://developer.apple.com/documentation/uikit/uiview/1622601-setneedslayout) method instead to do so prior to the next drawing update. If you want to update the layout of your views immediately, call the [layoutIfNeeded()](https://developer.apple.com/documentation/uikit/uiview/1622507-layoutifneeded) method.
공식 문서에 적힌대로 setNeedsLayout()과 layoutIfNeeded()를 통해 간접적으로 layoutSubviews()를 호출할 수 있습니다.

#### **setNeedsLayout()**
> Invalidates the current layout of the receiver and triggers a layout update during the next update cycle.
setNeedsLayout()는 다음 업데이트 주기까지 기다렸다가 해당 view와 subview의 모든 layout 변경 사항이 한꺼번에 적용되며, layoutSubviews()의 실행을 예약하는 메서드입니다. 예약만 하고 바로 반환되기 때문에 비동기로 동작한다고 볼 수 있으며 현재의 레이아웃을 무효화하는 역할을 합니다. 모든 레이아웃 업데이트를 하나의 업데이트 주기로 통합할 수 있기 때문에 성능향상을 기대할 수 있습니다.

#### **layoutIfNeeded()**
> Lays out the subviews immediately, if layout updates are pending.
layoutIfNeeded()는 subview의 layout을 즉시 업데이트하는 메서드로, 다음 업데이트 주기를 기다리지 않고 layout 변경 사항을 적용시키기 위해 바로 layoutSubviews()를 호출합니다. 호출이 끝나면 layout은 이미 변경되어 있어 동기적으로 동작한다고 볼 수 있습니다. 이런 특징으로 인해 애니메이션 관련 작업에서 자주 사용됩니다. 

### **Display**
배경 색, 텍스트, 이미지 등 view의 컨텐츠를 의미합니다. draw()메서드를 통해 해당 뷰의 display만을 새롭게 그려줄 수 있습니다. layoutSubviews()와 마찬가지로 직접 호출해서는 안되며 setNeedsDisplay()를 통해 간접적으로 실행시킬 수 있습니다.

#### **setNeedsDisplay()**
setNeedsDisplay()가 호출되면 draw()의 실행을 예약하고, 다음 update cycle동안 기다렸다가 해당 view의 컨텐츠를 다시 그려주는 메서드입니다. 일반적으로 view의 컨텐츠가 변경된 경우에만 다시 그려주는데, 우리는 setNeedsDisplay()를 호출한 적이 없는데 view가 새롭게 그려지는 것을 경험했을 것입니다.
그 이유는 view의 특정 property가 수정될 때마다 내부적으로 setNeedsDisplay()를 호출해주기 때문이었습니다!! property를 수정하는 것만으로도 view를 다시 그려주기에 충분하기 때문에 ifNeeded 메서드는 애플이 지원해주지 않는 듯 합니다. 만약 view의 프로퍼티에 직접적인 연관은 없지만 view를 즉각적으로 update해주고 싶다면 didSet에서 setNeedsDisplay를 명시적으로 호출해줄 수 있습니다.

### **constraints**
constraints를 통해 autolayout을 설정할 수 있습니다. updateConstraints()를 통해 view의 constraints를 변경할 수 있으며, 여기서 constraints의 변경이란 1) constraints 추가 및 제거 2) 기존의 constraints 수정 3) 우선순위 변경 등이 있습니다. layoutSubviews()와 draw()와 마찬가지로 직접 호출해서는 안되며, 이를 간접적으로 호출하기 위해서 setNeedsUpdateConstraints()와 updateConstraintsIfNeeded()를 사용할 수 있습니다.

#### **setNeedsUpdateConstraints()**
setNeedsUpdateConstraints()가 호출되면 updateConstraints()의 실행을 예약하고, 다음 update cycle에 constraints의 변경사항을 적용시킵니다.

#### **updateConstraintsIfNeeded()**
호출되는 즉시 constraints의 변경사항을 적용시킵니다.

위 두가지 메서드는 이름에서 바로 유추할 수 있었는데요, constraints에는 한가지 메서드가 더 존재합니다.

#### **invalidateIntrinsicContentSize()**
본질적인 컨텐츠 사이즈를 의미합니다. UILabel이나 UIButton의 width나 height에 대한 constraints를 설정해주지 않아도 오류가 나지 않는 이유는 내부적인 사이즈(intrinsicContentSize라는 프로퍼티)를 가지기 때문입니다. 즉, 커스텀 뷰의 경우 intrinsicContentSize와 invalidateIntrinsicContentSize() 메서드를 구현해주어야 합니다.
이를 구현함으로써 다음 update cycle에서 updateConstraints()의 실행을 예약하고, 그 때 intrinsicContentSize를 다시 계산하여 새로운 constraints를 적용할 수 있습니다.

**invalidateIntrinsicContentSize와 layoutIfNeeded의 차이?**

invalidateInstrinsicContentSize 

-   intrinsicContentSize : 고유값
    -   모든 view에 존재하지 않음

layoutIfNeeded

-   frame : autolayout을 통해 설정된 위치 및 크기
    -   모든 view에 존재
