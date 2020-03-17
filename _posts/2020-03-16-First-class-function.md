---
title: "일급함수(First class function)"
category: "Functional Programming"
---
# 일급 함수?

- 함수 자체를 값으로써 다룰 수 있다.
- 변수에 할당할 수 있어야 한다.
- 인자로 전달할 수 있어야 한다.
- 반환 값으로 전달할 수 있다.


# 함수의 합성

- 함수의 반환값이 다른 함수의 입력값으로 사용되는 것
- 함수가 합성되기 위해서는 함수의 반환값과 반환값을 받아들이는 값은 타입이 서로 같아야함
- 순수 함수를 만들어두고 그것을 조합 및 재활용
- 함수 단위의 재활용성이 높아진다
- 코드를 읽기 쉬워진다. ← 이건 아직 잘 모르겠다
- 함수 단위의 테스트가 쉬워진다
- 버그의 감소 ← 일단 써봐야 알겠다.


```swift
func increment(_ value: Int) -> Int {
    return value + 1
}
        
func message(with age: Int) -> String {
    return "2020년에 \(age)살이 되었다."
}
        
func compositor(increment: @escaping (Int) -> Int,
               message: @escaping (Int) -> String) -> (Int) -> String {
    return { (age: Int) in  // age -> compositor의 파라메터
        return message(increment(age))
    }
}
    
let composited: ((Int) -> String) = compositor(increment: increment(_:), message:   message(with:))
let result = composited(26)
print(result) // 27살이 되었다.
```

- @escaping을 쓰는 이유 - 전달인자로 받은 함수가 탈출 후 사라질 수 있기 때문에

# Async Result (비동기 반환)

- 결과는 나중에 생길 때 전달 받고, 프로그램의 수행을 멈추지 않음
- 시간이 걸리는 작업이나 네트워크를 통한 결과를 얻을 경우 비동기 방식으로 구현하는 것이 좋음
- 결과가 리턴값으로 전달 되는 것이 아니라 전달받은 함수를 호출함으로써 전달
- 결과가 발생하는 시점에 수행

```swift
func f(_ num: [Int]) -> Int {
    sleep(1)
    let sum = num.reduce(0, +)
    return sum
}

// async를 사용하는 경우 
func asyncF(_ num: [Int], onCompletion: @escaping (Int) -> Void) {
        DispatchQueue.main.async {
        sleep(1)
        let sum = num.reduce(0, +)
        print(sum)
        onCompletion(sum)
    }
}

dispatchMain() // 커맨드라인에서는 이거 안하면 async 안돌아감
```

# Currying (커링)

- 여러 개의 파라미터를 받는 함수를 하나의 파라미터를 받는 여러 개의 함수로 쪼개는 것
- 함수들이 서로 chain을 이루면서 연속적으로 연결되려면 output과 input의 타입과 **갯수가 같아**야한다.
- **함수의 합성을 원활하게 하기 위해서 커링을 사용**

```swift
func curry<A, B, C>(f: (A, B) -> C) -> (A -> (B -> C)) {
  return { a: A in
            { b: B in
              return f(a, b) // returns C
            }
       
}

func add(x: Int, y: Int) {
    return x + y
}

add(1,2)

func filterWithNum(_ n: Int) -> ([Int]) -> [Int] {
    return { b in
        return b.filter { $0 % n == 0}
    }
}

func filterArray(_ nums: [Int], _ r: Int) -> [Int] {
    let filteredArr = filterWithNum(r)
    return filteredArr(nums)
}

print(filterArray([1,2,3,4], 2))   // [2, 4]


// 함수의 실행을 늦추고 싶을경우 나중에 인자를 받으면 실행 된다.
// 인자를 고정
```
