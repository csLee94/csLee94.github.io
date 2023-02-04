---
title: Github Pages로 블로그 만들기(feat.Chirpy) [1]
author: csLee94
date: 2023-01-23 00:00:00 +0900
categories: [others]
tags: [github pages, blog, jekyll]
series:
    Github Pages로 블로그 만들기(feat.Chirpy) [2] - ga, google search console, hits 설정: 
        url: update-blog-2
---


> 21년, github와 jekyll 블로그를 개설해서 잠시 운영해본 적이 있습니다. 다만 이전에 설치했던 내용과 많이 달라지기도 했고, theme도 업데이트할 필요가 있어 재구축을 시도하며
 사소한 에러들을 해결하는데 꽤 애를 먹었습니다. 때문에 제가 잊지 않기 위해 남기는 내용이며, 누군가에겐 아주 작은 도움이라도 됐음 좋겠습니다.

<br>

## Why github page?

네이버 블로그부터 티스토리까지, 다양한 블로그 플랫폼이 있지만 github pages를 선택한 이유는 크게 두 가지입니다.
> 1. 프로젝트를 진행하며 `README.md`를 만들면서 markdown으로 문서 정리를 하는 것이 꽤 익숙해지기도 했고, 회사나 개인 프로젝트 진행하며 많이 사용하는 Notion 내용도 markdown으로 export해 업르도하는 것이 굉장히 편했습니다.
> 2. github의 profile page의 잔디심기가 가능했습니다. 사실 별거 아닌 문제일 수 있긴 하지만, 심어진 잔디가 주는 만족감과 github repository page를 자주 왔다갔다하면서 프로젝트로부터 멀어지지 않을 수 있다면 충분했습니다.
{: .prompt-tip }

<br>

## How to build Blog on github

### Setting prerequirements
github pages를 이용하기 위해서 몇 가지 설치가 필요합니다.

우선 **git**을 설치해줍니다.

