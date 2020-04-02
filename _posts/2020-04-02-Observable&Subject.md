---
title: "[RxSwift] Observable, Subject"
category: "RxSwfit"
---
# Observable

Observable → 관찰 가능한, 관찰할 수 있는

하나의 Sequence(수열)이며 async하다.

Observable이 이벤트를 발생 시키는 것을 emit한다고 표현한다.

[RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#from)

이건 그냥 참고 자료.

## just, of, from

- just - 하나의 파라미터를 받아 하나의 이벤트를 발생 시킨다
- of - 파라미터의 타입을 전달, 배열이라면 배열 자체를 전달하는 것
- from - 배열 타입을 전달받아 배열 안에 있는 요소들을 꺼내 sequence를 생성한다. (array, dictionary, set포함)
    - 타입이 다른 요소를 넣을 경우 error
    - 타입 추론을 Any로 한다면 된다.
    
```swift
let observable1 = Observable.just(1) // Observable<Int>

let observable2 = Observable.of(1, 2, 3) // Observable<Int>

let observable3 = Observable.of([1, 2, 3]) // Observable<[Int]>

let observable4 = Observable.from([1, 2, 3]) // Observable<Int>
```

## subscribe

subscribe()는 NotificationCenter에서 addObserver() 대신 사용되는 것이라고 생각하면 된다.

다른점이라면 NotificationCenter에서는 .default를 이용해 싱글톤을 사용했지만

Rx에서는 Observable은 subscriber가 없다면 **발생한 이벤트를 전송하지 않는다**.

```swift
// from
observable4.subscribe { event in
    print(event)
    if let ele = event.element {
        print(ele)
    }
}
/*
 print(event) 일 경우 (optional?)
 next(1)
 next(2)
 next(3)
 completed
 .next와 .completed 이벤트 발생

 if let ele = event.element {
       print(ele)
 }  일 경우
 1
 2
 3
 엘리멘트로 접근 가능
 */

// of + Array
observable3.subscribe { event in
    print(event)
    if let ele = event.element {
        print(ele)
    }
}

/*
 print(event) 일 경우 (optional)
 next([1, 2, 3])
 completed

 if let ele = event.element {
     print(ele)
 } 일 경우
 [1, 2, 3]
 */

// from
observable4.subscribe(onNext: { element in
    print(element)
})
/*
1
2
3
     */
 ```

 onNext를 사용하게 되면 next이벤트에 대해서만 emit하므로
 따로 바인딩하지 않아도 된다.

![RxDisposable](/assets/Images/RxDisposable.jpeg)

subscribe는 모두 합쳐진걸 emit하기 때문에 onNext, onError, completed 등을 emit하지만 

subscribe(onNext: ~~를 쓰면 나뉘어져서 그런게 아닐까 하는 생각이 든다.

### Disposable

subscribe()는 Disposable을 리턴한다.

Observable의 사용이 끝나면 메모리를 해제해야하는데 그때 사용하는 것이 Dispose()

```swift
let subscription1 = observable4.subscribe(onNext: { element in
    print(element)
})

subscription1.dispose()
```

DisposeBag은 **deinit()이 실행될** 때 모든 메모리를 해제한다.

subscribe가 리턴하는 Disposable 인스턴스를 넣어준다.

```swift 
let disposeBag = DisposeBag()

Observable.of("A", "B", "C")
    .subscribe {
        print($0)
}.disposed(by: disposeBag)
/*
 next(A)
 next(B)
 next(C)
 completed
 */
```

- DisposeBag에서 var 쓰는 이유

```swift
var disposeBag: DisposeBag 
... 많은 코드 사용후 ... 
disposeBag = DisposeBag()
```

disposeBag에 Observable이 잘 들어있다는 가정 하에 새로 DisposeBag을 생성하면 

기존의 disposeBag에 들어있는 Observable이 deinit에 의해 dispose되면서 구독 중인 이벤트들이 **초기화**되는 효과가 발생하게 된다.

### Create

create는 클로저를 사용해 subscribe 메서드를 쉽게 구현할 수 있게 한다.

```swift
Observable<String>.create { (observer) -> Disposable in
    observer.onNext("A")
    observer.onCompleted()
    observer.onNext("?")    // 될까?
    return Disposables.create()
}.subscribe(onNext: {
    print($0)
}, onError: {
    print($0)
}, onCompleted: {
    print("Completed")
}, onDisposed: {
    print("Disposed")
}).disposed(by: disposeBag)
/*
 A
 Completed
 Disposed
 */
```

 Completed가 되고 난 후에는 next가 호출되지 않고 Disposed가 되어 "?"가 프린트되지 않는다는 것을 알 수 있다.

- 언제 이벤트들이 방출되는지에 따라 Observable을 두가지로 분류할 수 있다
- **Hot Observable**
    - 생성과 동시에 이벤트를 방출하기 시작하는 Observable.
    - Subscribe 되는 **시점과 상관없이** Observer 에게 이벤트를 중간부터 전송한다.
- **Cold Observable (just, of, from, create)**
    - **Subscribe 되는 시점부터** 이벤트를 생성해 방출한다.

# Subject

Subjectsms Observable을 subscribe(구독)할 수 있고 다시 emit(방출)할 수 있다.

혹은 새로운 Observable을 emit할 수 있다.

- PublishSubject
- BehaviorSubject
- ReplaySubject
- ~~Variable~~ - deprecated!
- BehaviorRelay

## PublishSubject

PublishSubject는 subscribe전의 이벤트는 emit하지 않고,

subscribe 이후의 이벤트만 emit

에러 이벤트가 발행산다면 그 이후 이벤트는 emit하지 않는다.

```swift
let subject1 = PublishSubject<String>()
subject1.onNext("First")

subject1.subscribe { event in
    print(event)
}
// 이때는 프린트 되는 것이 없다. -> subscribe이전에 이벤트가 발생했기 때문에!

subject1.onNext("Second")
subject1.onNext("Third")
/*
 next(Second)
 next(Third)
 completed
 */
```

만약 이 상황에서 dispose나 onCompleted, onError를 추가하고 "Fourth"를 추가하면 Fourth가 프린트 될까?

```swift
subject1.dispose()    
// subject1.onCompleted()
// subject1.onError(Error.self as! Error)
subject1.onNext("Fourth")
```

정답은 찍히지 않는다. (코드에는 임의 주석 처리)

dispose()할 경우 그대로 이벤트가 종료되고

onCompleted()의 경우 completed가 프린트 되고

onError를 할 경우 에러가 어쩌구라고 프린트 된다.

## BehaviorSubject

BehaviorSubject는 PublishSubject와 거의 같지만 BehaviorSubject는 **반드시 초기화** 해줘야한다.

```swift
let subject2 = BehaviorSubject(value: "Initial value")
subject2.subscribe { event in
    print(event)
}
subject2.onNext("Second value")
// next(Initial value)
// next(Second value)
```

만약 subscribe되기 전 subject2에 "Last value"를 이벤트를 emit하면 어떻게 될까?

```swift
let subject2 = BehaviorSubject(value: "Initial value")
subject2.onNext("Last Value")
subject2.subscribe { event in
    print(event)
}
subject2.onNext("Second value")
// next(Last Value)
// next(Second value)
```

subscribe되기 전 emit이 되면 Initial value는 last value로 변해 방출되게 된다.

## ReplaySubject

ReplaySubject는 미리 정해진 사이즈 만큼 가장 최근의 이벤트를 최근의 subscriber에게 전달한다.

createUnbounded 는 무한!(모든이벤트) <- 메모리 관리에 유의하기!

```swift
let subject3 = ReplaySubject<String>.create(bufferSize: 2)

subject3.onNext("First")
subject3.onNext("Second")
subject3.onNext("Third")

subject3.subscribe { print($0) }
/*
 next(Second)
 next(Third)
 */

subject3.onNext("Fourth")
subject3.onNext("Fifth")
subject3.onNext("Sixth")

print("- Second subscribe")

subject3.subscribe { print($0) }
/*
 next(Second)
 next(Third)
 next(Fourth)
 next(Fifth)
 next(Sixth)
 - Second subscribe
 next(Fifth)
 next(Sixth)
 */
```

버퍼 사이즈 만큼 마지막 값이 replay된다.

만약 buffersize가 1이라면 제일 마지막의 Sixth만 방출된다.

## Variable

let subject4 = Variable("Initial Value") 를 치니

'Variable' is **deprecated**: Variable is deprecated. Please use `**BehaviorRelay**` as a replacement. 라고 해서 패스

### BehaivorRelay를 들어가기 전에,,

플레이 그라운드로 진행하는도중 subscribe하는데 자꾸

```swift
error: Couldn't lookup symbols:
RxRelay.BehaviorRelay.asObservable() -> RxSwift.Observable<A>
```

라고 해서 검색해보니

https://github.com/ReactiveX/RxSwift/issues/1502

observer가 아니기 때문에 subject가 아니라고 한다.  

맞는 말이지만 지금은 필요하지 않다.

그냥 variable를 주석 처리했다가 클린 빌드하니 되었다.

그럼 마지막 BehaviorRelay를 공부해보자.

## BehaviorRelay

BehaviorRelay 는 **RxCocoa**에 포함되어 있다.

Subject와 다르게 onNext가 아니라 accecpt. (value는 get-only)

error, complete가 없음(무시..)

UI에서 에러가 났다고 화면 안 그릴 순 없으니까.. (죽지 않는 친구)

PublishSubject와 BehaviorSubject를 래핑한 것이라고 보면 된다.

그래서 subscribe할 때 .asObservable()을 사용한다.

근데 .asObservable() 안써도 subscribe가 된다..?

```swift
let relay = BehaviorRelay(value: "Initial value")

relay.asObservable()
    .subscribe {
        print($0)
}
// next(Initial value)
```

이런 식으로 emit해준다.

```swift
let relay2 = BehaviorRelay(value: [String]())

relay2.accept(["Number 1"])

relay2.asObservable()
    .subscribe {
        print($0)
}
//next(["Number 1"])
```
[String]()만 subscribe를 하게 되면 []으로 뜨게 되고,

accept로 이벤트를 emit할 수 있다.

```swift
let relay3 = BehaviorRelay(value: ["Number 1"])

relay3.accept(["Number 2"])

relay3.asObservable()
    .subscribe {
        print($0)
}
// next(["Number 2"])
```

BehaviorSubject와 마찬가지로 subscribe 되기 전에 값이 emit된다면,

구독 시에도 값은 마지막 값이 된다.

### BehaviorRelay에 값 append 하기

방법 1.

```swift
let relay4 = BehaviorRelay(value: ["Number 1"])
relay4.asObservable()
    .subscribe {
        print($0)
}
relay4.accept(relay4.value + ["Number 2"])
// next(["Number 1", "Number 2"])
```

원래 값에 배열을 더해준다거나

방법 2.

```swift
let relay4 = BehaviorRelay(value: ["Number 1"])
var value = relay4.value
value.append("Number 2")
value.append("Number 3")
relay4.accept(value)

relay4.asObservable()
    .subscribe {
        print($0)
}
// next(["Number 1", "Number 2", "Number 3"])
```

value에 접근하여 값을 append해주고 그 값을 accept 한다.


[RxSwift Study Example](https://github.com/mini0212/RxSwiftStudy)
