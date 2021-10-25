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
3. [Git Flow 사용 예시](#git-flow-사용-예시)
4. [Reference](#reference)

<br>

---

<br><br>


# Git Flow란
Git Flow는 **Vincent Driessen**에 의해 고안된 방법론이다. <br> Git으로 개발할 때 거의 표준과 같이 사용되지만, Vincent Driessen이 언급했듯, 완벽한 방법론은 아니고 각자 개발 환경에 따라 수정하고 변형해서 사용한다.


Git Flow는 5가지의 브랜치를 사용한다. <BR>
항상 유지되는 메인 브랜치(master, develop)과 일정 기간 동안만 유지되는 보조 브랜치(feature, release, hotfix)가 있다.
> - master: 배포(Release) 이력을 관리하기 위해 사용. 즉, 배포 가능한 상태만을 관리하는 브랜치
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
- 기본 git flow는 로컬에서만 작동한다. (Merge, 브랜치 생성 등)
- **pull-request**사용 여부 등 중앙 원격 저장소에 push하는 것은 프로젝트별 정책을 따른다.

## git flow 초기화
1. `.git`파일이 있는, git repository에서 git flow 초기화 명령어를 실행한다.
    ```vim
    $ git flow init
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
    >   $ git flow init -d
    >   ```

<br>

## 브랜치 확인 & 동기화
`git branch -v` 명령어를 통해 생성된 브랜치들을 확인한다.
```vim
$ git branch -v
* develop 29990c7 first ☜ 체크아웃
  master  29990c7 first
```
원본 소스를 유지하는 **master** 브랜치와 개발 작업을 위한 **develop** 브랜치가 생성된 것을 확인할 수 있고, 자동으로 develop 브랜치로 자동 체크아웃된 것을 확인할 수 있다. 타 git flow의 브랜치는 처음부터 자동 생성되지 않고, 필요 시 생성해 작업한다.

**로컬**에서 초기화된 master 브랜치 & develop 브랜치와 원격 저장소를 동기화를 진행한다.
```vim
$ git push -u origin develop
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

## git flow 전반적인 사용 플로우



<br>

## Branch별 상세
> 아래선 git flow command가 아닌 git command 위주로 설정한다.
---
### Master Brench
제품으로 출시될 수 있는 브랜치이며, 배포(Release) 이력을 관리하기 위해 사용한다. 즉, 배포 가능한 상태만을 관리한다.

<br>

### Develop Branch
다음 출시 버전을 개발하는 브랜치이며, 기능개발을 위한 브랜치들을 위해 사용한다. 모든 기능이 추가되고, 디버깅이 완료된 안정적인 상태라면 develop 브랜치를 **'Master Branch'**에 병합(merge)한다.<br>
평소에 이 브랜치에서 개발을 진행한다.

<br>

### Feature Branch
새로운 기능 개발 및 버그 수정이 필요할 때마다 'Develop Branch'로부터 분기해 사용한다. 일반적으로 Feature Branch에서의 작업은 공유할 필요가 없기 때문에 자신의 로컬 저장소에서 관리하고, 개발이 완료되면 'develop' 브랜치로 병합(Merge)해 다른 사람들과 공유한다.

- feature branch 생성 및 종료 과정
    ```vim
    $ git checkout -b feature/temp develop # temp 기능 구현을 위한 브랜치를 분기

    # 새로운 기능에 대한 작업 수행

    $ git checkout develop # develop 브랜치로 이동

    $ git merge --no-ff feature/temp # feature 브랜치에 존재하는 커밋이력을 모두 합쳐서 develop 브랜치로 병합

    $ git branch -d feature/temp # feature/temp 브랜치 삭제

    $ git push origin develop # develop 브랜치를 원격 중앙 저장소에 올린다.
    ```

<br>

### Release Branch
배포를 위한 전용 브랜치를 사용함으로 한 팀이 해당 배포를 준비하는 동안, 다른 팀은 다음 배포를 위한 기능 개발을 계속할 수 있다. 
1. Release 브랜치를 develop 브랜치에서 분기한다.
    - **develop 브랜치에서 배포할 수 있는 수준의 기능이 모이면** release 브랜치를 분기하며, 배포를 위한 최종 작업(버그 수정, 문서 추가)을 제외하고는 추가로 merge하지 않는다.
    - release 브랜치를 만드는 순간부터 배포 사이클이 시작된다.
2. **모든 기능이 정상적으로 동작**하면 release 버전 태그를 부여해 'master' 브랜치에 병합한다.<br>
배포를 준비하는 동안 release 브랜치가 변경되었을 수 있으니, 배포 완료 후 'develop' 브랜치에도 병합한다.

> release 브랜치 이름은 `release-*` 또는, `release/*`와 같이 짓는 게 일반적이다.

- release branch 생성 및 종료 과정
    ```vim
    $ git checkout -b release/v1.1.3 develop # develop 브랜치로부터 release 브랜치를 분기

    # 배포 사이클이 시작되며, 배포 가능한 상태가 되면

    $ git checkout master 

    $ git merge --no-ff release/v1.1.3 #master 브랜치에 release/v1.1.3 브랜치 merge

    $ git tag -a v1.1.3 # 병합한 커밋에 release 버전 태그 부여

    $ git push origin master # master브랜치를 중앙 원격 저장소에 업로드

    $ git push origin master --tags # tag push
    
    # release 브랜치의 변경 사항을 develop 브랜치에도 적용

    $ git checkout develop

    $ git merge --no-ff release/v1.1.3

    $ git push origin develop # merge된 develop 브랜치를 중앙 원격 저장소에 업로드

    $ git branch -d release/v1.1.3 # release/v1.1.3에 해당하는 브랜치 삭제

    ```

<br>

### Hotfix Branch
배포한 버전에 긴급하게 수정해야할 경우, 'master' 브랜치에서 분기하는 브랜치이다. 'develop'브랜치에서 수정 & 배포하기엔 시간도 많이 소요되고, 안정성을 보장하기도 어려워 바로 배포가 가능한 'master' 브랜치에서 분기해 필요한 부분만을 수정 후 다시 'master' 브랜치에 병합해 배포하는 것이다.
> 'hoxfix' 브랜치는 'master' 브랜치를 부모로 하는 임시 브랜치로, 다음 배포를 위해 개발하던 작업 내용에 전혀 영향을 주지 않는다.

- hotfix branch 생성 및 종료 과정
    ```vim
    $ git checkout -b hotfix/v1.1.4 master # hotfix 브랜치를 master 브랜치에서 분기

    # 문제 부분 수정

    $ git checkout master
    
    $ git merge --no-ff hotfix/v1.1.4 # master 브랜치에 hotfix/v1.1.4 내용을 병합

    $ git tag -a 1.1.4 # 병합한 커밋에 새로운 버전 이름으로 태그 부여

    $ git push origin master # master 브랜치 중앙 원격 저장소에 업로드

    # hotfix 브랜치 변경 사항 develop 브랜치에도 적용

    $ git checkout develop

    $ git merge --no-ff hotfix/v1.1.4

    $ git push origin develop # develop 브랜치 중앙 원격 저장소에 업로드

    $ git branch -d hotfix/v1.1.4
    
    ```


<br><br>

# Reference

>   - [Gitflow 원본](https://nvie.com/posts/a-successful-git-branching-model/)
>   - [우아한형제들 기술 블로그](https://techblog.woowahan.com/2553/)
>   - [UX공작소](https://ux.stories.pe.kr/183)
>   - [Json님의 포스트](https://blog.gangnamunni.com/post/understanding_git_flow/)
>   - [Hackernoon님의 포스트](https://hackernoon.com/gitflow-is-a-poor-branching-model-hack-d46567a156e7)
>   - [Git Textbook](https://git.jiny.dev/gitflow/init)
>   - [heejeong Kwon님의 블로그(1)](https://gmlwjd9405.github.io/2018/05/11/types-of-git-branch.html)
>   - [heejeong Kwon님의 블로그(1)](https://gmlwjd9405.github.io/2018/05/12/how-to-collaborate-on-GitHub-3.html)
>   - [outsider님의 블로그](https://blog.outsider.ne.kr/644)
>   - [jinwoo1990님의 블로그](https://jinwoo1990.github.io/git/git-flow-tutorial/)