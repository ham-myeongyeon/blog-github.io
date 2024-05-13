---
title: 코어 자바스크립트, 클로저 이해하기
date: '2024-05-11T03:55:40'
# description: ''
---

# 클로저의 의미 및 원리 이해

일반적으로 내부 함수는 외부 함수의 변수를 참조할 수 있습니다.

```jsx
var outer = function () {
  var a = 1

  var inner = function () {
    return ++a
  }

  return inner()
}

var outer2 = outer()

console.log(outer2)
```

위 코드처럼 내부에 선언된 inner가 LexicalEnvironment의 outerEnvironmentReference에 접근하여 외부 함수 변수a를 참조합니다.

클로저는 내부 함수가 외부 함수로 전달되어질 때, 외부 함수의 실행 컨텍스트가 종료된 뒤에도 내부 함수가 외부 함수의 참조한 변수를 유지하는 현상을 말합니다.

```jsx
var outer = function () {
  var a = 1

  var inner = function () {
    return ++a
  }

  return inner
}

var outer2 = outer()

console.log(outer2()) // 2
console.log(outer2()) // 3
```

위 코드를 예시로 보면 outer를 호출하고 inner함수를 반환함으로서, outer함수에 대한 실행 컨텍스트는 종료되지만 변수 a에 접근하여 값을 증가 시키고 유지합니다.

이는 가비지 컬렉팅의 방식때문인데, 하나라도 참조하고 있는 변수가 있다면 추후에 사용될 수 있기 때문에 수거되지 않습니다.

outer()에서 inner가 반환되고 inner가 outer2를 통해 참조되기 때문에 언제라도 호출되고 이용될 가능성이 있어서 수거되지 않고, a도 같이 수거되지 않아 값을 유지하게 됩니다.

가비지 컬렉팅 방식 때문에 클로저가 발생하게 됩니다.

꼭 return으로 외부에 함수를 전달하지 않아도 콜백함수로 전달해도 클로저가 발생합니다.

```jsx
;(function () {
  var a = 0
  var intervalId = null

  var inner = function () {
    if (++a > 10) {
      clearInterval(intervalId)
    }

    console.log(a)
  }

  intervalId = setInterval(inner, 1000)
})()
```

# 클로저와 메모리 관리

클로저는 객체 지향 프로그래밍의 맥락으로 공통적인 부분을 연결시키는 특성이 있습니다. 다만 참조 카운트가 0이 되지 않아 메모리 누수가 발생할 위험이 있어 관리가 필요합니다.

관리 방법은 참조형이 아닌 기본형 데이터를 할당하면 됩니다.

```jsx
var outer = (function () {
  var a = 1

  var inner = function () {
    return ++a
  }

  return inner
})()

console.log(outer())
outer = null

;(function () {
  var a = 0
  var intervalId = null
  var inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId)
      inner = null
    }

    console.log(a)
  }

  intervalId = setInterval(inner, 1000)
})()
```

# 클로저 활용 사례

## 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

## 접근 권한 제어

## 부분 적용 함수

## 커링 함수