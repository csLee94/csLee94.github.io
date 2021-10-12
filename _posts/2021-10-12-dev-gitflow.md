---
layout: post
title:  "Git Flow"
subtitle:   Git Flow
categories: datascience
tags: dev git gitflow
comments: true
---

# 목차
1. [Git Flow란](#git-flow란)
2. [Git Flow 흐름](#git-flow-흐름)
0. [Reference](#reference)

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

# Git Flow 흐름
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

<br><br>

# Rerference
- [Gitflow 원본](https://nvie.com/posts/a-successful-git-branching-model/)
- [우아한형제들 기술 블로그](https://techblog.woowahan.com/2553/)
- [UX공작소](https://ux.stories.pe.kr/183)
- [Json님의 포스트](https://blog.gangnamunni.com/post/understanding_git_flow/)
- [Hackernoon님의 포스트](https://hackernoon.com/gitflow-is-a-poor-branching-model-hack-d46567a156e7)

