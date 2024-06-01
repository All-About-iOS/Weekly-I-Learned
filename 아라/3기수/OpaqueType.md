### **Opaque Type(불분명한 타입)이 무엇인가요?**

불분명한 타입은 간단히 말해 타입을 추상화하는 개념입니다.

내부적으로 구체적인 타입을 지정하지만, 호출자는 타입을 구체적으로 알지 못하죠.

선언부에서 타입을 추상화하고 호출부에서 타입을 구체화하는 Generic의 정반대 개념입니다.

불분명한 타입의 주된 목적은,

코드의 유연성과 타입 추론을 향상하기 위해서 

아직 불분명한 타입이 크게 와닿지 않는데요, 예시를 살펴보겠습니다.

``` swift
protocol Fruit { }

struct Apple: Fruit { }

func pickApple() -> some Fruit {
    return Apple()
}

var apple = pickApple()
```

pickApple() 함수의 반환값 앞에 some 키워드를 추가하면 Fruit는 불분명한 타입임을 의미합니다.

![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/ba131b02-db00-402a-bfb9-fcf0adae4fdc)


apple이라는 변수를 살펴보면,

함수 내부에서 Apple라는 구체적인 타입을 반환하고 있음에도 Fruit를 채택하는 어떠한 타입이라는 것만 알 수 있습니다.

말 그대로 불분명한 타입인거죠!!

그럼,, apple의 정확한 타입이 무엇인지는 아무도 모르는 걸까요??

``` swift
protocol Fruit {
    func ripe()
}

struct Apple: Fruit { 
    func ripe() {
        print("🍎")
    }
}

struct Banana: Fruit { 
    func ripe() {
        print("🍌")
    }
}

func pickApple() -> some Fruit {
    return Apple()
}

func pickBanana() -> some Fruit {
    return Banana()
}
```

프로토콜에 익은 과일을 print 하기 위해 ripe() 함수를 필수 요구사항으로 지정하고,

해당 요구사항을 Apple과 Banana 구조체에서 구현해주었습니다.

바나나를 따는 함수인 pickBanana()도 추가해주었어요!

apple과 banana라는 인스턴스 두개를 생성하고, ripe()을 호출해보면,

``` swift
let apple = pickApple()
let banana = pickBanana()

apple.ripe()
banana.ripe()

// 🍎
// 🍌
```

이렇게!! 각각의 구조체에서 구현한 결과대로 출력되는 것을 확인할 수 있습니다. 

호출자의 입장에서는 구체적인 타입을 감추며 추상화하지만, Swift 컴파일러는 구체적인 타입을 정확히 알고 있다는 것을 알 수 있어요.

이전 코드에서는 과일을 따는 함수를 Apple과 Banana의 상황에 따라 pickApple()과 pickBanana()로 분리하였는데요,

두가지 상황에 대응하기 위하여 제네릭을 사용하여 pickFruit() 함수로 수정해보겠습니다.

``` swift
// 🚨 Function declares an opaque return type 'some Fruit', 
// but the return statements in its body do not have matching underlying types
func pickFruit<T: Fruit>(_ fruit: T) -> some Fruit {
    if fruit is Banana {
        return Banana()
    }
    
    return Apple()
}
```

이렇게 수정하면 컴파일 에러가 발생하는데요,

에러 내용을 살펴보니 반환 타입이 일치하지 않는다고 합니다.

Apple도 Banana도 Fruit 타입을 준수하는데,, 왜 반환 타입이 일치하지 않다고 하는 걸까요?!

여기서 underlying types란 Opaque Type으로 추상화 되기 전의 구체적인 타입을 의미합니다.

불분명한 타입의 경우, underlying type을 1가지로 제한을 하는데요,

pickFruit의 반환 타입으로 Apple 타입과 Banana 타입, 2가지가 가능하기 때문에 에러가 발생한 것입니다.

즉, 불분명한 타입은 type identity를 보존하기 때문에 하나의 underlying type만 가지는 것을 알 수 있습니다.

이 덕분에 컴파일러가 구체적인 타입을 추론할 수 있었던거죠!

pickFruit()에서 에러가 발생하는 원인은, 상황에 따라 여러 타입을 반환하고 있지만, 반환 타입이 불분명하기 때문이었습니다.

some 키워드를 지워 Fruit 프로토콜을 반환하는 함수로 수정하여 에러를 해결할 수 있습니다.

``` swift
func pickFruit<T: Fruit>(_ fruit: T) -> Fruit {
    if fruit is Banana {
        return Banana()
    }
    
    return Apple()
}

let apple = pickFruit(Apple())
let banana = pickFruit(Banana())
```

그냥 기본적인 프로토콜을 반환하는 함수랑 동일해 보이는데요,

apple과 banana의 타입을 살펴보면

![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/2eb2e9e8-b19b-410b-96de-215097e029ad)
![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/97697d01-9939-44da-a9a4-c89dcd1ad192)


일반적인 Fruit 프로토콜 타입이 아닌, any Fruit 타입을 반환하고 있는 것을 확인 할 수 있습니다.

any가 뭔지, some과는 어떻게 다른지에 대해서는 다음 포스팅에서 다뤄보도록 할게요^ㅡ^

---

지금까지는 함수의 반환 타입의 경우만 살펴보았는데요, 이 밖에도

-   프로퍼티 타입
-   서브스크립트
-   함수 파라미터 타입(Swift 5.7+)

의 경우도 some 키워드를 붙여 불분명한 타입을 사용할 수 있습니다.

### **요약**

\- Opaque Type은 some 키워드를 붙인 프로토콜을 의미한다.

\- Opaque Type은 underlying type을 하나만 갖는다.

\- Opaque Type은 호출자가 구체적인 타입을 알 수 없지만, 컴파일러는 정확히 알고 있다.

\- Opaque Type은 Generic과 정반대의 개념이다.
