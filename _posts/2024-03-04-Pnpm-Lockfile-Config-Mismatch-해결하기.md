---
title: Pnpm Lockfile Config Mismatch 해결하기
date: 2024-03-04 15:06 +0900
categories: [post, error]
tags: [pnpm, lockfile, config, mismatch, error, ci]
image: /assets/img/posts/2024-03-04-Pnpm-Lockfile-Config-Mismatch-/pnpm-locfile-config-mismatch.webp
---

새로운 프로젝트를 시작하면서 CI를 설정하던 중 아래와 같은 에러가 발생하였다.

## Error

> **ERR_PNPM_LOCKFILE_CONFIG_MISMATCH**
>
> Cannot proceed with the frozen installation. The current `"settings.autoInstallPeers"` configuration doesn't match the value found in the lockfile
>
> Update your lockfile using "pnpm install --no-frozen-lockfile"
{: .prompt-danger}

에러 로그에 따르면 현재 `"settings.autoInstallPeers"` 설정이 lockfile에 있는 정보와 다르기 때문에 `frozen install`을 실행 할 수 없어 발생한 문제처럼 보인다. 로그에서는 문제 해결을 위해 `pnpm install --no-frozen-lockfile` 을 이용하여 lockfile을 업데이트할 것을 권하고 있다.

## Approach
에러 로그에서 제안하고 있는 `pnpm install --no-frozen-lockfile`를 사용하여 `lockfile`을 업데이트 하기에 앞서, 로그에서 이야기 하고 있는 `frozen installation`이 무엇인지 찾아보았다.

> **Frozen Installation**
>
> `--frozen-lockfile` 옵션을 활성화 하면 pnpm이 lockfile을 생성하지 않으며 lockfile의 내용이 명세서와 맞지 않거나 업데이트가 필요하거나 혹은 lockfile이 존재하지 않는 경우 설치에 실패한다. pnpm은 CI 환경에서 기본으로 이 옵션을 활성화한다.
{: .prompt-info}

즉, `frozen installation` 이란 lockfile을 생성하지 않고 의존성을 설치하는 것을 의미한다.

### Non Frozen Installation
위의 정보에 대한 이해를 바탕으로 다시 에러를 살펴보면 다음과 같이 생각할 수 있다.

> CI 환경에서 pnpm은 `frozen installation`을 진행하였고 `"settings.autoInstallPeers"` 설정이 lockfile에 있는 설정과 다르기 때문에 에러가 발생하였다. 따라서 `Non frozen installation`을 진행하여 새로운 lockfile을 생성할 것을 권하고 있는 것이다.
{: .prompt-info}

CI에서 `pnpm install` 스크립트를 `pnpm install --no-frozen-lockfile`로 업데이트하여 의존성 설치를 진행하면 에러가 해결될 것 처럼 보인다. 하지만 이 방법에는 한 가지 의문이 남아있다.


> "이전까지 동일한 스크립트를 사용하면서 문제가 발생하지 않았었는데 왜 갑자기 에러가 생기게 된 걸까?"
{: .prompt-warning}

이전 까지 문제 없이 해당 CI를 잘 사용하고 있었기 때문에 갑자기 스크립트에 문제가 있다고 보기는 어려웠다. 따라서 기존 CI 스크립트를 수정하는 것이 아닌 다른 방법을 찾아보게 되었다.

### Pnpm Issues
방법을 찾아보던 중 [pnpm](https://github.com/pnpm/pnpm/issues) 에 올라온 이슈를 통해 보다 명확한 이유와 해결방법을 알 수 있었다.

> [Getting a lockfile config mismatch after upgrading to version 8.6.1 #6649](https://github.com/pnpm/pnpm/issues/6649)
{: .prompt-info}

해당 이슈에 따르면 pnpm의 버전을 `8.6.1`로 업그레이드 한 후에 나와 동일한 문제가 발생하였다고 한다.

pnpm의 메인테이너 중 한 명인 `zkochan`의 코멘트에 따르면 이 상황이 pnpm v7과 v8을 동시에 사용하려고 했기 때문에 발생한 문제라고 한다. pnpm v7은 `auto-install-peers=false`를 기본값으로,  v8은 `true`를 기본값으로 가진 lockfile을 생성한다.

![image](/assets/img/posts/2024-03-04-Pnpm-Lockfile-Config-Mismatch-/pnpm-v8-update.webp){: width="600" }

> [pnpm v8](https://github.com/orgs/pnpm/discussions/4967)에서 `auto-install-peers`의 기본 값이 `true`로 변경되었다.
{: .prompt-info}

### In My Case
프로젝트를 시작할 때 pnpm의 업데이트 소식을 접하게 되면서 곧 바로 pnpm 버전 업데이트를 진행하였다. (아마 이때 v8을 가진 lockfile이 생성 되었을 것이다.) 이후 CI를 실행 하였을 때 위와 같은 문제가 발생하였다.

그동안 찾아본 정보를 종합해보았을 때 현재 프로젝트에 v7과 v8 두 가지 버전의 pnpm을 사용중일 것이라고 예측할 수 있다. 현재 `CLI`에서는 v8 버전을 사용하고 있었기 때문에 곧 바로 `CI`에서 사용하고 있는 pnpm의 버전을 찾아보았다.

![image](/assets/img/posts/2024-03-04-Pnpm-Lockfile-Config-Mismatch-/install-pnpm-action-v7.webp)

예상했던 대로 CI에서는 v7 버전의 pnpm을 설치하고 있었다.

> 현재 프로젝트에 있는 lockfile은 v8을 통해 생성 되었기 때문에 `autoInstallPeers`가 `true`로 설정되어 있지만 CI에서는 v7을 사용하고 있기 때문에 `autoInstallPeers`가 `false`이어야 한다.
{: .prompt-info}

따라서 이 문제를 근본적으로 해결하기 위해서는 다음의 두 가지 방법을 생각해볼 수 있다.

1. `CI`에서 설치하고 있는 `pnpm`의 버전 v8로 변경하기
2. `.npmrc`를 이용해서 `auto-install-peers=false` 옵션 설정하기

## Solving
도움을 받은 [Issue](https://github.com/pnpm/pnpm/issues/6649)에서 많은 사람들이 2번 방식을 사용하고 있어 아래와 같이 `.npmrc`를 추가하여 문제를 해결하였다. 추후에 새로운 프로젝트에서는 `CI`에 있는 pnpm의 버전을 v8로 업데이트 하여 사용할 것 같다.
### How
```diff
// .npmrc
+ auto-install.peers=false
```

```diff
// .pnpm-lock.yaml
- autoInstallPeers: true
+ autoInstallPeers: false
```

### Result

![image](/assets/img/posts/2024-03-04-Pnpm-Lockfile-Config-Mismatch-/pnpm-ci-problem-solving.webp){: width="600" }

> [ci: fix ci error pnpm/pnpm#6649](https://github.com/L2HYUNN/frontend-info/pull/1/commits/04b23ba3c59a7fb6727938cca33d8dbb129621d7)
{: .prompt-tip}
