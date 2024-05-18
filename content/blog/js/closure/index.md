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

일반적인 콜백 함수 코드

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', function () {
    alert('your choice is ' + fruit)
  })
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

var alertFruit = function (fruit) {
  alert('your choice is ' + fruit)
}

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', alertFruit)
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

위 코드는 반복을 피해 alertFruit함수를 따로 선언해서 사용했지만, addEventListener의 기본적으로 첫번째인자를 이벤트 객체를 활용하기 때문에 문제가 발생합니다.

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

var alertFruit = function (fruit) {
  alert('your choice is ' + fruit)
}

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', alertFruit.bind(null, fruit))
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

bind를 통해서 첫번째 인자에 원하는 fruit를 전달하지만, this가 null로 바인딩이 됩니다.

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

var alertFruitBuilder = function (fruit) {
  return function () {
    alert('your choice is ' + fruit)
  }
}

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', alertFruitBuilder(fruit))
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

위 문제점들을 모두 개선하기 위해 alertFruitBuilder를 선언하여 함수를 반환하며 이 반환된 함수는 클로저가 형성되어 추후에 fruit를 참조하여 활용하게 됩니다.

## 접근 권한 제어

자바스크립트에서는 다른 언어와 달리 변수에 대해 접근 권한 기능이 따로 없습니다. 접근 권한을 구현하기 위해 클로저를 이용하여 함수 차원에서 변수에 대한 접근을 차단할 수 있습니다.

클로저를 통해서 외부 함수로 반환할 때 반환한 변수는 공개, 반환하지 않은 변수는 접근을 제한할 수 있습니다.

```jsx
var car = {
  fuel: Math.ceil(Math.random() * 10 + 10),
  power: Math.ceil(Math.random() * 3 + 2),
  moved: 0,
  run: function () {
    var km = Math.ceil(Math.random() * 6)
    var wasteFuel = km / this.power

    if (this.fuel < wasteFuel) {
      console.log('이동불가')
      return
    }

    this.fuel -= wasteFuel
    this.moved += km
    console.log(km + 'km dlehd (총' + this.moved + 'km')
  },
}
```

예를들어 일반적인 객체는 fuel속성에 접근하여 변경이 가능하게 됩니다.

```jsx
var createCar = function () {
  var fuel = Math.ceil(Math.random() * 10 + 10)
  var power = Math.ceil(Math.random() * 3 + 2)
  var moved = 0

  return {
    get moved() {
      return moved
    },
    run: function () {
      var km = Math.ceil(Math.random() * 6)
      var wasteFuel = km / power

      if (fuel < wasteFuel) {
        console.log('이동불가')
        return
      }

      fuel -= wasteFuel
      moved += km
      console.log(km + 'km dlehd (총' + moved + 'km')
    },
  }
}

var car = createCar()
```

fuel와 같은 속성들은 외부로 노출하지 않으므로서 비공개로 하고, moved는 getter을 통해서 읽기 전용으로 설정했습니다.

접근은 막았지만 car.fuel =1000;와 같은 재할당으로 변경이 가능하게 됩니다.

```jsx
var createCar = function () {
  // ...

  var publicMembers = {
    fuel: Math.ceil(Math.random() * 10 + 10),
    power: Math.ceil(Math.random() * 3 + 2),
  }

  Object.freeze(publicMembers)

  return publicMembers
}
```

공개는 하지만 변경을 피하고 싶은 변수들은 위 코드처럼 Object.freeze를 통해서 변경을 막은뒤 반환하여 접근하게 할수 있습니다.

## 부분 적용 함수

```jsx
var add = function () {
  var result = 0

  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i]
  }

  return result
}

var addPartial = add.bind(null, 1, 2, 3, 4, 5)
console.log(addPartial(6, 7, 8, 9, 10))
```

```jsx
var partial = function () {
  var originalPartialArgs = arguments
  var func = originalPartialArgs[0]

  if (typeof func !== 'function') {
    throw new Error('is not function')
  }

  return function () {
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1)
    var restArgs = Array.prototype.slice.call(arguments)

    return func.apply(this, partialArgs.concat(restArgs))
  }
}

var add = function () {
  var result = 0

  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i]
  }

  return result
}

var addPartial = partial(add, 1, 2, 3, 4, 5)
console.log(addPartial(6, 7, 8, 9, 10))
```

## 커링 함수
