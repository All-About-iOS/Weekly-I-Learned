## **요약**

1. 기존의 Frame을 이용한 정적인 Layout의 문제점을 해결하기 위해 Constraint(관계의 제약)를 이용한 동적인 Layout, 즉 AutoLayout이 등장함.
2. AutoLayout은 AutoLayout Engine이라는 코어 모듈을 이용해 Constraint를 1차 방정식으로 계산한다.
3. 계산한 값을 Variable(minX, minY, width, Height)라는 형태로 View에게 전달하고,
4. View는 해당 값을 frame에 복사해 View 본인과 subView의 Layout을 결정한다.

## **AutoLayout의 등장 배경**

AutoLayout이 등장하기 전에는 Frame(size와 Position)만을 이용해 뷰의 위치와 크기를 정했다.

즉, 절대값이었기 때문에 디바이스가 늘어남에 따라 디바이스별 다른 Layout을 설정해줘야 했다는 점이다. 이러한 문제를 해결하기 위해 Constraint, 즉 관계의 제약을 통한 Layout인 AutoLayout이 등장하게 된다.

`AutoResizingMask가 AutoLayout과 유사한 기능을 수행할 수는 있지만, 상대적으로 간단한 UI에서만 적용이 가능하다.`

## **AutoLayout Engine**

AutoLayout은 View 사이의 관계를 1차 방정식으로 정의하는 것이다.

<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/809204aa-7760-4ce7-a5dc-7fd62df83dd1" width=500>


조금 더 정확히는, `RedView.minX = BlueView.minX + BlueView.width + 8` 과 같은 방정식이 생성되는데, 이러한 과정을 수행하는 것이 AutoLayout Engine이다.

`그런데, 상황에 따라 AutoLayout의 값(방정식의 값)이 여러개가 도출되는 상황이 나올 수 있는데, 이 때에는 `**AutoLayout의 Error minimization**을 통해 1개의 최적해를 도출해낸다.`

<img src="https://github.com/JUNY0110/Weekly-I-Learned/assets/98405970/46b04ddc-660b-4a41-950a-80d5c3b93d05" width=500>

이렇듯 AutoLayout Engine은 설정된 여러 AutoLayout을 계산하는 코어 모듈이며 아래와 같이 동작한다.

1. View가 Constraint를 추가하면, View는 추가된 Constraint를 방정식으로 변환해 Engine으로 전달한다.
2. Engine은 전달 받은 방정식을 계산하는데, 우리가 방정식의 해를 구하는 것처럼 각각의 Variable을 계산한다.
3. 계산 결과 값은 View의 Origin, Size 값과 유사한 형태인 Variable(minX, minY, width, height)을 View에게 전달된다.
4. Engine은 Variable에 값을 설정할 때마다 View에게 Variable의 변경 내용을 전달하고, 
5. View는 Variable 변경 이벤트를 받을 때마다 [setNeedsLayout()](https://developer.apple.com/documentation/uikit/uiview/1622601-setneedslayout)을 호출한다.
6. setNeedsLayout()이 호출되면, 해당 View의 [layoutSubViews()](https://developer.apple.com/documentation/uikit/uiview/1622482-layoutsubviews)가 호출된다. 여기서 View는 Engine의 데이터를 Frame으로 복사하고, View 자신과 subViews의 Layout을 결정한다.

## **AutoLayout Error**

### 1. 불명확한 조건 (**Ambiguous Layout)**

근데 만약 AutoLayout Engine이 방정식을 위한 x, y 값이 모호하다면 어떨까?

x, y에 포함될 top, left, bottom ,right의 constraint를 정하지 않았다면 Error Ambiguous Layout Error를 만나게 된다.

즉 이를 해결하기 위해서는 빠뜨린 Constraint를 반영해주면 된다.

### 2. **레이아웃 조건 충돌 (Unsatisfiable Layouts)**

수학문제를 풀 때에도 해가 없는 문제가 있듯, Layout 조건이 양립할 수 없는 경우 Conflicting Constraints Error를 만나게 되며, 이를 해결하기 위해서는 Constraint를 조정하거나, Priority를 설정해야 한다.
