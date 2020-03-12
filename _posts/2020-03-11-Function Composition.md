---
title: "함수의 합성(Function Composition)"
category: "Functional Programming"
---

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
    
    let composited: ((Int) -> String) = compositor(increment: increment(_:), message: message(with:))
    let result = composited(26)
    print(result) // 27살이 되었다.
```

- @escaping을 쓰는 이유 - 전달인자로 받은 함수가 탈출 후 사라질 수 있기 때문에
