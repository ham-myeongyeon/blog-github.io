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

메서드가 아닌 일반함수로 호출주체를 명시하지 않고 호출하면 this는 전역객체를 바라봅니다.
이는 객체 내부에서도 일반함수로 호출돼도 this는 전역객체에 바인딩 됩니다.

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

여기서 차이점은 객체의 메서드인 outer에서 this는 호출 주체가 있어 obj1을 바라보지만, innerFunc() 처럼 일반함수로 호출할때는 호출주체가 명시되지 않으므로 window가 찍히게 됩니다.

this는 실행 컨텍스트가 생성되는 시점에 바인딩을 합니다. 하지만 중요한점은 this는 함수를 실행하는 주변 환경은 중요하지 않고, 해당 함수를 호출하는 구문 앞에 점표기를 기준으로 결정됩니다.

### this를 바인딩하지 않는 함수

함수 내부에서 호출할때 this가 전역객체를 바라보는 문제를 보와하고자 ES6부터 화살표함수를 도입 했으며 this를 바인딩하지 않습니다.
화살표 함수에서 실행 컨텍스트가 생길때는 this바인딩 과정이 빠지게 되어 상위 스코프의 this를 그대로 활용하게 됩니다.

```
var obj = {
  outer: function () {
    console.log(this); // { outer: f }

    var innerFunc = () => {
      console.log(this); // { outer: f}
    }

    innerFunc();
  }
};

obj.outer();

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
