---
title: 우아한테크코스 자바스크립트 온보딩 2번 풀어보기
date: 2023-06-28 18:47 +0900
categories: [challenge, woowacourse]
tags: [javascript, woowacourse]
image: /assets/img/posts/2024-05-23----3-/woowacourse.webp
---
## 문제

### 🚀 기능 요구 사항

암호문을 좋아하는 괴짜 개발자 브라운이 이번에는 중복 문자를 이용한 새로운 암호를 만들었다. 예를 들어 "browoanoommnaon"이라는 암호문은 다음과 같은 순서로 해독할 수 있다.

1. "browoanoommnaon"
2. "browoannaon"
3. "browoaaon"
4. "browoon"
5. "brown"

임의의 문자열 cryptogram이 매개변수로 주어질 때, 연속하는 중복 문자들을 삭제한 결과를 return 하도록 solution 메서드를 완성하라.

### 제한사항

- cryptogram은 길이가 1 이상 1000 이하인 문자열이다.
- cryptogram은 알파벳 소문자로만 이루어져 있다.

### 실행 결과 예시

| cryptogram        | result  |
| ----------------- | ------- |
| "browoanoommnaon" | "brown" |
| "zyelleyz"        | ""      |


## 접근

전체적으로 1번 문제에 비해 구현할 양이 적었다.

문제 풀이를 위해 먼저 전체 코드를 작성한 후에 작은 부분으로 나누는 리팩토링을 진행하였다.

풀이에 앞서 작성한 `구현 목록`은 다음과 같다.

### 🧑🏻‍💻 구현 목록

#### 구현 
- [x] 주어진 문자열의 중복 문자를 제거하는 함수를 구현한다.
- [x] 반복문을 이용하여 주어진 문자열의 중복 문자를 하나씩 제거한다.
- [x] 연속하는 중복 문자들을 삭제한 결과를 반환한다.

#### 예외
- [x] 입력된 문자열의 길이가 1이상 1000이하가 아닌 경우 예외를 발생시킨다.
- [x] 입력된 문자열이 알파벳 소문자 이외의 문자를 포함할 경우 예외를 발생시킨다.


## 고민

1번 문제를 통해 배웠던 부분들을 바탕으로 코드를 작성하여 크게 어려움 없이 문제를 해결할 수 있었다.

이번 문제를 풀면서 가장 많이 신경을 쓴 부분은 `함수 이름`과 `가독성`이다.


## 풀이
중복된 문자열을 제거한 이후 만들어진 새로운 문자열에서 중복된 문자를 다시 제거해야만 한다. 이를 위해 재귀를 이용하여 해당 함수를 구현하였다.

가독성을 위해 `재귀`, `중복 문자 제거`, `중복 조건 확인` 3가지 함수로 나눠 작성하였다. 

### 전체 코드
```javascript
function isDuplicate(string, index) {
  return (
    string[index - 1] === string[index] || string[index] === string[index + 1]
  );
}

function deleteDuplicateCharacter(string) {
  return Array.from(string)
    .filter((_, index) => !isDuplicate(string, index))
    .join("");
}

function deleteAllDuplicateCharacter(string) {
  const result = deleteDuplicateCharacter(string);

  return string.length !== result.length
    ? deleteAllDuplicateCharacter(result)
    : result;
}

const ErrorMessage = Object.freeze({
  ERROR_INVALIDE_CRYPTOGRAM: "입력된 암호문의 형태가 유효하지 않습니다.",
  ERROR_INVALIDE_CRYPTOGRAM_LENGTH: "입력된 암호문의 길이가 올바르지 않습니다.",
});

function validateCryptogram(cryptogram) {
  if (cryptogram.match(/[a-z]/g).join("") !== cryptogram)
    throw new Error(ErrorMessage.ERROR_INVALIDE_CRYPTOGRAM);
  if (cryptogram.length < 0 || cryptogram.length > 1000)
    throw new Error(ErrorMessage.ERROR_INVALIDE_CRYPTOGRAM_LENGTH);
}

function decrypt(cryptogram) {
  validateCryptogram(cryptogram);

  return deleteAllDuplicateCharacter(cryptogram);
}

function problem2(cryptogram) {
  try {
    return decrypt(cryptogram);
  } catch (error) {
    return error.message;
  }
}

console.log(problem2("browoanoommnaon"));

module.exports = problem2;
```
