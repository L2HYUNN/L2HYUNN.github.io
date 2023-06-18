---
title: 우아한테크코스 자바스크립트 온보딩 1번 풀어보기
date: 2023-06-12 21:58 +0900
categories: [challenge, woowacourse]
tags: [javascript, woowacourse]
---
## 문제

### 🚀 기능 요구 사항

포비와 크롱이 페이지 번호가 1부터 시작되는 400 페이지의 책을 주웠다. 책을 살펴보니 왼쪽 페이지는 홀수, 오른쪽 페이지는 짝수 번호이고 모든 페이지에는 번호가 적혀있었다. 책이 마음에 든 포비와 크롱은 페이지 번호 게임을 통해 게임에서 이긴 사람이 책을 갖기로 한다. 페이지 번호 게임의 규칙은 아래와 같다.

1. 책을 임의로 펼친다.
2. 왼쪽 페이지 번호의 각 자리 숫자를 모두 더하거나, 모두 곱해 가장 큰 수를 구한다.
3. 오른쪽 페이지 번호의 각 자리 숫자를 모두 더하거나, 모두 곱해 가장 큰 수를 구한다.
4. 2~3 과정에서 가장 큰 수를 본인의 점수로 한다.
5. 점수를 비교해 가장 높은 사람이 게임의 승자가 된다.
6. 시작 면이나 마지막 면이 나오도록 책을 펼치지 않는다.

포비와 크롱이 펼친 페이지가 들어있는 배열 pobi와 crong이 주어질 때, 포비가 이긴다면 1, 크롱이 이긴다면 2, 무승부는 0, 예외사항은 -1로 return 하도록 solution 메서드를 완성하라.

### 제한사항

- pobi와 crong의 길이는 2이다.
- pobi와 crong에는 [왼쪽 페이지 번호, 오른쪽 페이지 번호]가 순서대로 들어있다.

### 실행 결과 예시

| pobi       | crong      | result |
| ---------- | ---------- | ------ |
| [97, 98]   | [197, 198] | 0      |
| [131, 132] | [211, 212] | 1      |
| [99, 102]  | [211, 212] | -1     |

## 접근
여러 알고리즘 문제와 비교하면 문제 자체는 크게 어렵지 않았다.

하지만 단순한 문제 풀이가 아닌 `실제 프로그램`을 만든다고 생각하니 고민할 부분이 많았다. 

풀이에 앞서 문제를 파악한 후 구현해야할 `기능 목록`을 다음과 같이 작성해보았다.

### 📝 기능 목록

#### 구현
- [x] 각 페이지 번호의 각 자리 숫자를 모두 더한다.
- [x] 각 페이지 번호의 각 자리 숫자를 모두 곱한다.
- [x] 위의 과정에서 나온 수 중 가장 큰 수를 정한다.
- [x] 점수를 비교하여 높은 점수를 가진 사람을 승자로 정한다.

#### 예외
- [x] 입력된 배열의 길이가 2가 아닐 경우 -1을 반환한다.
- [x] 입력된 배열이 연속된 값이 아닐 경우 -1을 반환한다.

## 고민

1번 문제를 풀면서 가장 고민한 부분은 `어떻게 하면 더 좋은 프로그램을 만들 수 있을까?` 이다.

평소 문제 풀이에만 집중했던 알고리즘과는 달리 `실제 프로그램`을 만들기 위해서 좋은 프로그램이란 무엇인지에 대해 고민해야만 했다.

좋은 프로그램에 대한 많은 정의가 존재하지만 이번 문제에서는 특히 `재사용성`, `가독성`, `확장성`, `일관성`을 가지는 프로그램을 만들기 위해 노력하였다.

## 풀이

1. 함수는 가능한 작게 하나의 기능만을 담당하도록 만들었다.

2. 명확하고 일관된 변수, 함수 이름을 사용하고자 하였다.

3. 상수는 별도의 변수로 따로 정의하여 사용하였다.

4. `JSDoc`을 이용하여 함수에 대한 주석을 추가하였다.

5. `enum`을 이용하여 직관적으로 결과를 확인할 수 있게 하였다.

