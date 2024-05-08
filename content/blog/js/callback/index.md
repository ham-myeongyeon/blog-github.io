---
title: 코어 자바스크립트, 콜백함수 이해하기
date: '2024-05-06T03:57:40'
# description: ''
---

# 콜백함수?

---

콜백 함수는 다른 함수 또는 메서드에게 인자로 넘겨지며, **제어권**이 함께 위임된 함수를 말합니다.

# 제어권

---

## 호출 시점

```jsx
var count = 0

var cbFunc = function () {
  console.log(count)

  if (++count > 4) clearInterval(timer)
}

var timer = setInterval(cbFunc, 300)
```

cbFunc 콜백함수는 setInterval에 제어권을 넘겨 0.3초라는 호출시점에 대한 제어권을 넘겨 받습니다

## 인자

```jsx
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(currentValue, index);
});

var newArr = [10, 20, 30].map(function (index, currentValue) {
  console.log(index, currentValue);
});

Array.prototype.map(callback[, thisArg]);
callback: function (currentValue, index, array);
```

콜백함수는 인자에 대한 제어권도 넘깁니다. 위 코드처럼 콜백함수내에서 currentValue, index를 바꿔도 값은 변하지 않으며, 제어권을 넘겨받은 함수 map에 따라서 콜백함수 인자를 제어하게 됩니다.

그래서 네이밍이 관계없이 순서에 따라서만 바뀌게 됩니다.

## this

```jsx
Array.prototype.map = function (callback, thisArg) {
  var mappedArr = []

  for (var i = 0; i < this.length; i++) {
    var newValue = callback.call(thisArg || window, this[i], i, thisArg || this)

    mappedArr[i] = newValue
  }

  return mappedArr
}
```

this에 대한 제어권도 call/apply등을 통해서 this를 명시적으로 바인딩 할수 있습니다.

위 코드는 thisArg를 어떻게 바인딩 될 수 있는지 구현한 map 메서드 입니다.

# 콜백 함수는 함수다

```jsx
var obj = {
  vals: [1, , 2, 3],
  logValues: function (value, index) {
    console.log(this, value, index)
  },
}

obj.logValues(1, 2)
;[4, 5, 6].forEach(obj.logValues)
```

logValues는 메서드로서 호출되므로 첫번째 호출에서는 obj를 this로 바인딩 하지만, 두번째 호출 forEach의 콜백함수로 호출될 때는 window를 출력합니다.

이는 obj.logValues라는 메서드로 전달한것 이 아닌, obj.logValues가 가리키는 함수만 전달한것이기 때문입니다.

객체의 메서드를 콜백함수로 전달하더라도 함수로서 전달됩니다.

# 콜백 함수 내부의 this에 다른 값 바인딩하기

# 콜백 지옥과 비동기 제어
