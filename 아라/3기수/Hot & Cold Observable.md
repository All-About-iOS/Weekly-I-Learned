RxSwift를 공부하다보면 한 번 쯤 들어보셨을 Hot Observable과 Cold Observable에 대해 알아보겠습니다.
Hot, Cold는 너무나 익숙한 단어이지만 Observable 앞에 붙을 경우 어떤 의미일까요?

우선 ReactiveX 공식 문서에 나와있는 설명을 살펴보겠습니다.

![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/768b69cb-0e6c-4d78-a8c4-445e4063b09a)


위 내용을 살펴보았을 때!
Observable은 시퀀스를 관찰할 수 있는 시점에 따라, 혹은 이벤트가 방출되는 시점에 따라 Hot과 Cold로 구분할 수 있을 것 같습니다.


#### **Hot Observable**
어떠한 Observer가 구독을 했는지 안했는지 여부와는 상관 없이 이벤트를 방출시킬 수 있습니다. 그래서 여러 Observer가 하나의 Observable을 공유하는 것도 가능합니다.
이 때, 구독하기 전에 발생한 이벤트는 관찰 할 수 없는데요! 때문에 발생한 이벤트를 처음부터 관찰하기 힘들지도 모릅니다.

예시를 살펴보면,

``` swift
    func hotObservable() {
        let hotSubject = PublishSubject<Int>()
        
        // 이벤트 방출
        hotSubject.onNext(1)
        hotSubject.onNext(2)
        
        // 1번 구독
        hotSubject.subscribe { event in
            guard let num = event.element else { return }
            print("1번 🍎 : \(num)")
        }.disposed(by: disposeBag)
        
        // 이벤트 방출
        hotSubject.onNext(3)
        
        // 2번 구독
        hotSubject.subscribe { event in
            guard let num = event.element else { return }
            print("2번 🍎 : \(num)")
        }.disposed(by: disposeBag)
        
        // 이벤트 방출
        hotSubject.onNext(4)
    }

1번 🍎 : 3
1번 🍎 : 4
2번 🍎 : 4
```

1번 Subject를 구독하기 전 이벤트인 1, 2 는 1번 🍎에서 출력되지 않고, 구독 후 이벤트인 3, 4는 잘 출력됩니다.
2번 Subject도 마찬가지로 구독 전 이벤트는 무시되고, 구독 후의 이벤트만 출력하는 것을 알 수 있었어요!!
또한 구독 시점에 따라 관찰 가능한 이벤트가 달라진다는 점을 확인할 수 있었습니다.

다음 코드를 확인해볼까요?

``` swift
    func hotObservable() {
        let hotSubject = PublishSubject<Int>()
        
        hotSubject.subscribe { event in
            guard let num = event.element else { return }
            print("1번 🍎 : \(num)")
        }.disposed(by: disposeBag)
        
        hotSubject.subscribe { event in
            guard let num = event.element else { return }
            print("2번 🍎 : \(num)")
        }.disposed(by: disposeBag)
        
        hotSubject.onNext(Int.random(in: 0...100))
    }

// 1번 🍎 : 44
// 2번 🍎 : 44
```

일단 구독을 하고 나면, 구독 이후에 발생하는 이벤트는 구독하고 있는 모든 Observer에게 동일한 이벤트를 방출한다는 것을 알 수 있었습니다. 즉, 특정 그룹(여기서는 구독하고 있는 Observer가 되겠죠?)에 한가지 데이터를 전달하는 Multicast 방식이라고 볼 수 있겠습니다.


#### **Cold Observable**
구독하기 전에는 대기하고 있다가, 구독과 동시에 발생한 이벤트를 모두 방출합니다. 때문에 구독하는 시점에 상관 없이 시퀀스를 처음부터 관찰할 수 있습니다.

``` swift
    func coldObservable() {
        let coldSubject = ReplaySubject<Int>.create(bufferSize: 3)
       
        // 이벤트 방출
        coldSubject.onNext(1)
        coldSubject.onNext(2)
        
        // 1번 구독
        coldSubject.subscribe { event in
            guard let num = event.element else { return }
            print("1번 🍏 : \(num)")
        }.disposed(by: disposeBag)
        
        // 2번 구독
        coldSubject.subscribe { event in
            guard let num = event.element else { return }
            print("2번 🍏 : \(num)")
        }.disposed(by: disposeBag)
        
        // 이벤트 방출
        coldSubject.onNext(3)
    }
    
// 1번 🍏 : 1
// 1번 🍏 : 2
// 2번 🍏 : 1
// 2번 🍏 : 2
// 1번 🍏 : 3
// 2번 🍏 : 3
```

앞서 설명한대로 구독 여부와는 상관 없이, 발생한 모든 이벤트를 관찰할 수 있는 것을 알 수 있었습니다!

``` swift
    func coldObservable() {
        let coldSubject = ReplaySubject<Int>.deferred {
            Observable.just(Int.random(in: 0...100))
        }
       
        coldSubject.subscribe { event in
            guard let num = event.element else { return }
            print("1번 🍏 : \(num)")
        }.disposed(by: disposeBag)
        
        coldSubject.subscribe { event in
            guard let num = event.element else { return }
            print("2번 🍏 : \(num)")
        }.disposed(by: disposeBag)
    }
    
// 1번 🍏 : 36
// 2번 🍏 : 42
```

코드를 살짝 바꿔보았는데요, .deferred는 구독하는 시점에 아이템을 생성한다고 합니다!
결과를 살펴보면 같은 Subject임에도 구독하는 시점에 따라 다른 이벤트를 방출하는 것을 알 수 있었어요.
즉, 1 : 1로 데이터를 전달하는 Unicast 방식이라고 볼 수 있겠네요!!

이러한 특성으로 인해 UIEvent를 처리 등은 Hot Observable을, API 통신을 할 때는 Cold Observable을 사용한다고 합니다^ㅡ^
