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

```jsx
var suzi = new Person('Suzi', 28)
suzi.getName() // Suzi
```

**proto**를 빼면 this가 instance에 바인딩되어 Suzi가 나오는 건 맞지만 생략해도 접근이 가능항 이유는 **proto**가 생략 가능한 프로퍼티이기 때문입니다.

```jsx
suzi.__proto__.getName
=> suzi(.__proto__).getName
=> suzi.getName
```

위에 3가지모두 같은 의미입니다.

따라서 **proto**를 생략하여 접근하면 this는 인스턴스에 바인딩이 되고, prototype의 속성을 참조하여 사용할 수 있게 됩니다.

프로토타입의 과정은

- 자바스크립트는 함수에 자동으로 객체인 prototype프로퍼티를 생성합니다.
- 해당함수를 생성자 함수로, new와 호출하면 인스턴스를 생성하고, 그 인스턴스에는 숨겨진 **proto**가 자동으로 생성됩니다.
- **proto**는 생성자 함수의 prototype프로퍼티를 참조합니다.
- 이 **proto**는 생략가능하도록 되어 있습니다.

최종적으로 생성자 함수의 prototype에 어떤 메서드, 프로퍼티가 있다면 인스턴스가 자기의 메서드나 프로퍼티처럼 사용할 수 있습니다.

## constructor 프로퍼티

생성자 함수의 prototype 객체 내부에도 constructor라는 프로퍼티가 있습니다. 인스턴스의 **proto**도 가지고 있는데 이는 그 원형이 무엇인지를 알수 있게 해줍니다.

```jsx
var arr = [1, 2]

Array.prototype.constructor === Array // true
arr.__proto__.constructor === Array // true
arr.constructor === Array // true

var arr2 = new arr.constructor(3, 4)
console.log(arr2) // [3, 4]
```

**proto**의 constructor를 통해서 Array를 참조할 수 있습니다. 그리고 **proto**는 생략이 가능하기 때문에 인스턴스는 바로 constructor를 참조할수 있게 됩니다.

constructor은 읽기 전용 속성이 부여된 예외적인 경우(기본형 리터럴 변수 - number, string, boolean)을 제외 하고는 값을 바꿀수 있습니다.

```jsx
var NewConstructor = function () {
  console.log('new constructor')
}

var dataTypes = [1, 'test', true, {}, [], function () {}, new Number()]

dataTypes.forEach(data => {
  data.constructor = NewConstructor
  console.log(data.constructor.name, '&', data instanceof NewConstructor)
})
```

1, ‘test’, true를 제외한 모든 데이터의 출력결과는 `NewConstructor & false`입니다. 즉 constructor의 참조하는 대상은 변경되더라도 원형이나 데이터 타입이 바뀌지는 않습니다.
따라서 아래의 코드들은 서로 같은 대상을 가리킵니다.

```jsx
;[Constructor][instance].__proto__.constructor[instance].constructor
Object.getPrototypeOf([instance]).constructor[Constructor].prototype.constructor
```

```jsx
;[Constructor].prototype[instance].__proto__[instance]
Object.getPrototypeOf([instance])
```

# 프로토 타입 체인

## 메서드 오버라이드

```jsx
var Person = function (name) {
  this.name = name
}

Person.prototype.getName = function () {
  return this.name
}

var iu = new Person('지금')
iu.getName = function () {
  return '바로' + this.name
}
console.log(iu.getName()) // 바로 지금
```

getName을 할당하고 메서드를 호출하면 prototype이 아닌 새로 할당한 메서드에서 결과를 출력합니다.

여기서 일어난 현상은 메서드 오버라이드인데 원본을 제거하지 않고 대체하는 것으로, 원본이 있는 상태에서 다른 대상을 그위에 얹는 것 같은 상태를 의미합니다.

이 코드에서 getName을 찾는 방식은 가장 가까운 대상부터 찾는데, 우선 자신의 프로퍼티를 검색하고 없다면 **proto**를 검색하게 됩니다.

이러한 방식으로 원본을 대체하지 않고, 메서드위에 메서드를 얹는 것처럼 검색순서만 달라지는 메서드 오버라이딩 현상이 발생합니다.

원본을 대체하지 않았기 때문에 원본 메서드 prototype.getName에 접근할 수 있게 됩니다.

`iu.__proto__.getName()` 여기서는 this 문제가 생기므로 먼저 할당하고 접근하게 되면

```jsx
Person.prototype.name = '이 지금'
console.oog(iu.__proto__.getName()) // 이 지금
```

오버라이딩 되기전 메서드에 접근할 수 있게 됩니다.

## 프로토타입 체인

```jsx
console.dir([1, 2])
```

