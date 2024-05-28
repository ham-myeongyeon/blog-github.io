---
title: 코어 자바스크립트, 프로토타입 이해하기
date: '2024-05-11T03:55:40'
# description: ''
---

# 프로토타입의 개념 이해

## constructor, prototype, instance

우선 생성자 함수를 호출하며 프로토 타입이 참조하게 되는 과정은 아래와 같습니다.

```jsx
var instance = new Constructor()
```

- 생성자 함수를 new연산자와 함께 호출
- Constructor에서 정의된 내용을 바탕으로 새로운 instance가 생성
- instance에는 **proto**라는 프로퍼티가 부여되고
- 이 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조하게 됩니다.

> \_ _proto_ \_

\_ _proto_ _는 브라우저가 [[prototype]]을 구현한 대상입니다. 명세에는 _ _proto_ \_를 통해서 직접 접근하는 것을 허용하지 않지만 기존 브라우저에서 접근하는 시도와 같은 레거시와의 호환 때문에 접근은 가능합니다.

대신 Object.getPrototypeof()/Object.create()등을 이용하는게 권장됩니다.

```jsx
var Person = function (name) {
  this._name = name
}

Person.protoype.getName = function () {
  return this._name
}
```

Person의 prototype에 getName을 지정했으므로 Person의 인스턴스는 getName을 호출할 수 있습니다.

```jsx
Person.prototype === suzi.__proto__ // true
```

이제 같은 prototype을 바라보기 때문에 결과는 true입니다.

```jsx
var suzi = new Person('Suzi')
suzi.__proto__.getName() // undefined
```

타입에러가 아닌 undefined를 출력했는데 이는 getName이 있다는 것이고 Person의 prototype을 참조한다는 것입니다.

다만 undefined를 출력한 이유는 this가 suzi.**proto**.getName();에서 suzi.**proto**를 바인딩 했기 때문에 undefined를 출력합니다.

## constructor 프로퍼티

# 프로토 타입 체인

## 메서드 오버라이드

## 프로토타입 체인

## 객체 전용 메서드의 예외사항

## 다중 프로토타입 체인
