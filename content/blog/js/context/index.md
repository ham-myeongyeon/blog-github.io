---
title: 코어 자바스크립트, 실행 컨텍스트 이해하기
date: '2024-04-19T08:16:40.250109Z'
# description: ''
---

# 실행 컨텍스트

실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체 입니다.
실행 컨텍스트는 함수를 실행하여 구성하며, 스택 자료구조 형태로 가장 최신 정보가 맨위로 쌓여 나가는 형태 입니다.

```
var a = 1;
function outer() {
  function inner() {
    console.log(a);
    var a = 3;
  }

  inner();
  console.log(a);
}

outer();
console.log(a);
```

위 코드를 예시로 실행 한다면 과정은 아래와 같습니다.

- 먼저 코드를 실행하는 순간 전역 실행 컨텍스트가 생기며 콜스택에 담깁니다. 이때 변수 a와 outer함수에 대한 정보가 담깁니다.
- 그 뒤 outer() 함수 호출부를 만나면 outer에 대한 실행 컨텍스트가 생성되며 콜스택에 담깁니다. 이때 inner 함수에 대한 정보가 담깁니다.
- 이제 최신 콜스택에 outer 실행 컨텍스트가 놓인 상태이므로 그 이전 컨텍스트인 전역 컨텍스트는 실행을 중단합니다.
- outer실행 컨텍스트와 관련된 코드 outer 함수 내부를 순차적으로 실행합니다.
- inner함수 호출부를 만나면 outer 컨텍스트와 관련된 코드의 실행을 일시 중단합니다.
- 다시 inner함수에 대한 실행 컨텍스트가 생성되며 최상단에 콜스택이 담기며 inner컨텍스트와 관련된 코드를 실행합니다.
- 변수 a를 할당하며 inner함수가 종료되며 inner 실행 컨텍스트가 콜 스택에서 제거됩니다.
- 다시 outer에서 실행 중단되었던 부분 console.log(a)을 통해 a변수를 출력합니다.
- outer 실행 컨텍스트가 종료되며 그 이전 컨텍스트인 전역 컨텍스트에서 일시 중지되었던 부분부터 다시 실행합니다.
- 마지막으로 console.log(a)를 통해 a를 출력하고 종료됩니다.

실행 컨텍스트를 기준으로 코드 실행과정을 정리하자면
각 함수 호출을 기준으로 실행 컨텍스트가 생성되며, 해당 컨텍스트에는 코드실행에 필요한 환경 정보가 담깁니다.
생성된 최신 실행 컨텍스트는 콜스택 최상단에 담기며, 담기는 순간이 자바스크립트 엔진이 코드에 관여하는 시점입니다. 따라서 코드는 함수 호출부를 만나면 일시 중지되며 최신 실행 컨텍스트를 순서로 실행해 나갑니다.
최신 컨텍스트가 실행되고 마치면 컨텍스트는 사라지고 그 이전의 컨텍스트에서 일시 중지된 부분부터 코드를 실행해 나갑니다.

실행 컨텍스트 객체에는 아래 항목들이 담기게 됩니다.

- VariableEnvironment
- LexicalEnvironment
- ThisBinding

# VariableEnvironment

VariableEnvironment, LexicalEnvironment 모두 environmentRecord, outerEnvironment로 구성되어 있으며 초기화 과정에 담기는 내용은 같습니다.
차이점은 VariableEnvironment은 실행 컨텍스트를 생성할 때 스냅샷을 유지하는 점입니다.
생성 과정은 먼저 VariableEnvironment에서 정보를 먼저 담은 다음에 복사하여 LexicalEnvironment를 만들며 이후에는 주로 LexicalEnvironment를 활용합니다.

# LexicalEnvironment

단어적으로는 사전적 환경이라는 의미인데, 컨텍스트 내부의 식별자와 참조하는 외부 정보로 구성되어 있는 환경으로 해석하면될 것 같습니다.

## environmentRecord와 호이스팅

environmentRecord에는 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장됩니다. 식별자 정보로는 선언한 함수, var로 선언된 변수,함수에 지정된 매개변수 식별자 등이 있으며, 컨텍스트 내부를 훑어 나가며 순서대로 수집합니다.

코드 실행전에 변수정보들을 미리 저장하는데, 이때 미리 저장하므로 변수 정보들을 상위로 올려놓아도 실제 실행과정과 해석이 같게 됩니다. 이러한 현상을 호이스팅이라고 하며 실제로는 끌어올리지 않지만 해석의 편의를 위해 이렇게 간주하여 해석합니다.

### 호이스팅 과정

원본 코드

```
function a(x) {
  console.log(x);
  var x;
  console.log(x);
  var x = 2;
  console.log(x);
}

a(1);
```

environmentRecord는 할당은 관여하지 않고 오직 어떤 식별자가 있는지에만 관심이 있어 변수명만 끌어올린다고 생각하면 됩니다.

호이스팅이 적용된 코드

```
function a() {
  var x; // 매개변수 x
  var x;
  var x;

  x = 1; // 매개변수 x 할당과정

  console.log(x);
  console.log(x);

  x = 2;
  console.log(x);
}

a(1);
```

코드 실행 과정은

- 변수 x를 선언하면, 식별자를 x로 하는 변수영역의 메모리 공간을 확보하여 저장합니다.
- 그 다음 변수 x선언은 이미 x가 선언되어 있으므로 무시합니다.
- 1이라는 값을 데이터영역의 메모리 공간에 저장하고 이 주소값을 변수 x에 저장합니다.
- x의 값 1을 출력합니다.
- 2라는 값을 데이터영역의 메모리 공간에 저장하고 이 주소값을 변수 x에 저장합니다.
- 이제 x는 2라는 값을 바라보기 때문에 2를 출력하며 실행 컨텍스트가 콜 스택에서 제거되며 종료됩니다.