배열의 내부구조를 보게 되면 **proto**안에는 객체의 **proto**와 동일한 **proto**가 있습니다.

왜냐하면 prototype 객체가 자체가 ‘객체’이기 때문입니다. 그래서 기본적으로 모든 객체의 **proto**에는 Object.prototype이 연결됩니다.

따라서 **proto**는 생략이 가능하므로 배열의 경우 Array.prototype뿐만 아니라, Object.prototype 내부의 메서드도 자신의 것처럼 사용할 수 있습니다.

```jsx
var arr = [1, 2];
arr(.__proto__).push(3);
arr(.__proto__)(.__proto__).hasOwnProperty(2);
```

어떤 데이터의 **proto** 프로퍼티 내부에 다시 **proto**프로퍼티가 연쇄적으로 이어진 것을 프로토타입 체인 이라고 하며, 이 체인을 따라가며 검색하는 것을 프로토타입 체이닝이라고 합니다.

그래서 프로토타입 체이닝 과정은 메서드 오버라이드와 동일한 맥락입니다.

메서드를 호출할 때 자신의 프로퍼티를 검색하고 있다면 사용, 없다면 **proto**를 검색합니다. 동일하게 또 없다면 다시 **proto**를 검색하여 찾고 결국 찾아낸다면 실행하는 방식입니다.

```jsx
var arr = [1, 2]

Array.prototype.toString.call(arr) // 1, 2
Object.prototype.toString.call(arr) // [object Array]
arr.toString() // 1, 2

arr.toString = function () {
  return this.join('_')
}
arr.toString() // 1_2
```

위 코드를 보면 프로토 타입체인 과정에 따라 arr.toString()을 호출하면 Array prototype의 toString()메서드의 결과와 같게 호출됩니다.

또 arr.toString에 함수를 할당하면 검색순위에 따라 결과 값이 변경 됩니다.

## 객체 전용 메서드의 예외사항

어떤 생성자 함수이든 prototype은 반드시 객체이기 때문에 Object.prototype이 언제나 프로토타입 체인의 최상단에 존재합니다. 그래서 객체에서만 사용할 메서드를 Obejct.prototype에 정의한다면 다른 데이터 타입에서도 접근이 가능하기 때문에 Object.prototype내부에 정의하지 않습니다.

```jsx
Object.prototype.getEntries = function () {
  var res = []

  for (var prop in this) {
    if (this.hasOwnPropertyz(prop)) {
      res.push([prop, this[prop]])
    }
  }

  return res
}

var data = [
  ['number', 345],
  ['string', 'abc'],
]
```

위에 코드를 실행하게 되면 프로토타입 체이닝 과정에 따라 Object.prototype에 접근하여, 오류를 발생시키지 않고 실행됩니다.

따라서 객체 전용 메서드들은 Object에 스태틱 메서드로 부여하게 됩니다. 객체만을 대상으로 동작하는 객체 전용 메서드들은 메서드명 앞의 대상이 곧 this가 되는 방식 대신, 대상 인스턴스를 인자로 직접 주입해야 하는 방식으로 구현돼 있습니다.

반대로 Object.prototype에는 어떤 데이터에서도 활용할 수 있는 범용적인 메서드들만이 있습니다.

> Object.create()

Object.create() 메서드는 인자를 프로토타입으로 하는 객체를 만듭니다. 따라서 **proto**가 없는 객체를 만든다면 Object.create(null)를 호출하여 만들수 있습니다.

```jsx
var _proto = Object.create(null)
_proto.getValue = function (key) {
  return this[key]
}

var obj = Object.create(_proto)
obj.a = 1

console.log(obj.getValue('a'))
console.dir(obj)
```

먼저 \_proto 프로토타입이 없는 객체를 반환하여 할당합니다. 다시 \_proto에 getValue메서드를 추가하고 이 \_proto를 인자로 하여 Object.create()를 호출합니다. 그러면 \_proto를 프로토타입으로 가지는 obj를 만들게 됩니다.

이 obj에 getValue에 접근하면 메서드를 호출하게 되며, console.dir로 구조를 출력하면 **proto**에는 getValue만 있게 됩니다.

## 다중 프로토타입 체인

**proto**로 연결하여 체인 관계를 이어나가는 방법으로, 다른 언어의 클래스와 비슷하게 인스턴스를 구성할 수 있습니다.

```
var Grade = function () {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
}

var g = new Grade(100, 80);
```

변수 g는 Grade의 인스턴스로 유사배열객체입니다.

```
Grade.prototype = [];
```

이렇게 되면 Grade.prototype이 배열의 인스턴스를 바라보며 **proto** 연결로 다중 프로토타입체인 형성으로 배열의 메서드를 사용할 수 있게 됩니다.
