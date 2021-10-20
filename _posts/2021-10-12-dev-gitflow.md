---
layout: post
title:  "Gitflow"
subtitle:   Gitflow
categories: datascience
tags: dev git gitflow 
comments: true
---

# 목차
1. [Git Flow란](#git-flow란)
2. [Git Flow 흐름](#git-flow-흐름)
3. [Git Flow 사용 예시](#Git-Flow-사용-예시)
4. [Reference](#reference)

<br>

---

<br><br>


# Git Flow란
Git Flow는 **Vincent Driessen**에 의해 고안된 방법론이다. <br> Git으로 개발할 때 거의 표준과 같이 사용되지만, Vincent Driessen이 언급했듯, 완벽한 방법론은 아니고 각자 개발 환경에 따라 수정하고 변형해서 사용한다.


Git Flow는 5가지의 브랜치를 사용한다. <BR>
항상 유지되는 메인 브랜치(master, develop)과 일정 기간 동안만 유지되는 보조 브랜치(feature, release, hotfix)가 있다.
> - master: 제품으로 출시될 수 있는 브랜치
> - develop: 다음 출시 버전을 개발하는 브랜치
> - feature: 기능을 개발하는 브랜치
> - release: 이번 출시 버전을 준비하는 브랜치
> - hotfixes: 출시 버전에서 발생한 버그를 수정하는 브랜치

<br><br>

## Git Flow 흐름
Git Flow의 프로세스는 아래 이미지와 같다.
![img](https://drive.google.com/uc?id=1UbfqgDi3dBkBxIgZLiM9BCgY9fA89PHb)


1. 최초 **master** 브랜치에서 시작
2. 동일한 브랜치를 **develop**에도 생성. 개발자들은 이 develop 브랜치에서 개발 진행
3. 개발 도중 특정 기능 구현이 필요할 경우, 해당 기능의 **feature** 브랜치를 생성해 기능을 구현
4. 완료된 feature 브랜치는 검토를 거쳐, **develop** 브랜치에 통합(Merge)
5. 모든 기능이 완료되면 develop 브랜치를 **release** 브랜치로 생성. QA를 진행하며 보완 및 디버깅
6. 완료되면 release 브랜치를 **master** 브랜치와 **develop** 브랜치로 이관. **master** 브랜치에서 버전 추가를 위해 태그를 하나 생성하고 배포
7. 배포 후 발견된 버그가 있을 경우, **hotfixes** 브랜치를 만들어 긴급 수정 후 태그를 생성하고 바로 수정 배포

> 브랜치를 Merge할 때는 항상 `--no-ff` 옵션을 붙여서 브랜치에 대한 기록이 사라지는 것을 방지
> ![img2](https://drive.google.com/uc?id=1y_iAEPDwBtiLLB5ROVIZ4o2zJZrvQHi_)

> [git command 참고](https://cslee94.github.io/datascience/2021/10/13/dev-git/)
<br><br>

# Git Flow 사용 예시

## git flow 초기화
1. `.git`파일이 있는, git repository에서 git flow 초기화 명령어를 실행한다.
    ```vim
    git flow init
    ```
    
    해당 명령어를 실행하면, git flow 전략에 맞는 브랜치들을 자동으로 생성해준다. 각 단계에서 생성되는 브랜치의 이름을 확인하도록 요청하고, Enter키를 눌러 기본값으로 생성한다.

    ```vim
    $ git flow init ☜ 플로우 초기화

    Which branch should be used for bringing forth production releases?
        - master
    Branch name for production releases: [master] ☜ 기본 브랜치
    Branch name for "next release" development: [develop] ☜ 기본 브랜치

    How to name your supporting branch prefixes?
    Feature branches? [feature/] ☜ 기본 브랜치
    Bugfix branches? [bugfix/] ☜ 기본 브랜치
    Release branches? [release/] ☜ 기본 브랜치
    Hotfix branches? [hotfix/] ☜ 기본 브랜치
    Support branches? [support/] ☜ 기본 브랜치
    Version tag prefix? []
    Hooks and filters directory? [E:/gitstudy13/.git/hooks]
    ```

    > 생성 확인이 귀찮다면 -d 옵션을 사용할 수 있다.
    >   ```vim
    >   git flow init -d
    >   ```

<br>

## 브랜치 확인 & 동기화
`git branch -v` 명령어를 통해 생성된 브랜치들을 확인한다.
```vim
git branch -v
* develop 29990c7 first ☜ 체크아웃
  master  29990c7 first
```
원본 소스를 유지하는 **master** 브랜치와 개발 작업을 위한 **develop** 브랜치가 생성된 것을 확인할 수 있고, 자동으로 develop 브랜치로 자동 체크아웃된 것을 확인할 수 있다. 타 git flow의 브랜치는 처음부터 자동 생성되지 않고, 필요 시 생성해 작업한다.

**로컬**에서 초기화된 master 브랜치 & develop 브랜치와 원격 저장소를 동기화를 진행한다.
```vim
git push -u origin develop
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'develop' on GitHub by visiting:
remote:      https://github.com/jinygit/gitstudy13/pull/new/develop
remote:
To https://github.com/jinygit/gitstudy13.git
 * [new branch]      develop -> develop ☜ 원격 브랜치 생성
Branch 'develop' set up to track remote branch 'develop' from 'origin'.
```

<br>

## git flow branch 사용법 - feature
버그 수정이나 기능추가를 위한 feature를 생성하기 위해 다음 명령을 실행한다.
```vim
git flow feature start <feature name>
```

생성된 feature 브랜치에서 평소 git을 이용하는 것처럼 개발을 진행하고, 작업이 끝난 이후 feature 브랜치를 마무리할 땐, 다음 명령어를 실행한다.

```vim
git flow feature finish <feature name>
```

이 명령어를 실행하면, git-flow가 develop 브랜치로 checkout한 다음 feature 브랜치의 내용을 병합 후 feature 브랜치를 삭제한다.

<br>

## git flow branch 사용법 - release
release 브랜치를 생성하기 위해 다음 명령어를 실행한다.

```vim
git flow release start <version>

```
이 명령어를 실행하면 release/version이라는 release 브랜치가 생성된다. 이후 release 준비가 끝났으면 finish 명령어를 통해 마무리한다.

```vim
git flow release finish <version>
```


<br><br>

# Rerference

>   - [Gitflow 원본](https://nvie.com/posts/a-successful-git-branching-model/)
>   - [우아한형제들 기술 블로그](https://techblog.woowahan.com/2553/)
>   - [UX공작소](https://ux.stories.pe.kr/183)
>   - [Json님의 포스트](https://blog.gangnamunni.com/post/understanding_git_flow/)
>   - [Hackernoon님의 포스트](https://hackernoon.com/gitflow-is-a-poor-branching-model-hack-d46567a156e7)
>   - [Git Textbook](https://git.jiny.dev/gitflow/init)
>   - [킴루코님의 블로그](https://hbase.tistory.com/60)