### 함수 선언의 호이스팅

```
function a() {
  console.log(b);
  var b = 'bbb';
  console.log(b);

  function b() { };

  console.log(b);
}

a();
```

호이스팅이 적용된 코드

```
function a() {
  var b;
  function b() {};

  console.log(b);
  b = 'bbb';
  console.log(b);
  console.log(b);
}

a();
```

똑같이 변수b가 올라가게 되고 추가로 함수 선언문도 저장되므로 올라가게 됩니다.
또 function b() {}; 부분은 var b = function b() {} 처럼 변수b에 할당된것으로 간주할수 있습니다.

위에 코드 실행과정과 같으며 다른 부분은 함수 선언부 입니다.
전에 선언된 변수b가 있으므로 선언과정은 무시하며 함수는 별도의 메모리영역에 저장되며 b는 함수b를 가리킵니다.

### 함수 선언문 과 함수 표현식

둘의 중요한 차이점은 호이스팅에 있습니다. 함수 선언문은 호이스팅이 되는 반면, 함수 표현식은 변수에 할당하여 표현하므로 변수만 호이스팅되고 함수는 호이스팅 되지 않습니다.

```
console.log(sum(1, 2));
console.log(multiply(3, 4));

function sum(a, b) {
  return a + b;
}

var multiply = function (a, b) {
  return a * b;
}
```

호이스팅이 적용된 코드

```
var sum = function sum(a, b) {
  return a + b;
}
var multiply;

console.log(sum(1, 2));
console.log(multiply(3, 4));

multiply = function (a, b) {
  return a * b;
}
```

- sum이라는 식별자로 하는 변수영역의 메모리 공간을 확보하며 저장합니다.
- multiply라는 식별자로 하는 변수영역의 메모리 공간을 확보하며 저장합나다.
- sum 함수를 데이터 영역의 메모리에 저장하고 이 주소값을 sum 변수에 저장합니다.
- sum(1, 2) 결과값을 출력합니다.
- multiply(3, 4)을 출력하려고 하지만 아직 빈 변수이므로 에러가 발생합니다.
- 에러가 발생했기 때문에 아래 코드는 실행되지않고 종료됩니다.

실제 코드를 구현할때는 되도록 선언문보다는 표현식이 더 낫습니다. 왜냐하면 선언문은 호이스팅 되고 표현식은 호이스팅 되지 않아, 표현식이 작성된 코드 아래줄에만 영향을 주기 때문에 선언문과 달리 윗줄에 영향을 주지않아 오류가 발생하지 않습니다.

## 스코프, 스코프 체인, outerEnvironmentReference

스코프는 식별자에 대한 유효범위로 함수를 기준으로 생성됩니다. 그리고 이러한 유효범위를 안에서 부터 바깥으로 차례로 검색해나가는 것을 스코프 체인이라고 합니다. 이 스코프체인을 가능하게 해주는 것이 LexicalEnvironment의 outerEnvironmentReference입니다.

```
var a = 1;

var outer = function () {
  var inner = function () {
    console.log(a); // undefined
    var a = 3;
  };

  inner();
  console.log(a); // 1
};

outer();
console.log(a); // 1
```

위 코드를 컨텍스트 관점으로 실행과정을 설명하면 다음과 같습니다.

- 전역 컨텍스트가 활성화 되고, environmentRecord에 식별자 a, 함수 outer가 담깁니다. 전역 컨텍스트에는 선언시점이 없어 outerEnvironmentReference는 저장하지 않습니다.
- a에 1을, outer에 함수를 할당합니다.
- outer함수 호출부를 만나면, 전역 컨텍스트에 대한 코드 실행은 일시 중단되고 outer에 대한 실행 컨텍스트가 생성되고 콜 스택 최상단에 담기며 활성화 됩니다.
- outer 컨텍스트의 environmentRecord에 식별자 inner가 담깁니다.
- outer 컨텍스트의 outerEnvironmentReference에는 outer함수가 선언될 시점의 LexicalEnvironment가 담기며 전역 공간에서 선언됐으므로 전역 컨텍스트의 LexicalEnvironment를 참조 복사합니다.
- inner에 함수를 할당 합니다.
- inner함수 호출부를 만나면 outer컨텍스트의 코드를 일시중단 하며 inner컨텍스트가 생성되고 활성화 됩니다.
- inner실행 컨텍스트의 environmentRecord에는 식별자a를 저장합니다.
- inner실행 컨텍스트의 outerEnvironmentReference는 inner함수가 선언될 시점의 LexicalEnvironment가 담기며, outer함수 내부에서 선언됐으므로 outer함수의 LexicalEnvironment가 참조복사됩니다.
- console.log(a); 출력 부분에서 inner컨텍스트의 environmentRecord에서 선언한 변수는 있지만 값은 없으므로 undefined를 출력합니다.
- a에 3을 할당하며 inner컨텍스트가 종료됩니다.
- 다시 outer컨텍스트에서 일시중단되었던 console.log(a) 부분부터 활성화 됩니다. outer컨텍스트의 environmentRecord에 a가 없으므로 outerEnvironmentReference를 찾으며 전역 LexicalEnvironment에 a에 할당된 값 1을 출력합니다.
- outer컨텍스트가 종료되며 다시 전역 컨텍스트에서 일시중단되었던 부분 console.log(a)을 실행합니다.
- 전역 컨텍스트 environmentRecord의 a에 접근하며 저장되어 있는 값 1을 출력하며 종료됩니다.

위와 같은 스코프 체인의 순서로 바깥에서 내부의 변수에는 접근하지 못하게 됩니다.

# this