6. `Object.freeze`를 이용하여 불변 객체를 사용하고자 하였다.

### 전체 코드
```javascript
/**
 * pageNumber를 인자로 받아 각 자릿수의 합을 반환한다.
 *
 * @param {number} pageNumber
 * @returns {number}
 */
function addPageDigit(pageNumber) {
  return pageNumber
    .toString()
    .split("")
    .reduce((acc, cur) => acc + +cur, 0);
}

/**
 * pageNumber를 인자로 받아 각 자릿수의 곱을 반환한다.
 *
 * @param {number} pageNumber
 * @returns {number}
 */
function multiplyPageDigit(pageNumber) {
  return pageNumber
    .toString()
    .split("")
    .reduce((acc, cur) => acc * +cur);
}

/**
 * numbers를 인자로 받아 가장 큰 값을 반환한다.
 *
 * @param {number[]} numbers
 * @returns {number}
 */
function getMaxNumber(numbers) {
  return Math.max(...numbers);
}

/**
 * pageNumbers를 인자로 받아 pageNumber의 각 자릿수의 합을 가진 배열을 반환한다.
 *
 * @param {number[]} pageNumbers
 * @returns {number[]}
 */
function getAddedPageDigitList(pageNumbers) {
  return pageNumbers.map((pageNumber) => addPageDigit(pageNumber));
}

/**
 * pageNumbers를 인자로 받아 pageNubmer의 각 자릿수의 합을 가진 배열을 반환한다.
 *
 * @param {number[]} pageNumbers
 * @returns {number[]}
 */
function getMultipliedPageDigitList(pageNumbers) {
  return pageNumbers.map((pageNumber) => multiplyPageDigit(pageNumber));
}

/**
 * pageNumbers를 인자로 받아 게임의 규칙에 따라 가장 큰 수를 반환한다.
 *
 * @param {number[]} pageNumbers
 * @returns {number}
 */
function getScoreResult(pageNumbers) {
  return getMaxNumber([
    ...getAddedPageDigitList(pageNumbers),
    ...getMultipliedPageDigitList(pageNumbers),
  ]);
}

const ErrorMessage = Object.freeze({
  ERROR_INVALID_PAGE_LENGTH: "페이지의 길이가 올바르지 않습니다.",
  ERROR_INVALID_PAGE_ORDER: "페이지의 순서가 올바르지 않습니다.",
});

/**
 * pobi 혹은 crong의 길이가 2가 아닐때 에러를 발생시킵니다.
 * pobi 혹은 crong의 페이지가 순서대로 나열되어 있지 않을때 에러를 발생시킵니다.
 *
 * @param {number[]} pobi
 * @param {number[]} crong
 *
 * @throws {ExceptionError}
 */
function throwExceptionError(pobi, crong) {
  if (pobi.length !== 2 || crong.length !== 2)
    throw new Error(ErrorMessage.ERROR_INVALID_PAGE_LENGTH);
  if (pobi[0] + 1 !== pobi[1] || crong[0] + 1 !== crong[1])
    throw new Error(ErrorMessage.ERROR_INVALID_PAGE_ORDER);
}

const Result = Object.freeze({
  EXCEPTION: -1,
  DRAW: 0,
  POBI_WIN: 1,
  CRONG_WIN: 2,
});

function problem1(pobi, crong) {
  try {
    throwExceptionError(pobi, crong);
  } catch (error) {
    console.error(error);
    return Result.EXCEPTION;
  }

  const pobiNumberResult = getScoreResult(pobi);

  const crongNumberResult = getScoreResult(crong);

  if (pobiNumberResult > crongNumberResult) return Result.POBI_WIN;
  if (crongNumberResult > pobiNumberResult) return Result.CRONG_WIN;
  if (pobiNumberResult === crongNumberResult) return Result.DRAW;

  return Result.EXCEPTION;
}

console.log(problem1([97, 98], [197, 198]));
console.log(problem1([131, 132], [211, 212]));
console.log(problem1([99, 102], [211, 212]));

module.exports = problem1;
```