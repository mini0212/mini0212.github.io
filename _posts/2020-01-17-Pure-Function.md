---
title: "순수 함수(Pure Function)"
category: "Functional Programming"
---

# Functional Programming?

- 자료 처리를 **수학적 함수의 계산**으로 취급
- 상태와 가변 데이터를 멀리함
- 변수와 반복문이 없음
- Side-effect가 없음 → 동작을 이해하고 예측하기 쉬워짐

 * Side-effect → 함수형 프로그래밍에서는 잘못된 code로 인한 오동작의 의미가 아닌 실행 결과 상태의 변화를 일으키는 모든 것을 지칭함

→ 모듈화 수준이 높으면 재사용성이 높고 좋은 프로그래밍이라 할 수 있다.

→ 평가 시점이 무관하다는 특성으로 효율적인 로직을 구성하는 것이 함수형 프로그램의 궁극적인 패러다임

# 순수함수

- 동일한 인자가 주어졌을 때 **항상** 같은 값을 리턴하는 함수

    → side effect 가 없음

- Thread에 안전하고 병렬적인 계산이 가능
- 코드의 블록을 이해하기 위해 일련 상태 갱신을 따라갈 필요가 없고 국소 추론만으로도 코드 이해 가능
- 모듈적인 프로그램은 독립적으로 재사용할 수 있는 구성요소(component)로 구성

```swift
    // a와 b를 더하는 것 외의 어떤 side-effect도 발생시키지 않음
    func addValue(_ a: Int, b: Int) -> Int {
            return a + b
    }
    
    print(addValue(3, b: 5)) // 8
```
- 입력과 결과가 분리되어 있으며, 어떻게 사용되는 지에 대해 전혀 신경 쓰지않아도 되므로 재사용성이 높아진다.

### 순수 함수가 아닌 경우

- 변수가 포함되어 있을 경우

```swift
    var c = 10
    func addValue2(a: Int, b: Int, c: Int) -> Int {
            return a + b + c
    }
    
    print(addValue2(a: 3, b: 5, c: c)) // 18
    c = 20
    print(addValue2(a: 3, b: 5, c: c)) // 28
```

함수 내에서 외부의 c라는 **'변수'** 값이 변하면 결과 값도 달라지기 때문

c가 변하지 않는 **'상수'**라면 addValue2는 순수 함수이다.

외부의 값을 참조해도 리턴 값이 동일하다는 것을 보장됨

- 외부 상태에 영향을 미칠 경우(1)

```swift
    var c = 20
    func addValue3(a: Int, b: Int) -> Int {
            c = b  // 외부 상태에 영향을 미침 -> 부수효과(side-effect)
            return a + b
    }
    
    print(c) // 20
    print(addValue3(a: 3, b: 5)) // 8
    print(c) // 5
```

함수가 *외부의 값을 변경하는 코드*를 가지고 있기 때문
리턴하는 값이 항상 일정하더라도 외부의 상태를 변경하는 코드가 있으면 순수 함수가 아님.

- 외부 상태에 영향을 미칠 경우(2)

```swift
    class ObjectEx {
        var num = 10
    }
    
    func addVlaue4(obj: ObjectEx, b: Int) -> Int {
            obj.num += b
            return obj.num
    }
    
    let objEx = ObjectEx()
    print(objEx.num) // 10
    print(addValue4(obj: objEx, b: 5)) // 15
    print(objEx.num) // 15
```
객체를 인자로 받아서 그 상태를 변경시키는 코드를 가지고 있음

### 객체를 순수 함수로 나타내기
```swift
    class ObjectEx {
        var num = 10
    }
    
    func addValue5(obj: ObjectEx, b: Int) -> Int {
            return obj.num + b
    }
    
    let objEx = ObjectEx()
    print(objEx.num) // 10
    print(PureFunction.addValue5(obj: objEx, b: 5)) // 15
    print(objEx.num) // 10
```
ObjectEx의 num의 값만 참조 하는 식으로 작성한다.

> **순수 함수** → 외부의 상태를 변경하지 않으면서 동일한 인자에 대해 항상 똑같은 값을 리턴하는 함수
