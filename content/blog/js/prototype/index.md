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

## constructor 프로퍼티

# 프로토 타입 체인

## 메서드 오버라이드

## 프로토타입 체인

## 객체 전용 메서드의 예외사항

## 다중 프로토타입 체인
