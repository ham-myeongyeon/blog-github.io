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

```
setTimeout(function () {
  console.log(this);
}, 300);

[1, 2, 3, 4].forEach(function (x) {
  console.log(this, x);
});

document.body.innerHTML += `<button id='a'>클릭</button>`;
document.body.querySelector("#a").addEventListener("click", function (e) {
  console.log(this, e);
});
```

콜백함수의 종류에 따라 this가 어떻게 사용될지 결정됩니다. 위에 두함수는 일반함수처럼 호출되어 window가 출력되며, 이벤트리스너는 콜백함수를 호출할 때 자신의this를 상속하도록 되어 있어 .앞에 부분 button이 this로 출력됩니다.

## 생성자 함수 내부에서의 this

```
var Cat = function (name, age) {
  this.bark = '야옹';
  this.name = name;
  this.age = age;
};

var moca = new Cat('모카', 1);
```

생성자 함수는 공통된 성질을 가진 객체를 생성하는 함수 입니다. 호출시 함수 앞에 new를 붙여 호출하면 해당 함수가 생성자로서 동작하며 여기서 내부의 this는 새로 만들 인스턴스가 됩니다.

# 명시적으로 this를 바인딩하기

## call 메서드

call메서드는 함수내부에 this를 명시적으로 지정할 수 있습니다. call은 호출함수를 즉시 실행하도록 합니다. 첫번째 인자는 this를 명시하고, 두번째 인자부터는 호출할 함수의 인자로 들어가게 됩니다.

```
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3);  // window, 1, 2, 3
func.call({ x: 1 }, 4, 5, 6);  // { x : 1 }, 4, 5, 6
```

## apply 메서드

apply도 call 메서드처럼 this를 명시하는 방법으로, 첫번째 인자로 this를 받으며 두번째 인자는 배열로 호출할 함수의 인자로 요소로 들어가게 됩니다.

```
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3);  // window, 1, 2, 3
func.apply({ x: 1 }, [4, 5, 6]);  // { x : 1 }, 4, 5, 6

```

## call/apply 메서드 활용

### 유사배열객체(array-like object)에 배열메서드 적용

```
var obj = {
  0: "a",
  1: "b",
  2: "c",
  length: 3,
};

Array.prototype.push.call(obj, "d");
console.log(obj); // { "0": "a", "1": "b", "2": "c", "3": "d", "length": 4 }

var arr = Array.prototype.slice.call(obj);
console.log(arr); // [ "a", "b", "c", "d" ]
```

유사배열객체는 키가 양의 정수인 프로퍼티와 length프로퍼티의 값이 있는 배열구조와 유사한 객체를 말합니다.
여기서 유사배열객체는 배열의 메서드를 사용하지 못하는데, call이나 apply메서드를 이용해 배열로서 사용이 가능하게 됩니다.

```
function a() {
  var argv = Array.prototype.slice.call(arguments);

  argv.forEach(function (arg) {
    console.log(arg);
  });
}

a(1, 2, 3);

document.body.innerHTML = `<div>a</div><div>b</div<div>c</div>`;

var nodeList = document.querySelectorAll("div");
var nodeArr = Array.prototype.slice.call(nodeList);

nodeArr.forEach(function (node) {
  console.log(node);
});
```

arguments, nodeList등 모두 유사 배열객체입니다. slice.call 메서드를 통해서 배열로 복사한 후에 배열로서 사용이 가능하게 됩니다.

```
var str = "abc def";

Array.prototype.push.call(str, "pushed string"); // 에러 발생

console.log(Array.prototype.concat.call(str, "string"));
<!-- [
    "abc def",
    "string"
] -->
```

문자열의 경우 length가 읽기전용 프로퍼티이므로 push와 같이 변경을 가하면 에러가 발생합니다. concat의 경우 새로운 배열을 반환하므로 에러는 발생하지 않지만 원한는 값이 안나오게됩니다.
따라서 call/apply메서든는 본래 목적인 this를 지정하는 방법으로 사용하고, ES6문법이전에 유사배열객체등을 복사하기위해 사용하는 방법정도로 slice.call메서드를 활용하면 됩니다.

ES6부터는 유사배열객체 또는 순회가능한 종류의 데이터 타입을 배열로 전환하는 Array.from메서드가 도입됐으니 이를 활용하면 될 것 같습니다.

```
var obj = {
  0: "a",
  1: "b",
  2: "c",
  length: 3,
};

var arr = Array.from(obj);
console.log(arr); // [ 'a', 'b', 'c' ]
```

### 생성자 내부에서 다른 생성자 호출

```
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}

function Employee(name, gender, company) {
  Person.call(this, name, gender);
  this.company = company;
}

var mj = new Student('명진', 'male', '서강대');
var musk = new Employee('머스크', 'male', '테슬라');

```

이렇게 하여 공통적인 속성을 Person.call을 통해 호출하여, 인스턴스로 만들어질 this를 Person의 속성을 추가하여 코드의 반복을 줄일수 있습니다.

### 여러 인수를 묶어서 하나의 배열로 전달하고 싶을때 (apply)

```
var numbers = [10, 20, 3, 16];

var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);

console.log(max, min);
```

numbers의 배열을 펼쳐야 하지만 apply를 통해서 배열 자체로 넘길수 있게 됩니다.
ES6부터는 스프레드 연산자로 간단하게 전달할 수 있습니다.

```
var numbers = [10, 20, 3, 16];

var max = Math.max(...numbers);
var min = Math.min(...numbers);

console.log(max, min);
```

## bind 메서드

bind는 ES5에서 추가된 기능으로, call과 비슷하지만 함수를 호출하지 않고 넘겨 받은 this를 바탕으로 새로운 함수를 반환합니다. 새로운 bind된 함수에 인자를 넘기면 인수들이 뒤이어 등록되어 사용됩니다.

```
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};

func(1, 2, 3, 4); // window, 1, 2, 3, 4

var bindFunc = func.bind({ x: 1 });

bindFunc(1, 2, 3, 4); // { x: 1 }, 1, 2, 3, 4


var bindFunc2 = func.bind({ x: 2 }, 4, 5);

bindFunc2(6, 7); // { x: 2 }, 4, 5, 6, 7
```

bindFunc2 함수처럼 this뿐만 아니라 뒤에 매개변수도 같이 전달하여 사용가능합니다.

### name 프로퍼티

bind 메서드를 통해서 만들어진 함수는 name프로퍼티 앞에 bound가 붙어서 나오게 됩니다.

```
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};

var bindFunc = func.bind({ x: 1 }, 4, 5);

console.log(func.name); // func
console.log(bindFunc.name); // bound func
```

### 상위 컨텍스트의 this를 내부함수나 콜백함수에 전달하기

```
var obj = {
  outer: function () {
    console.log(this);

    var innerFunc = function () {
      console.log(this);
    };

    innerFunc.call(this);
  },
};

obj.outer();
```

```
var obj = {
  outer: function () {
    console.log(this);

    var innerFunc = function () {
      console.log(this);
    }.bind(this);

    innerFunc();
  }
}

obj.outer();
```

위 두 코드처럼 innerFunc 일반함수를 상위 컨텍스트의this에 바인딩하여 호출할 수 있습니다.

## 화살표 함수에서 예외사항

## 별도의 인자로 this를 받는 경우
