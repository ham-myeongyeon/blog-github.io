---
title: 코어 자바스크립트, 클래스 이해하기
date: '2024-06-10T03:55:40'
# description: ''
---

# 클래스와 인스턴스의 개념 이해

클래스는 과일, 음식과 같은 집단과 같은 추상적인 개념이며, 이러한 개념을 구체화 할 틀같은 대상입니다. 그리고 클래스는 상위/하위 개념으로 상속되며 하위 클래스로 더 구체화 될 수 있습니다.

하지만 하위 클래스로 속성이 추가되고 상속되더라도 구체화 될 뿐 추상적인 개념 입니다.

이 추상적인 클래스를 구현한 실존하는 구체적인 예시를 인스턴스라고 합니다.

현실에서는 인스턴스들로 부터 공통적인 요소를 지니는 개체들을 클래스로 정의하지만, 프로그래밍에서는 클래스가 정의돼야만 공통적인 요소를 지니는 인스턴스를 생성할 수 있습니다.

# 자바스크립트의 클래스

프로토타입을 클래스 관점에서 접근하면 new 연산자와 함께 Array를 호출하면 Array를 일종의 클래스로 보고 Array prototype 객체 내부요소들이 인스턴스에 상속되는 것처럼 볼 수 있습니다.

인스턴스에 상속되는지 여부에 따라 스태틱 멤버(static member)와 인스턴스 멤버(instance member)로 나뉩니다. 그리고 자바스크립트에서는 프로토타입 및 인스턴스의 메서드가 같이 인스턴스에 적용돼므로 스태틱 메서드, 프로토타입 메서드, 인스턴스 메서드로 나눌수 있습니다.

```jsx
// 생성자
var Rectangle = function (width, height) {
  this.width = width
  this.height = height
}
// (프로토타입)메서드
Rectangle.prototype.getArea = function () {
  return this.width * this.height
}
// 스태틱 메서드
Rectangle.isRectangle = function (instance) {
  return (
    instance instanceof Rectangle && instance.width > 0 && instance.height > 0
  )
}

var rect1 = new Rectangle(3, 4)
console.log(rect1.getArea())
console.log(rect1.isRectangle(rect1)) // Error
console.log(Rectangle.isRectangle(rect1))
```

# 클래스 상속

## 기본 구현

- 일반적인 프로토타입 상속

```jsx
var Grade = function () {
  var args = Array.prototype.slice.call(arguments)
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i]
  }
  this.length = args.length
}

Grade.prototype = []
var g = new Grade(100, 80)

g.push(90).console.log(g)

delete g.length
g.push(70)
console.log(g)
```

여기서 문제점은 기존 인스턴스에서 상속받은 length프로퍼티는 삭제가 가능하기 때문에 지워지고 그다음 push를 실행하게 되면 프로토타입 체인을 따라 검색하며 빈 배열의 length를 참고하며 push하게 됩니다.

그러면 length는 1이 되며 빈 배열에 70을 추가하게 되어 맨앞에 70이 들어가게 됩니다.

이 방법은 클래스에 있는 값이 인스턴스의 동작에 영향을 주므로 오류가 발생할 가능성이 높습니다.

## 클래스가 구체적인 데이터를 지니지 않게 하는 방법

- 더글라스 크락포드가 제시한, 빈 생성자 함수(Bridge)로 연결하는 방법

```jsx
var Rectangle = function (width, height) {
  this.width = width
  this.height = height
}
Rectangle.prototype.getArea = function () {
  return this.width * this.height
}
var Square = function (width) {
  Rectangle.call(this, width, height)
}

var Bridge = function () {}
Bridge.prototype = Rectangle.prototype
Square.prototype = new Bridge()
Object.freeze(Square.prototype)
```

Bridge라는 빈 함수를 만들고 Bridge.prototype이 Rectangle.prototype을 참조하게 한 다음, Square.prototype에 new Bridge()로 할당하면 Rectangle자리에 Bridge가 대체하게 됩니다. 이로써 인스턴슬를 제외한 프로토타입 체인 경로상에는 구체적인 데이터가 남지않게 됩니다.

- Object.create를 활용하는 방법

```jsx
Square.prototype = Object.create(Rectangle.prototype)
Object.freeze(Square.prototype)
```

Object.create로 Rectangle.prototype을 **proto**로 가지는 객체를 Square.prototype에 연결합니다.

이렇게 하면 인스턴스의 구체적인 데이터도 남기지 않으며 연결할 수 있습니다.

## constructor 복구하기

위 방법 모두 prototype상속은 이루어졌지만 Subclass의 인스턴스의 constructor은 SuperClass를 가리키게 됩니다. 정확히는 SupaerClass.prototype의 constructor를 여전히 가리키는 상태입니다.

따라서 명시적으로 constructor를 지정해야 합니다.

```jsx
Square.prototype = Object.create(Rectangle.prototype)
// 생성자 함수 연결
Square.prottype.constructor = Square

Object.freeze(Square.prototype)
```