다음으론, **ruby**를 설치해줍니다. Mac은 [여기]( https://jekyllrb.com/docs/installation/macos/)를 참고해 설치해주시고, 윈도우의 경우 아래 사진에서 **WITH DEVKIT** section에서 `=>` 표시가 있는 버전으로 설치해주시면됩니다.
![img](/assets/img/blog-update/img_1.png)
_빨간 네모 참고_

> 이 때, ruby 언어를 반드시 알 필요는 없습니다. `jekyll`이 ruby로 만들어져, 환경 설정이나 포스팅 결과를 미리 확인하기 위해 로컬에서 실행하실 때만 사용되기 때문입니다. 심지어 local 환경에서 테스트가 필요없으시다면 ruby를 설치하실 필요도 없긴 하지만, 이는 권장하지 않습니다!
{: .prompt-warning }

마지막으로, 저는 블로그 에디터로 **VScode**를 사용합니다. 별도의 에디터를 사용하시는 게 아니라면, vscode도 설치해주세요.

### Select jekyll theme
먼저 아래 사이트에서, 원하는 블로그 테마를 선택합니다. 이 때, github repository에서 readme 파일을 확인해보시거나 Demo site 내 Docs 설명이 충분한지 살펴보는 것을 권해드립니다. 아무래도 설명이 부족한 테마를 선택하시게 되면, 설정 난이도가 확 올라가버리게 됩니다. 구글에 해당 테마를 검색해본 다음, 참고할만한 레퍼런스가 충분한지 확인해보시는 것도 좋은 방법입니다.[^footnote_1]
> - [http://jekyllthemes.org/](http://jekyllthemes.org/)
> - [https://jekyllthemes.io/free](https://jekyllthemes.io/free)
> - [http://themes.jekyllrc.org/](http://themes.jekyllrc.org/)
> - [https://github.com/topics/jekyll-theme](https://github.com/topics/jekyll-theme)

참고로 저는, 처음엔 `Hydejack` 이라는 테마를 사용했으며 지금은 `Chirpy` 라는 테마를 사용하고 있습니다. (유료 버전을 포함할 때) 다양한 기능들과 화려한 UI에 반해 `Hydejack`이라는 테마로 시작했지만, 유료버전 전용 기능부터 자잘한 기능을 제가 다루기엔 버겁다고 느꼈습니다. 또한 블로그 셋팅에 너무 많은 시간을 들이게 되어, 글쓰기 자체에 집중하고자, 조금 더 Compact한 현재 테마로 변경했습니다.

### Getting Source Code of themes
> 아래 내용부터는 현재 제 블로그 테마인 `Chirpy` 테마를 기준으로 진행합니다.

자, 마음에 드시는 테마를 결정하셨다면 자신의 github에 새로운 repository를 만드셔야합니다. 이 때, 두 가지 방법을 사용할 수 있습니다.

#### option 1. Forking on Github
가장 간단한 방법입니다. 각 테마의 offical repository를 fork하셔도 되고, 아니면 다른 분들께서 사용 중인 repository를 fork하셔도 됩니다. 다만 이 경우 몇 가지 유의하실 사항이 있습니다. 
> 1. Fork 기능을 이용 시, "잔디심기"가 불가합니다.
> 2. 만약 특정 User의 repository를 fork하셨다면, GA나 social 계정 등 개별 설정 파일들을 모두 찾아 수정해주셔야 합니다.
{: .prompt-warning}

반면에 아주 강력한 장점도 있습니다.
> 1. 블로그 주인께서 개별적으로 custom한 내용까지 블로그에 적용할 수 있습니다.
> 2. setting 과정에서 마주할 수 있는 사소한 error로 부터 자유롭습니다.
{: .prompt-info}

#### option 2. Download Starter Pack or Repository
새로운 Repository를 만들고, `starter pack` 혹은 `starter kit`이라고 있는 Source Code를 다운로드해 commit하는 방식입니다. 물론 starter pack이 아닌 현재 사용 중인 Repository를 local에 clone하고, 만드신 repository에 copy&poste를 통해 commit도 가능합니다.


두 가지 방법을 통해, 자신의 repository에 성공적으로 Source code를 받으셨다면, 아래 사진과 같을 겁니다.
![img](/assets/img/blog-update/img_2.png)
_example of repository page_

이 때 repository 이름이 ${username}.github.io인지 확인해주세요! 만약 이름이 다르다면, `Settings` tab에서 General 메뉴 중 **Repository name**에서 변경하실 수 있습니다. github username으로 설정한 이름 뒤에 .github.io를 붙여서 완셩해주세요.

자, 이 상태로 git을 통해 해당 repository에 commit하고 잠시 기다리면[^footnote_2] `https://${username}.github.io/` 주소에서 Blog를 확인하실 수 있습니다. 


### Initializing of Chirpy
기본적인 준비를 마쳤으니, 본격적으로 Blog를 내 것으로 만듭니다. Vscode terminal에서 아래 명령을 사용해, chirpy를 초기화합니다.

```terminal
$ bash tools/init
[INFO] Initialization successful! # 이 메세지가 출력되면 성공
```

해당 명령어를 실행하면, chirpy 개발에 필요한 파일들이나 포스트 파일들이 삭제됩니다. 

그리고 아래 명령어를 통해 의존성이 있는 모듈을 모두 설치합니다. 이미 chirpy 개발자 분들이 사전 정의해뒀기 때문에, 아래 명령만 실행시키시면 됩니다.

```terminal
$ bundle
```

초기화 과정이 정상적으로 진행됐는지 확인해보시려면, terminal에 아래 명령어를 실행시킵니다.

```terminal
$ bundle exec jekyll serve

... # 아래 메세지 출력시 성공
    Server address: http://127.0.0.1:4000
Server running... press ctrl-c to stop
```

브라우저에 http://127.0.0.1:4000 주소를 입력했을 때 기본 블로그 화면이 나타나면 성공입니다.

<br>

## Customizing
> 본격적으로 Customizing하기 전에 **꼭!** 아래 공식 Docs를 확인해보실 것을 권장드립니다!
> [https://chirpy.cotes.page/posts/getting-started/](https://chirpy.cotes.page/posts/getting-started/)
{: .prompt-danger}

소스를 확인해보시면, 굉장히 많은 파일들이 생성되어 있을 텐데, 우선은 기본 설정에는 몇 가지 항목만 알면 됩니다. 전반적으로 세부 설정에 필요한 상세 내용은 [여기](https://jekyllrb.com/docs/)에서 확인할 수 있으며, 다음 포스트에서 다뤄 보겠습니다.

|file Name|Description|
|---|---|
|**_config.yml**| 블로그의 기본 설정 파일로, 기초적인 setting은 이 파일만으로도 할 수 있습니다.|
|_data/authors.yml| username, url 등을 설정합니다. |
|_data/contact.yml| sidebar 하단 영역에 들어갈 github 주소, 메일 등을 관리할 수 있습니다.|
|_posts/| 작성할 포스트들이 저장될 곳입니다. `yyyy-mm-dd-name.md` 형식으로 파일을 저장합니다.|
|assets/| 블로그 로고 사진, favicon 부터 포스트에 사용될 이미지들을 저장하는 곳입니다.|

우선은, 간단하게 `_config.yml` 파일부터 수정해보겠습니다. 부분적으로 바꾸어야할 부분만 살펴보고, 제 블로그와 [_config.yml](https://github.com/csLee94/csLee94.github.io/blob/main/_config.yml)를 비교해보시면 다른 부분들도 참고하실 수 있습니다.

```yaml
timezone: Asia/Seoul
title: csLee94 # side bar 영역 내 logo 아래 큰 글씨 영역입니다.
tagline: We are what we repeatedly  do ... # 그 아래 작은 글씨 영역입니다.
url: https://cslee94.github.io
github:
    username: csLee94
social:
    name: csLee94
    email: cslee94@gmail.com
    links: # 사용 중인 SNS의 link를 추가합니다.
        - https://github.com/csLee94

google_site_verification: l5a... # google search console에서 메타 tag string을 입력
avatar: /assets/img/me_128.jpg # sidebar 영역의 logo 이미지
```
{: file='_config.yml'}

그 밖에 google_analytics나, comments와 같은 부분들은 `_config.yml` 내에서 설정할 때 에러가 발생했고, 이 부분은 아직 해결할 방법을 찾지 못했습니다. 그래서 해당 파일이 아닌 직접 html 소스를 추가했는데, 이 부분은 다음 포스트에서 다루도록 하겠습니다.

바뀐 내용을 로컬에서 확인하실 땐, 위와 동일하게 `bundle exec jekyll serve` command를 실행 후 http://127.0.0.1:4000에서 확인가능합니다. 다만, _confing.yml파일을 수정했을 땐 항상 종료 후 다시 실행시켜주셔야 합니다.

```terminal
$ bundle exec jekyll serve

... # 아래 메세지 출력시 성공
    Server address: http://127.0.0.1:4000
Server running... press ctrl-c to stop
```

<br>

## Writing first Post
vscode를 통해 첫 번째 글을 작성해보겠습니다. _posts/ 폴더 아래에 `yyyy-mm-dd-name.md` 형식의 이름으로 md 파일을 생성합니다. 만약 name 부분에 띄어쓰기가 필요하다면, 공백 없이 `-`으로 구분짓습니다.

저는 *2023-01-23-hello-world.md* 이름으로 파일을 생성했습니다.

```markdown
---
title: Hello world   *post의 제목입니다.*
author: csLee94   *_data/authors.yml과 _config.yml 파일에서 설정한 내용을 토대로 링크가 생성됩니다.*
date: 2019-08-08 14:10:00 +0800   *post page에서 posted 내용입니다.*
categories: [Blogging, Tutorial]   *sidebar의 category에 반영됩니다. 앞에 쓸 수록 상위 카테고리로 인식됩니다.*
tags: [writing]   *sidebar의 tag 반영됩니다.*
math: True   *mathJax를 이용해 수식을 표현합니다.*
mermaid : True   *mermaid를 이용해 diagram*을 표현합니다.*
---

## Hello world
toc 기능에 따라 header들이 우측에 Contents 영역에 표시됩니다.

```
{: file='_posts/2023-01-23-helloworld.md'}

작성을 마치고, 로컬에서도 문제가 없다면 git을 통해 진짜 블로그에 반영해 볼 차례입니다.

```terminal
$ git add .
...
$ git commit -m "first commit"
...
$ git push
```

잠시 기다리시고, 저희 블로그 주소인 `https://${username}.github.io`로 접속해보시면, 작성하신 포스트를 확인하실 수 있습니다.

<br>

## Wrap up
> 항상 기본 테마는 뭔가 아쉽고, 그렇다고 Front source를 손보는 것은 너무 부담되는 일이었는데 Chirpy 테마는 원했던 기능들이 대부분 잘 구현되어 있어 굉장히 좋았습니다. 물론 조금 더 복잡했던 기존 "Hydejack" 테마와 비교했을 때 몇몇 기능들이 빠진 것이 못내 아쉽기는 하지만요. <br>
> 하지만, category&tag 관리의 용이성과 toc 기능, 검색 기능 등 제가 생각했던 필수기능들이 대부분 minimal하게 잘 구현된 것 같아 굉장히 만족합니다! 이제 Customizing 지옥에서 벗어나, posting 자체에 조금 더 집중해봐야겠습니다.

<br>

## APPEDIX. jekyll이란?
Jekyll 이란 github 설립자 중의 한명이 Ruby 언어를 통해 개발한 Framework입니다. jekyll의 핵심은 HTML/MARKDOWN 등의 마크업 언어로 글을 작성하면, 이를 사전 정의된 레이아웃으로 포장해, 정적 웹사이트로 만들어줍니다. 

> 저희 블로그 repository에 `_site/` 폴더 아래서 만들어지는 정적 웹사이트를 확인할 수 있습니다.
{: .prompt-tip }

서버 리소스 없이, 정적 파일만으로 사이트를 Build하기 때문에 빠르고 가볍다는 장점이 있고 복잡한 로직이 없는 블로그와 같은 사이트에 적합하다고 볼 수 있습니다.

나중에, jekyll을 이용한 블로그에 익숙해지고 좀 더 디테일한 customizing을 할 때 jekyll의 기본 directory 구조를 알아 두시면, 소스를 분석하는데 훨씬 수월해집니다.

[여기](https://jekyllrb.com/docs/)에서 확인해보실 수 있고, 이 내용도 나중에 기회가 되면 정리해보겠습니다!

<br>

## reference
> - [Docs of jekyll](https://jekyllrb.com/docs/)
> - [jekyll이란?](https://cheershennah.tistory.com/214)
> - [jekyll Chirpy Getting Started](https://chirpy.cotes.page/posts/getting-started/)
> - [Jekyll Chirpy 테마 사용하여 블로그 만들기](https://www.irgroup.org/posts/jekyll-chirpy/)

<br>

---
[^footnote_1]: github pages는 public repository를 사용합니다. 해당 테마를 사용하고 있는 블로그들이 많다면, 그 블로그의 repository를 통해 source code를 참고하시면, 조금 더 쉽게 설정하실 수 있습니다.

[^footnote_2]: repository에 commit하면, `github/workflow/pages-deploy.yml`에 설정되어 있는 내용에 따라 블로그 배포 로직이 시작됩니다. 해당 내용은 repository page의 **Actions** tab에서 확인하실 수 있습니다만, 지금 당장 중요한 내용은 아닙니다.