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

## environmentRecord와 호이스팅

## 스코프 체인, outerEnvironmentReference

# this
