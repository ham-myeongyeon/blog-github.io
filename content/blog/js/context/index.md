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

위 코드를 예시로 실행 한다면 먼저 위에서 부터 순차적으로 실행 됩니다.

- 먼저 코드를 실행하는 순간 전역 컨텍스트가 실행 컨텍스트에 담깁니다. 변수 a와 outer함수에 대한 정보가 담깁니다.
- 그뒤 outer()실행부를 만나면 outer에 대한 컨텍스트가 생깁니다.

# VariableEnvironment

# LexicalEnvironment

## environmentRecord와 호이스팅

## 스코프 체인, outerEnvironmentReference

# this
