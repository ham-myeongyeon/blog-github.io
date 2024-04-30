---
title: 코어 자바스크립트, this 이해하기
date: '2024-04-28T03:57:40'
# description: ''
---

# 상황별 this

this는 실행 컨텍스트가 생성될 때 결정되며 이 실행 컨텍스트는 함수를 호출할 때 생성되므로, this는 함수를 호출하는 시점에 결정됩니다.

## 전역공간

전역공간에서는 전역 컨텍스트를 생성하는 주체가 전역 객체이기 때문에 this는 전역객체를 가리킵니다.
브라우저 환경에서는 window이고, Node.js환경에서는 global입니다.

자바스크립트의 모든 변수는 실질적으로 특정 객체의 프로퍼티로 동작하기 때문에, 전역변수를 선언하면 전역객체의 프로퍼티로 할당됩니다. 여기서 특정 객체는 실행 컨텍스트의 LexicalEnvironment입니다.

## 메서드로서 호출할 때, 그 메서드 내부에서의 this

### 함수, 메서드

자바스크립트에서 함수를 실행하는 방법으로는 두 가지가 있는데, 첫번째는 함수로 선언하여 실행하는 방법, 두번째는 객체의 메서드로서 호출하는 방법으로 나뉩니다.
구분하는 방법은 단순히 메서드 앞에 .이 붙어 있으면 메서드, 단독적으로 호출되면 함수로서 호출된 경우입니다.

### 메서드 내부에서의 this

this는 호출한 주체에 대한 정보가 담기므로 호출한 .앞에 부분이 this가 됩니다.

```
var obj = {
  methodA: function () { console.log(this) },
  inner: {
    methodB: function () { console.log(this) }
  }
};

obj.methodA(); // obj
obj.inner.methodB; // obj.inner

```

## 함수로서 호출할 때, 그 함수 내부에서의 this

### 함수 내부에서의 this

메서드가 아닌 전역객체에서 호출주체를 명시하지 않고 함수를 호출하여 this는 전역객체를 바라봅니다.

### 메서드의 내부함수에서의 this

```
var obj1 = {
  outer: function () {
    console.log(this); // obj1

    var innerFunc = function () {
      console.log(this);
    }

    innerFunc(); // window

    var obj2 = {
      innerMethod: innerFunc
    };

    obj2.innerMethod(); // obj2
  }
};

obj1.outer();
```

## 콜백 함수 호출할 때, 그 함수 내부에서의 this

## 생성자 함수 내부에서의 this

# this를 바인딩하기

## call 메서드

## apply 메서드

## call/apply 메서드 활용

## bind 메서드

## 화살표 함수에서 예외사항

## 별도의 인자로 this를 받는 경우
