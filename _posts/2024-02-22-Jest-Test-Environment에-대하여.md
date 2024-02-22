---
title: Jest Test Environment에 대하여
date: 2024-02-22 23:05 +0900
categories: [post, jest]
tags: [test, jest, node, jsdom]
image: /assets/img/posts/2024-02-22-Jest-Test-Environment/jest-logo-1.webp
---

`Next` 프로젝트에서 `Jest`를 설정하던 중 `testEnvironment`에 대한 의문이 생겨 간단히 정리한 글입니다.

## Jest
[Jest](https://jestjs.io/)는 [Meta](https://about.meta.com/ko/)에서 개발한 JavaScript Testing Framework이다.

`Jest`는 Test Runner, Test Matcher, Test Mock 등 테스트에 필요한 여러 라이브러리를 제공해주기 때문에 별도의 테스트 라이브러리 설치 없이 테스트를 진행할 수 있다.

## Test Environment
[Setting up Jest with Next.js](https://nextjs.org/docs/app/building-your-application/testing/jest#creating-your-first-test)에 따르면 Jest를 설정할 때 testEnvironment를 `jsdom`으로 설정하는 것을 볼 수 있다.

> `JSdom`은 Node 환경에서 DOM과 HTML 표준과 같은 수 많은 웹 표준을 순수 JavaScript로 구현한 것이다. `JSdom`을 이용하면 Node 환경에서 실제 브라우저에서 실행하는 것처럼 JavaScript 코드를 동작시킬 수 있다.
{: .prompt-info}

문득 Jest의 기본 테스트 환경이 궁금해 [공식 문서](https://nextjs.org/docs/app/building-your-application/testing/jest#creating-your-first-test)를 살펴보니 `node`를 기본 테스트 환경으로 사용하는 것을 알 수 있었다. 이러한 테스트 환경을 살펴 보던 중 아래와 같은 궁금증이 생기게 되었다.

> Jest는 React, Angular, Vue등 다양한 웹 프레임워크와 함께 주로 사용되는데 왜 기본 테스트 환경을 `jsdom`이 아닌 `node`로 설정한 걸까?

이에 대한 해답은 21년 작성된 [Jest 27: New Defaults for Jest, 2021 edition](https://jestjs.io/blog/2021/05/25/jest-27#flipping-defaults)에서 찾을 수 있다.

해당 글에 따르면 v26 까지의 Jest는 `jsdom`을 기본 테스트 환경으로 사용하고 있었다는 것을 알 수 있다. 하지만 이러한 기본 동작이 DOM 과 같은 웹 브라우저 환경을 사용하지 않는 Node Application에 있어서는 개발자도 모르는 퍼포먼스 오버헤드를 발생시킬 수 있기 때문에 기본 테스트 환경을 `node`로 변경하게 되었다고 한다.

Jest팀에서는 이러한 퍼포먼스적 이유에서 DOM 환경을 필요로 하는 테스트와 그렇지 않은 테스트가 섞인 프로젝트의 경우 기본 테스트 환경을 `node`로 설정하고 DOM이 사용되는 곳에 [docblocks](https://jestjs.io/docs/configuration#testenvironment-string)을 이용할 것을 권장하고 있다.

> 위와 같은 이유를 바탕으로 더 작은 패키지 사이즈를 위해 [Jest v28](https://jestjs.io/blog/2022/04/25/jest-28#breaking-changes)에서  `jsdom`을 구성하기 위한 패키지인 `jest-environment-jsdom`이 Jest dependency tree에서 제거되었다.
{: .prompt-info}

## Conclusion
테스트 환경에 대해서 찾아보면서 `jsdom`을 이용하는 것이 `node`에 비해 상당한 퍼포먼스 오버헤드를 발생시킨다는 것을 알게되었다. 이후 `jsdom`이 얼마나 큰 오버헤드를 발생시키는지 벤치마크를 만들어보면 두 환경의 차이를 더욱 분명히 알 수 있을 것 같다. 또한 `node`를 기본 테스트 환경으로 구성하고 docblocks을 부분적으로 이용하는 방식이 얼마나 큰 성능적 이점을 얻을 수 있는지 테스트해본다면 좋을 것이다.
