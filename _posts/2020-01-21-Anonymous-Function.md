---
title: "익명 함수(Anonymous Function)"
category: "Functional Programming"
---

# 익명함수

- 일반 함수의 경우 func + 함수 이름을 선언하고 사용
- swift에서는 `클로저`에 해당
- `in` 키워드를 통해 전달 인자와 실행 코드를 구분
- 함수의 이름을 선언하지 않고 바로 몸체만 만들어 일회용 함수로 사용

```swift
    func test(_ isFinish: Bool) -> Void {
            print("isFinish: \(isFinish)")
    }
```
위의 함수를  익명 함수로 바꾸면
```swift
    let test = { (isFinish: Bool) -> Void in
            print("isFinish: \(isFinish)")
    }
    
    {
        (매개변수) -> (반환타입) in
        실행구문
    }
```
가 된다.

- 컴파일러가 반환 타입을 미리 알고 있다면 반환 타입도 **생락이 가능**하다
```swift
    let test = { (isFinish: Bool) in
            print("isFinish: \(isFinish)")
    }
    { (매개변수) in
        실행구문
    }
```
경량 문법이 있지만 그것은 클로저에서 공부하시면 될듯

 * 클로저 내부 코드가 한 줄이라면 `return` 생략 가능

`@escaping`을 붙이지 않아도 탈출하는 방법
