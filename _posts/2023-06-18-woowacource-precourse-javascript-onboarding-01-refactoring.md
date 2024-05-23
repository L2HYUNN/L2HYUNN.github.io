---
title: 우아한테크코스 자바스크립트 온보딩 1번 리팩토링
date: 2023-06-18 23:15 +0900
categories: [challenge, woowacourse]
tags: [javascript, woowacourse]
image: /assets/img/posts/2024-05-23----3-/woowacourse.webp
---

> 온보딩 1번 문제 풀이는 [이전글](https://l2hyunn.github.io/posts/woowacourse-precourse-javascript-onboarding-01-start/)에서 확인하실 수 있습니다.
{: .prompt-info }

## 리팩토링
1번 문제 풀이 이후 기존 풀이에 몇 가지 아쉬운 부분이 있어 리팩토링을 해야겠다고 생각했다.

그러던 중 우연히 우테코에 참여하고 계신 지인분께 코드 리뷰를 받을 수 있었고 리뷰 내용과 개인적인 생각을 바탕으로 리팩토링을 진행하였다. 리팩토링을 위해 고려한 부분과 세부 과정은 다음과 같다.

### 주석이 꼭 필요할까?
처음 문제를 풀때는 매 함수마다 동작을 설명하는 주석을 `JSDoc`을 이용하여 상세히 추가 하였다.

당시 왜 그랬는지는 모르겠지만 주석을 상세히 다는 것이 더 좋은 코드를 만든다고 생각했던 것 같다. 하지만 필요 이상의 주석은 오히려 코드의 가독성을 낮출 수 있다. 그렇기에 추가 설명이 꼭 필요한 부분에만 주석을 다는 것이 옳다. 

문제를 풀면서 함수를 가능한 작게 만들려고 노력하였고 가독성을 고려하여 최대한 함수의 동작에 알맞은 이름을 붙이기 위해 노력하였다 그렇게 만들어진 함수들은 주석 없이도 그 동작을 충분히 설명해주고 있었다.

위와 같은 이유에서 기존 코드에 있던 주석들을 모두 제거하였다.

### 함수가 하나의 기능만 하는가?
처음 문제를 풀면서 가장 오래 고민했던 부분이다. 

당시에는 충분한 고민 끝에 가능한 작게 나눴다고 생각했지만 사실 몇몇 함수는 더 작게 나눌 수 있었다. 작게 나누는 과정에서 `이렇게까지 작게 만들어야 할까?` 라는 고민을 했던 것 같다.

하지만 함수는 가능한 작게, 하나의 기능만 하는 것이 좋다. 그렇기에 기존 함수들을 가능한 더욱 작게 나누어보기로 했다. 이전 코드를 살펴보며 함수를 더욱 작게 만들어보자.

```javascript
function addPageDigit(pageNumber) {
  return pageNumber
    .toString()
    .split("")
    .reduce((acc, cur) => acc + +cur, 0);
}
```

위의 함수는 이전에 구현한 `pageDigit`을 구하는 함수이다.

이 함수는 다음과 같이 총 3가지 기능을 하고 있다.

1. `pageNumber`를 `string`으로 변경한다.
2. `string`으로 변경된 `pageNumber`를 한 글자씩 잘라 배열로 만든다.
3. 배열로 만들어진 `pageNumber`의 각 자릿수를 모두 더한다.

함수의 이름을 생각해보았을 때 `addPageDigit`은 3번의 역할만을 하는 것이 옳다. 따라서 다음과 같이 1,2번과 3번을 각각 하나의 함수로 추출하였다.

```javascript
function splitPageNumber(pageNumber) {
  return pageNumber.toString().split("");
}

function addPageDigit(pageNumbers) {
  return pageNumbers.reduce((acc, cur) => acc + +cur, 0);
}
```

이제 각각의 함수가 이름에 맞는 하나의 기능만을 담당하고 있다.

또한 위와 같이 리팩토링한 덕분에 이전에는 `pageDigits`을 만들기 위해 `addPageDigit`, `getAddedPageDgitList` 두 곳을 살펴보아야 했다면 이제는 `getAddedPageDigitList`만을 통해 충분히 동작을 확인할 수 있게 되었다.

> 이와 같은 리팩토링에 따라 함수 이름을 `createAddedPageDigits`으로 변경하였다.

```javascript
function createAddedPageDigits(pageNumbers) {
  return pageNumbers.map((pageNumber) =>
    addPageDigit(splitPageNumber(pageNumber))
  );
}
```
### 함수는 어디까지 작게 만들어야 할까?
함수는 가능한 작게 만들어야 한다. 그렇다면 과연 함수는 어디까지 작게 만들어야 하는 것일까? 가능한 작게라고 하지만 분명 그 끝은 있을 것이다. 이전 코드의 `getMaxNumber` 함수를 살펴보자.

```javascript
function getMaxNumber(numbers) {
  return Math.max(...numbers);
}
```

`getMaxNumber`의 동작은 숫자 배열을 받아 `Math.max()`를 통해 최대값을 얻는 함수이다. 이것은 기존 `Math.max()`라는 함수에 새로운 이름을 붙여준 것과 동일하다. 이러한 경우에는 함수로 만들기 보다 있는 그대로 사용하는 것이 오히려 가독성을 높일 수 있는 방법일 것이다.

### 가독성을 더욱 높여보자
기존에 `getMaxNumber`를 사용하여 결과를 얻는 함수는 다음과 같다.

```javascript
function getScoreResult(pageNumbers) {
  return getMaxNumber([
    ...getAddedPageDigitList(pageNumbers),
    ...getMultipliedPageDigitList(pageNumbers),
  ]);
}
```


이후 `Math.max()`를 이용하여 다음과 같이 코드를 변경할 수 있다.

```javascript
function getScoreResult(pageNumbers) {
  return Math.max([
    ...getAddedPageDigitList(pageNumbers),
    ...getMultipliedPageDigitList(pageNumbers),
  ]);
}
```

코드를 읽는 것이 어렵지는 않지만 두 개의 스프래드 연산자가 가독성을 헤치고 있다. 이러한 문제를 해결하기 위해 다음과 같이 함수를 추가하고 일부 함수명을 변경하였다.


```javascript
function selectMaxAddedPage(pageNumbers) {
  return Math.max(...createAddedPageDigits(pageNumbers));
}

function selectMaxMultipliedPage(pageNumbers) {
  return Math.max(...createMultipliedPageDigits(pageNumbers));
}

function selectMaxScore(pageNumbers) {
  return Math.max(
    selectMaxAddedPage(pageNumbers),
    selectMaxMultipliedPage(pageNumbers)
  );
}
```

코드의 총 양은 늘어났지만 스프래드 연산자를 분리함으로써 가독성을 높일 수 있었다.

### 명령형이 아닌 선언형으로
명령형 보다는 적절한 추상화를 이용하여 선언형으로 코드를 짜는 것이 코드를 재사용하기 쉽고 가독성을 높일 수 있는 방법이다. 기존에 명령형으로 짜여져있던 본문을 가능한 추상화하여 선언형으로 만들어보자.

기존의 문제를 하나의 프로그램이라고 생각하고 프로그램에서 발생할 수 있는 예외를 처리할 수 있도록 `try/catch` 문을 이용한다면 `problem1`은 다음과 같이 생각할 수 있다.

```javascript
function problem1(pobi, crong) {
  try {
    return // 프로그램
  } catch(error) {
    return // 에러 
  }
}
```

문제의 프로그램은 `pobi`와 `crong`중 승자를 정하는 프로그램이기 때문에 다음과 같이 만들 수 있다.

```javascript
function selectResult(pobiScore, crongScore) {
  if (pobiScore > crongScore) return Result.POBI_WIN;
  if (crongScore > pobiScore) return Result.CRONG_WIN;

  return Result.DRAW;
}

function selectWinner(pobi, crong) {
  verifyInput(pobi);
  verifyInput(crong);

  const pobiScore = selectMaxScore(pobi);
  const crongScore = selectMaxScore(crong);

  return selectResult(pobiScore, crongScore);
}
```
> 결과를 결정하는 조건문 부분을 하나의 함수로 추출하였다.

이제 `problem1`을 다음과 같이 만들 수 있다.

```javascript
function problem1(pobi, crong) {
  try {
    return selectWinner(pobi, crong);
  } catch (error) {
    return Result.EXCEPTION;
  }
}
```

## 결론
하나의 프로그램을 만든다는 생각으로 코드를 만들고 리팩토링까지 해보았다. 이번 문제를 통해 `좋은 코드란 무엇인가?` 라는 질문을 스스로에게 던지고 많은 고민을 해볼 수 있었던 것 같다.

## 전체 코드

```javascript
function splitPageNumber(pageNumber) {
  return pageNumber.toString().split("");
}

function addPageDigit(pageNumbers) {
  return pageNumbers.reduce((acc, cur) => acc + +cur, 0);
}

function multiplyPageDigit(pageNumbers) {
  return pageNumbers.reduce((acc, cur) => acc * +cur);
}

function createAddedPageDigits(pageNumbers) {
  return pageNumbers.map((pageNumber) =>
    addPageDigit(splitPageNumber(pageNumber))
  );
}

function createMultipliedPageDigits(pageNumbers) {
  return pageNumbers.map((pageNumber) =>
    multiplyPageDigit(splitPageNumber(pageNumber))
  );
}

function selectMaxAddedPage(pageNumbers) {
  return Math.max(...createAddedPageDigits(pageNumbers));
}

function selectMaxMultipliedPage(pageNumbers) {
  return Math.max(...createMultipliedPageDigits(pageNumbers));
}

function selectMaxScore(pageNumbers) {
  return Math.max(
    selectMaxAddedPage(pageNumbers),
    selectMaxMultipliedPage(pageNumbers)
  );
}

const ErrorMessage = Object.freeze({
  ERROR_INVALID_PAGE_LENGTH: "페이지 배열의 길이는 2여야만 합니다.",
  ERROR_INVALID_LEFT_PAGE_NUMBER: "왼쪽 페이지는 홀수여야만 합니다.",
  ERROR_INVALID_RIGHT_PAGE_NUMBER: "오른쪽 페이지는 짝수여야만 합니다.",
  ERROR_INVALID_PAGE_ORDER: "페이지의 순서가 올바르지 않습니다.",
});

function verifyInput(input) {
  if (input.length !== 2)
    throw new Error(ErrorMessage.ERROR_INVALID_PAGE_LENGTH);
  if (input[0] % 2 !== 1) throw new Error(ErrorMessage.ERROR_INVALID_LEFT_PAGE);
  if (input[1] % 2 !== 0)
    throw new Error(ErrorMessage.ERROR_INVALID_RIGHT_PAGE);
  if (input[0] > input[1])
    throw new Error(ErrorMessage.ERROR_INVALID_PAGE_ORDER);
  if (input[0] + 1 !== input[1])
    throw new Error(ErrorMessage.ERROR_INVALID_PAGE_ORDER);
}

const Result = Object.freeze({
  EXCEPTION: -1,
  DRAW: 0,
  POBI_WIN: 1,
  CRONG_WIN: 2,
});

function selectResult(pobiScore, crongScore) {
  if (pobiScore > crongScore) return Result.POBI_WIN;
  if (crongScore > pobiScore) return Result.CRONG_WIN;

  return Result.DRAW;
}

function selectWinner(pobi, crong) {
  verifyInput(pobi);
  verifyInput(crong);

  const pobiScore = selectMaxScore(pobi);
  const crongScore = selectMaxScore(crong);

  return selectResult(pobiScore, crongScore);
}

function problem1(pobi, crong) {
  try {
    return selectWinner(pobi, crong);
  } catch (error) {
    return Result.EXCEPTION;
  }
}

console.log(problem1([97, 98], [197, 198]));
console.log(problem1([131, 132], [211, 212]));
console.log(problem1([99, 102], [211, 212]));

module.exports = problem1;

```
