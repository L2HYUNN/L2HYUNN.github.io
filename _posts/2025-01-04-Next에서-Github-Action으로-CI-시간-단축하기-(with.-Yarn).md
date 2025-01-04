---
title: "Next에서 Github Action으로 CI 시간 단축하기 (with. Yarn)"
date: 2025-01-04 20:56 +0900
categories: [post, next]
tags: [next, ci]
image: /assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/next-build.webp
---

개발을 진행하다보면 빌드 과정에서 문제가 되는 부분을 발견하지 못한 상태로 계속 개발을 진행할 수 있습니다. 이러한 문제를 미리 발견하여 해결할 수 있다면 좋겠지만 보통 뒤 늦게 배포 시기에 해당 문제를 알아차리고는 합니다.

이러한 문제를 해결하기 위해 [CI(Continuous Integration)](https://www.atlassian.com/continuous-delivery/continuous-integration)를 구성할 수 있고 CI는 Github를 사용중이라면 [Github Action](https://docs.github.com/ko/actions/about-github-actions/understanding-github-actions)을 통해 간단히 구성해볼 수 있습니다.

최근 진행한 프로젝트에서 또한 CI를 구성하였고, 이 과정에서 CI 시간을 성공적으로 단축한 방법을 소개합니다.

> 본 글은 Next.js `14.2.5`, Node.js `20.18`, Yarn `4.4.0` 패키지 매니저를 활용하여 작성되었습니다.
{: .prompt-info}

## Workflow

아래는 처음 작성한 CI 워크플로우 입니다.

비교적 간단한 형태로 의존성 설치 이후 빌드 커맨드를 실행하고 있습니다.

```yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out current commit (${{ github.sha }})
        uses: actions/checkout@v4

      - name: Enable corepack
        run: corepack enable

      - name: Set up node v${{ env.NODE_VERSION }}
        uses: actions/setup-node@v4
        with:
          node-version: "20.16.0"

      - name: Install pacakges
        run: yarn install

      - name: Build
        run: yarn build
```

해당 워크플로우를 통해 CI를 실행해보면 아래와 같은 결과를 얻을 수 있습니다.

![image](/assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/long-build.webp)

간단하지만 CI 환경을 구축한 덕분에 이제 빌드 과정에서 문제가 발생한다면 바로 확인을 할 수 있게 되었습니다.

하지만 한 가지 아쉬운 점이 남아있습니다. 바로 CI를 완료하는데 총 2분 12초가 소요된다는 점입니다. 이 시간을 단축할 수 있는 방법은 없을까요? CI 시간을 단축하기 위해 빌드 과정에서 시간을 단축할 수 있는 방법을 찾아보았습니다.

## Cache

### 의존성 캐싱

첫 번째로 생각해낸 방법은 바로 **의존성을 설치하는 시간을 단축하는 것** 입니다. CI가 반복적으로 실행될 때 의존성이 변경되지 않는다면 기존에 설치한 의존성을 그대로 사용할 수 있지 않을까요? 즉, 의존성 설치 결과를 `Caching` 할 수 있지 않을까요?

[Cache action](https://github.com/actions/cache/tree/main?tab=readme-ov-file#creating-a-cache-key)을 사용한다면 이러한 아이디어를 쉽게 적용해볼 수 있습니다.

```yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out current commit (${{ github.sha }})
        uses: actions/checkout@v4

      - name: Enable corepack
        run: corepack enable

      - name: Set up node v${{ env.NODE_VERSION }}
        uses: actions/setup-node@v4
        with:
          node-version: "20.16.0"

      - name: Get yarn cache directory path
        id: yarn-cache-dir-path
        run: echo "dir=$(yarn config get cacheFolder)" >> $GITHUB_OUTPUT

      - uses: actions/cache@v4
        id: yarn-cache
        with:
          path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-

      - name: Install pacakges
        run: yarn install

      - name: Build
        run: yarn build
```

`Cache action`은 `path`에 속하는 파일들을 캐싱하고 이에 대응하는 고유한 캐시 키를 생성합니다. 이후 워크플로우가 다시 실행될 때 저장 된 캐시 키를 기반으로 캐시를 조회하여 일치하는 캐시가 존재하는 경우 해당 캐시를 다운로드하여 데이터를 재사용할 수 있습니다.

> `Cache action`에 대한 자세한 설명은 다음의 [링크](https://github.com/actions/cache/tree/main?tab=readme-ov-file#inputs)를 통해 확인하실 수 있습니다.
>
> 다른 환경에 대한 스크립트가 궁금하신 분들은 다음의 [예제](https://github.com/actions/cache/blob/main/examples.md#node---yarn-2) 파일을 확인해주세요.
{: .prompt-info}

이렇게 `Caching`이 적용된 워크플로우를 실행하면 아래와 같은 결과를 얻어볼 수 있습니다.

**Before Caching**
![image](</assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/install-packages-not-cache-(1).webp>)

**After Caching**
![image](/assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/install-packages-cache.webp)

위의 그림에서 알 수 있듯이 `Fetch step`의 소요시간이 0초에 가깝게 줄어든 것을 확인할 수 있습니다.

`Fetch step`은 `Resolution step`에서 생성된 dependency tree를 가지고 fetch로 요청해 각 dependency 설치를 합니다. 이 과정에서 만약 `.yarn/cache`에 dependency가 존재한다면 새로 설치 하지 않고 `cache`에 존재하는 파일을 사용합니다.

> **Yarn Install Architecture**에 대한 자세한 정보는 다음의 [링크](https://yarnpkg.com/advanced/architecture)를 참조하세요.
{: .prompt-tip}

따라서 우리는 이를 통해 `Cache action`이 정상적으로 동작함을 확인할 수 있습니다.

의존성을 설치하는 단계만 본다면 13초가 줄어들었다고 생각할 수 있지만 전체 과정을 살펴보면 저장된 캐쉬를 불러오는 과정이 존재하기 때문에 4초 정도를 제외해야합니다.

![image](/assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/run-action-cache.webp)

따라서 의존성 캐싱을 적용한 결과 평균적인 CI 실행 시간이 **8초** 정도 줄어들었음을 확인할 수 있었습니다.

CI 시간을 단축할 수 있었지만 이정도에서 만족하기는 아쉽겠죠. 따라서 다른 방법을 함께 찾아보았습니다.

### 빌드 캐시

두 번째로 생각해낸 방법은 바로 **빌드 시간을 단축하는 것** 입니다. 사실 전체 워크플로우가 실행되는 시간을 살펴보면 빌드 시간이 **약 1분 20초** 정도로 가장 긴 시간을 차지한다는 것을 확인할 수 있습니다. 위와 같은 캐싱 방법을 적용한다면 이 시간을 줄여볼 수 있지 않을까요?

![image](/assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/build-time.webp)

Next에서는 이러한 CI 시간 단축을 위한 `Caching` 방법을 제공하고 있습니다. 바로 `.next/cache` 를 사용하는 것 입니다. 이곳에는 빌드 과정에서 공유될 수 있는 캐시 파일을 저장할 수 있습니다.

> **Next Build Cache**에 대한 자세한 정보는 다음의 [링크](https://nextjs.org/docs/pages/building-your-application/deploying/ci-build-caching)에서 확인하실 수 있습니다.
{: .prompt-info}

빌드 캐시를 사용할 수 있게 워크플로우를 수정하면 다음과 같이 변경할 수 있습니다.

```yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out current commit (${{ github.sha }})
        uses: actions/checkout@v4

      - name: Enable corepack
        run: corepack enable

      - name: Set up node v${{ env.NODE_VERSION }}
        uses: actions/setup-node@v4
        with:
          node-version: "20.16.0"

      - name: Get yarn cache directory path
        id: yarn-cache-dir-path
        run: echo "dir=$(yarn config get cacheFolder)" >> $GITHUB_OUTPUT

      - uses: actions/cache@v4
        id: yarn-cache
        with:
          path: |
            ${{ steps.yarn-cache-dir-path.outputs.dir }}
            ${{ github.workspace }}/.next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/yarn.lock') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/yarn.lock') }}-

      - name: Install pacakges
        run: yarn install

      - name: Build
        run: yarn build
```

이제 우리는 의존성 설치 결과 뿐 아니라 빌드 결과 또한 캐싱할 수 있게 되었습니다.

변경된 워크플로우를 실행해보면 다음과 같은 결과를 얻을 수 있습니다.

![image](/assets/img/posts/2025-01-04-Next-Github-Action-CI---with-Yarn/build-time-reduce.webp)

결과적으로 기존에 1분 20초 정도 걸리던 빌드 시간을 49초까지 줄여볼 수 있었습니다. `Cache action`을 실행하는 시간이 그만큼 늘어났지만 이를 감안해도 전체 CI 실행 시간이 **약 30초** 정도 줄어들었음을 확인할 수 있었습니다.

## Conclusion

Next에서 Github Action을 통해 CI 시간을 단축하는 방법을 알아봤습니다. 30초가 크게 느껴지지 않을 수 있지만 의존성, 빌드 파일의 크기에 따라 이러한 캐싱은 더 큰 효용을 낼 수 있습니다. 개발 중 지속적인 통합 환경을 확보하는게 중요한 만큼 그 시간을 줄이는 방법에 대해 더욱 고민해보면 좋을 거 같습니다.
