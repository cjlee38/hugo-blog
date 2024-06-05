---
categories: git
date: "2024-06-05T00:00:00Z"
tags: ['git', 'absorb']
title: 'Git absorb 소개'
draft: false
---

어디서 처음 봤는지는 정확히 기억 안나지만, 우연히 괜찮은 git 관련 오픈소스를 발견해서 소개해본다.  이름은 `git absorb` 라고, `git` 기반의 일종의 extension 인데, commit history를 관리할 때 유용하게 사용할 수 있다. 먼저 어떤 extension 인지 서두에 정리해보자면, 내가 코드를 수정한 뒤, 해당 작업을 과거의 commit 어딘가에 같이 포함시키고 싶을 때, “가장 관련성이 높은 commit을 찾아”서 흡수시킬 수 있다.

### fixup commit

먼저 `git absorb` 를 알기 전에 먼저 알아둬야 하는 것이 있는데, 바로 `fixup commit`이다. manpage(`man git-commit 1`) 에도 설명이 잘 나와있는데, commit 을 수행할 때 단순히 새로운 commit 을 추가하는게 아니라, 이전의 commit 과 “합칠 수 있는” commit을 제공한다.

가령, 아래와 같이 `a.txt` 파일을 만들고 commit 한번 `b.txt` 파일을 만들고 commit 한 번으로, 총 두 번의 commit 을 통해 아래와 같이 commit history가 있다고 가정해보자.

```bash
ee8f6a4 second # b.txt
8eb5b88 first # a.txt
```

이 상황에서, `a.txt` 파일에 누락된 사항을 뒤늦게 발견했다고 가정해보자. 새로운 commit 을 만들기는 싫고, 과거의 commit 에 그대로 포함시키고 싶다면 어떻게 해야할까 ? 가장 익숙한(?) 방법은 아마  `git reset` 을 통해 commit history 를 제거하고, `git commit --amend` 와 같은 명령어를 이용해서 누락된 수정사항을 포함시킨 뒤, 다시 commit 을 수행하는 방법이다.

하지만, 목표 commit 에 도달하기까지 수많은 commit history 가 이미 존재하는 상황이라면, `--fixup` commit을 통해 손쉽게 해결할 수 있다. 사용 방법은 간단하다. 아래와 같이 파라미터를 넘겨주면 된다.

```bash
git commit --fixup=8eb5b88 # `first` commit
```

그리고 `git log` 를 통해 살펴보면 다음과 같이 ‘fixup commit’ 이 추가된다

```bash
# git log --oneline
e23e811 (HEAD -> main) fixup! first
ee8f6a4 second
8eb5b88 first
```

이후, `git rebase -i --autosquash <commit-id>` 를 통해 interactive rebase 를 진행하면, 다음과 같이 fixup commit 이 수정하고자 하는 commit 바로 아래에 자동으로 위치하게 된다. (commit-id 는 first commit 보다 이전 commit id 로 지정한다.)

```bash
# vim editor
pick 8eb5b88 first
fixup e23e811 fixup! first
pick ee8f6a4 second
```

그대로 저장하고 나와서 rebase 를 수행하면 (별도의 conflict 가 없다면) fixup commit 이 first commit 으로 포함되어 들어가게 된다. (이 외에도 `squash commit`, `amend commit` 도 존재한다)

### git absorb

git absorb 는 수정사항이 “어느 commit 으로 포함되어야 하는지” 를 알아서 찾아서, `fixup commit` 단계를 스킵하도록 해준다. 

위 예시에서, `fixup commit` 을 수행하는 대신, `git absorb` 를 입력하면, `fixup commit` 을 대신 수행해준다. 이 때 어떤 commit 으로 fixup 할 것인지에 대한 판단은 file change 를 기준으로 선정한다. (사실 좀 더 정확한 알고리즘은 [이곳](https://github.com/tummychow/git-absorb?tab=readme-ov-file#how-it-works-roughly) 에 나와있다)

> The command essentially looks at the lines that were modified, finds a changeset modifying those lines, and amends that changeset to include your uncommitted changes.
> 

그렇기에, 앞서 이야기했듯 누락된, 즉 수정된 부분은 `a.txt` 이고, 해당 commit 은 ‘first commit’ 이므로 아래와 같이 해당 commit 으로의 `fixup commit` 을 수행한다. 반대로 만약 수정한 파일이 `b.txt` 였다면 'second commit' 으로 fixup commit 이 수행되었을 것이다.

```bash
# git log --oneline
e23e811 (HEAD -> main) fixup! first
ee8f6a4 second
8eb5b88 first
```

또한, 이 `fixup commit` 은 hunk 단위로 수행되기 때문에, 여러 파일을 수정했다면 각 hunk에 걸맞는 `fixup commit` 을 만들어낸다.

### 주의사항

단, 이렇게 git absorb 를 사용할 때에는 몇 가지를 고려해야 한다.(사실은 `git absorb` 보단, `fixup commit` 자체에 대한 고려사항이다.)

아마 대부분의 상황에서, 팀 단위의 협업으로 코드를 수정할 때는 Pull Request(Merge Request) 로 코드리뷰를 진행하고, 이를 merge 하는 방식으로 진행할텐데, 이 때 squash merge 를 하는 경우에는 사실 상 이 기능이 필요없을 것이다. 

반대로, merge commit 을 하는 경우에는 유용하게 사용될 수 있다. 피쳐 브랜치에서 수행한 commit 이 main stream 브랜치 commit history 에 그대로 반영되므로, 가급적 깔끔하게 유지해야 할 것이고 이 때 fixup commit 이 도움을 줄 수 있다. 하지만 여전히 코드리뷰 단계에서 `fixup commit` 을 수행하면 force push 를 해야하는 상황이 발생한다. 

심지어 PR 을 사용하지 않는다면 훨씬 더 심각한 문제([예를 들면 이런 상황](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0#_rebase_peril))를 초래할 수 있다는 점에 유의하자.
