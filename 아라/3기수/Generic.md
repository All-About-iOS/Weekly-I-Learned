## Generic(제너릭)이란 무엇인가요?
제너릭은 타입에 제한이 없이 범용적으로 사용할 수 있는 문법입니다.

유연하고 재사용 가능한 코드를 작성할 수 있다는 장점이 있는데요!

 

이러한 특징으로 인해 Apple은 제네릭을 Swift의 가장 강력한 기능 중 하나로 소개하고 있습니다.

그래서 일까요? Swift 대부분의 라이브러리는 제네릭으로 구현되어 있습니다.

우리는 알게 모르게 제네릭을 사용하고 있었다는 사실,,

 

Array나 Dictionary도 제너릭으로 구현되어있는데요, 

``` swift
var array = [] 			// 🚨 에러 발생
var array1 = [Int]()		// 1번 방식
var array2 = Array<Int>()	// 2번 방식
```

빈 배열을 생성할 때는 컴파일러가 어떤 요소를 담을 배열인지 명시해주어야 합니다.

2가지 방식이 있지만 그 중 2번 방식에 집중해주세요!

 

Array는 배열을 의미하고 Int는 배열에 담을 요소의 타입을 의미합니다.

여기서 어떤 부분을 통해 제네릭의 흔적을 찾을 수 있을까요?

 

바로 < > 입니다!

 

Array의 구현부를 잠시 살펴보면,

``` swift
@frozen public struct Array<Element> {
	// ...
}
```

우선 Array는 구조체 타입으로 구현되어 있는 것을 알 수 있습니다.

그럼 Element는 뭘까요?

 

Int 타입의 빈 배열을 생성할 때, < > 안에 Int를 담아주었던거 기억하시죠?!

<Element> 이 표현이 바로 제네릭 타입을 명시하는 방법입니다.

 

Element는 선언부에서 전달 받은 타입을 의미하며 Type Parameter라고 부르는데요,

타입 파라미터는 실제 타입을 대신하는 placeholder 역할을 합니다.

앞 서 언급한 예시에서, 'Element를 대신해서 Int 타입으로 Array를 만들어줘~'하고 요청하는 것이죠.

보통은 Type의 앞 글자를 따서 <T>를 많이 사용하지만, 대문자로 시작하기만 한다면 상관없답니다^ㅡ^ (ex. <Element>, <A>, <T, U> 등)

 

⭐️ 왜 타입 파라미터를 < >로 감싸고 있을까요?

단순히 Element로 사용할 경우, 컴파일러는 Element라는 타입이 존재하는지 찾게 됩니다.
당연히 구현한 적이 없으니 에러가 나겠죠?
그렇기 때문에 < >로 감싸서 'Element는 원래 있는 타입이 아니고 placeholder 역할이야' 하고 알려주는 것입니다!

 

그럼 제너릭의 개념과 어떤 형태로 구현할 수 있는지 알았으니, 직접 제너릭 타입을 만들어볼까요?

 

## 제네릭 사용해보기
``` swift
struct Stack<T> {
    var stack: [T] = []
    
    mutating func push(_ element: T) {
        stack.append(element)
    }
    
    mutating func pop() -> T? {
        return stack.popLast()
    }
}
```

Swift 언어에는 Stack 자료구조를 지원하지 않는데요,

배열과 제너릭을 사용하면 Stack을 직접 구현할 수 있습니다!

 

T타입의 요소를 담을 배열인 stack을 선언해주고,

배열에 요소를 담는 push와 마지막에 추가된 요소를 빼내는 pop 메서드를 구현해주었습니다.

(mutating 키워드에 대해서는 나중에 정리해볼게요)

 

stack 배열에 어떤 타입이 들어올지는 모르겠는데, 그 타입을 T라고 한다면

 

push 할 때 T타입의 요소만 stack에 넣을 수 있어!

pop 할 때는 T타입의 요소만 반환할 수 있어!

 

라고 하는 것이죠.

 
``` swift
var myStack = Stack<Int>()
myStack.push(1)
myStack.push("orzr")	//🚨에러 발생
```

Stack의 타입 파라미터로 Int 타입을 전달햇는데,

String타입을 push 하려하면 에러가 발생하는 것을 확인할 수 있습니다.

 

이미 Stack의 실제 타입을 Int로 지정을 했기 때문이죠!!

 

실제 타입을 String으로 하는 Stack을 생성하려면

``` swift
var myStack = Stack<String>()
myStack.push("orzr")
``` 

굳이 Stack 구조체를 새로 생성하지 않고도

이전에 제너릭 타입으로 구현한 Stack을 그대로 사용해주면 됩니다.

 

만약 제너릭을 사용하지 않는다면,,,

``` swift
struct IntStack {
    var stack: [Int] = []
    
    mutating func push(_ element: Int) {
        stack.append(element)
    }
    
    mutating func pop() -> Int? {
        return stack.popLast()
    }
}

struct StringStack {
    var stack: [String] = []
    
    mutating func push(_ element: String) {
        stack.append(element)
    }
    
    mutating func pop() -> String? {
        return stack.popLast()
    }
}
```

이렇게 원하는 타입에 맞게 새로운 구조체를 생성해주어야는데 참 귀찮고 비효율적이지 않나요~?

애플이 많은 라이브러리를 제너릭으로 구현한 이유를 체감할 수 있었습니다^ㅡ^
