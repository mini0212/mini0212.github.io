---
title: "고차 함수(Higher Order Function)"
category: "Functional Programming"
---

# 고차함수

- 함수를 매개변수로 받는 함수
- 함수를 반환 값으로 사용하는 함수
- 대표적인 고차 함수로는 `map`,  `filter`,  `reduce`가 있다.

### map

- 데이터를 변형하고자 할 때 사용
- 기존 컨테이너의 값은 변경되지 않고 새로운 컨테이너를 생성해 반환
- `for-in` 구문과 별 차이가 없지만 더 좋은 점
    - 코드 재사용이 용이
    - 컴파일러 최적화 측면에서 성능이 좋음
        - 빈 배열 초기 생성할 필요 x
        - `append` 연산 수행할 필요 x
    - 다중 스레드 환경에서 하나의 컨테이너에 여러 스레드들이 동시에 변경하려고 할 때 예측 못한 결과 발생을 방지할 수 있다
    
```swift
    let numbers = [0, 1, 2, 3, 4]
    
    // 이것은 함수형이 아닙니다ㅜ
    func multiply2(_ numbers: [Int]) -> [Int] {
        var multiplyNumbers = [Int]()
            
        for number in numbers {
            multiplyNumbers.append(number * 2)
        }
        return multiplyNumbers
    }
    
    print(multiply2(numbers)) // [0, 2, 4, 6, 8]
    
    // map 을 활용한다면?
    print(numbers.map({ (num) -> Int in
        num * 2
    }))  // [0, 2, 4, 6, 8]
    
    print(numbers.map { $0 * 2 }) // [0, 2, 4, 6, 8]
```

> 매개 변수로 전달 할 함수를 클로저 상수로 두어 코드를 재사용할 수 있다.

```swift
    let multiple: (Int) -> Int = { $0 * 2 }
    print(numbers.map(multiple) // [0, 2, 4, 6, 8]
```

### filter

- 컨테이너 내부의 값을 걸러서 추출하고자 할 때 사용
- `filter`를 매개 변수로 전달되는 함수의 반환 타입은 `Bool`

```swift
    let numbers = [0, 1, 2, 3, 4]
    
    func filterEvens(_ numbers: [Int]) -> [Int] {
        var evenNumbers = [Int]()
            
        for number in numbers {
            if number % 2 == 0 { evenNumbers.append(number) }
        }
            
        return evenNumbers
    }
    
    print(filterEvens(numbers)) // [0, 2, 4]
    
    print(numbers.filter({ (num) -> Bool in
        num % 2 == 0
    }))  // [0, 2, 4]
    print(numbers.filter({ $0 % 2 == 0 })) // [0, 2, 4]
    
    // 물론 위의 map에서 했듯 클로저 상수로 둘 수 있다.
    let filterEven2: (Int) -> Bool = { $0 % 2 == 0}
    print(numbers.filter(filterEven2))
```

> `map`과 `filter`를 연결하여 사용할 수도 있다.

```swift
    let doubleEven = numbers
        .map { $0 + 2 }
        .filter { $0 % 2 == 0 }
    print(doubleEven)  // [2, 4, 6]
```

### reduce

- 컨테이너 내부를 하나로 합쳐주는 기능
- 정수 배열이면 전달받은 함수의 연산 결과로 합쳐주고, 문자열 배열이라면 문자열을 하나로 합쳐준다
- 첫 번째 매개변수를 통해 초깃값을 지정할 수 있다.
    - 이 초깃값이 최초의 `$0`으로 사용된다
    
```swift
    let numbers = [0, 1, 2, 3, 4]
    
    func reduceNumbers(_ numbers: [Int]) -> Int {
        var reduceNumbers = 10 // 초깃값
            
        for number in numbers {
            reduceNumbers += number
        }
        return reduceNumbers
    }
    
    print(reduceNumbers(numbers)) // 20
    
    // 여기서도 10이 초깃값
    print(numbers.reduce(10, { (result: Int, currentItem: Int) -> Int in
        return result + currentItem
    })) // 20
    
    print(numbers.reduce(10) { $0 + $1 }) // 20
    
    let texts = ["a", "b", "c", "d"]
    print(texts.reduce("") { $0 + $1 }) // abcd
```

- reduce에서 클로저의 매개변수 이름을 first, second보다는 result, currentItem이라고 하는 것이 좋다.
    - result 는 초깃값으로부터 출발하여 마지막 요소까지 순회하는 결과값
    - currentItem은 현재 순회하고 있는 요소의 값
    - 만약 currentItem들이 없으면 초기값을 반환

### reduce(into:_:)

```swift
    let letters = "abracadabra"
    let letterCount = letters.reduce(into: [:]) { counts, letter in
        counts[letter, default: 0] += 1
    } // ["d": 1, "c": 1, "a": 5, "r": 2, "b": 2]
```

- 단어 빈도 같은 것에 사용 가능하다
- 결과는 copy-on-write(inout)으로 배열이나 딕셔너리와 같다

## forEach

- 순차적으로 element에 접근할 수 있음
- collection에서 제공하는 기능이며 클로저 방식으로 사용됨
- for-in과 다르게 중간에 break로 탈출하지 못함
- return을 하여 종료하지만 element를 인자로 가진 클로저가 실행이 된다.

```swift
    enum Calculator {
        case nomal
        case multiple
        case plus
        var calc: (Int) -> Void {
            switch self {
            case .nomal: return {print($0)}
            case .multiple: return {print($0 * $0)}
            case .plus: return {print($0 + $0)}
            }
        }
    }
    
    let calc: Calculator = Calculator.nomal
    let numbers: [Int] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    numbers.forEach(calc.calc) //enum 내부의 함수를 실행
```

### flatMap

- flatten + map의 합성
- map에서 Optional로 둘러싸진 결과에서 Optional을 풀어낸 값이 나올 수 있게 한 것
- swift4 이후 compactMap으로 변경

```swift
    // 2차 배열을 1차 배열로 합성
    let arr = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    let flatArr = arr.flatMap { $0 }
    print(flatArr) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
    
    // Optional 제거
    let a = [1, 2, 3, 4, 5]
    let c: (Int) -> Int? = { n in
        if n % 2 == 0 {
            return n * 2
        }
        return nil
    }
    
    print(a.map(c)) // [nil, Optional(4), nil, Optional(8), nil]
    let b = a.compactMap(c)
    print(b)  // [4, 8]
```

