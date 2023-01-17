---
layout: post
title:  "Github를 이용한 블로그 개설하기"
subtitle:   Github를 이용한 블로그 개설하기
categories: datascience
tags: other github jekyll
comments: true
---
# 목차
1. [Jekyll 블로그 설치](#jekyll-블로그-설치)
2. [Jekyll 블로그 작성](#jekyll-블로그-작성)
3. [Jekyll 블로그에 Jupyter Notebook 업로드](#jekyll-블로그에-jupyter-notebook-업로드)
<br>

# Jekyll 블로그 설치
이 글은 대부분 아래의 사이트에서 참고하였습니다.
> 참고사이트: [Theory DB Blog](https://theorydb.github.io/envbunops/2019/05/03/envops-blog-github-pages-jekyll/)<br>


0. Blog 개요<br>
다음과 같은 이유로 여러 블로그 중 이 블로그로 선택
    - github.io 형태의 URL로 제공되는 Jekyll 기반의 Github Pages 블로그
    - 여러 자료를 정리할 때 내부이동/링크 삽입 등의 이유로 MD 형식을 이용하는데 이것을 그대로 블로그에 업로드 가능
    - Git contribution 가능    
    
1. 설치
    1. Git
        - Git을 설치
        - Repository 개설<br>
        이 때 Repsitory의 이름은 Github의 User Name으로 설정
        - PC 상에 Clone 
    <br>

    2. ruby
        - Jekyll은 ruby로 만들어졌기 때문에 ruby를 설치
        - ruby언어를 몰라도 가능
        - [https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)에 접속하여 다운로드
        - ruby 프롬프트 상에서 `chcp 65001`을 입력하고 PC 내 Clone 했던 위치로 이동
        - `gem install bundler jekyll minima jekyll-feed tzinfo-data rdiscount` 명령어를 통해 Jekyll에 종속된 필요한 라이브러리를 설치
        > **여기서 오류가 발생한다면?**<br>
        > * 오류에서 말하는 버전에 맞는 라이브러리를 재설치
        > * bundle install

2. Blog 꾸미기 및 환경설정
    1. 아래 사이트에 접속하면 무료로 제공되는 테마를 확인 가능
        - [http://jekyllthemes.org/](http://jekyllthemes.org/)
        - [https://jekyllthemes.io/free](https://jekyllthemes.io/free)
        - [http://themes.jekyllrc.org/](http://themes.jekyllrc.org/)
        - [https://github.com/topics/jekyll-theme](https://github.com/topics/jekyll-theme)
        > Blog Template을 선정 시 Tip<br>
        > 최대한 설명이 자세히 적혀 있는 것을 고르자.<br>
        > 아무것도 모르는 상태에서 환경설정에 대한 설명이 부족한 Template을 선택하면 입문 난이도가 확 올라간다.
    
    <br>

    2. 원하는 Tempalte을 선택했다면, 해당 Repository를 fork
        - fork 기능을 사용하여 그대로 내 Repository에 복사해올 수 있다.
        > *다만 이 경우 Git에 Contribution이 카운팅 되지 않는다.* 

    <br>

    3. 환경설정
        1. 반드시 변경
            - _featured_tags/ : 카테고리 대분류 폴더
            - _featured_categories/ : 카테고리 소분류(태그) 폴더
            - _data/ : 개발자 및 운영자, 기타 정보 폴더 (author.yml 수정이 필요)
            - _config.yml : 가장 중요한 환경변수 설정 파일
            - README.md : GitHub 프로젝트 애서 소개하게 될 글
            - favicon.ico : 블로그 접속 시 브라우저 주소창에 표시되는 대표 아이콘
            - about.md : About 메뉴 클릭 시 나타나는 블로그에 대한 소개글
        2. 필요시 변경
            - assets/ : 이미지, CSS 등을 저장 폴더
            - _layouts/ : 포스트를 감싸기 위한 레이아웃 정의 폴더(페이지, 구성요소 등 UI변경 시 수정)
            - _includes/ : 재사용을 위한 기본 페이지 폴더
            - Gemfile.lock : Gemfile에 기록한 레일 기반 라이브러리를 설치 후 기록하는 파일(중복설치 방지)
            - Gemfile : 필요한 레일 기반 라이브러리를 자동으로 설치하고 싶을 때 명시하는 설정 파일
            - .gitignore : GitHub에 올리고 싶지 않은 파일들은 이 파일에 경로지정 가능(예: _site 산출물, 환경설정,   개인정보,  작성중인 글 등)
            - sitemap.xml : 테마의 사이트맵
            - search.html : Tipue Search 설치 시, 검색결과를 출력하는 페이지로 활용
            - robots.xml : 구글 웹로봇 등 검색엔진 수집 등에 대한 정책을 명시하는 설정파일
            - posts.md : 포스트 작성 관련 설정파일
        3. 변경 필요없음(참고)
            - _posts/ : 포스트를 저장하는 폴더
            - .git/ : GitHub 연동을 위한 상태정보가 담긴 폴더
            - _site/ : Jekyll 빌드 생성 결과물 폴더(실제 GitPages에서 WEB으로 보여지는 산출물)
            - .sass-cache/ : 레일 엔진에서 사용하는 캐시 저장폴더(변하지 않는 산출물들에 대한 파싱을 하지 않아 속도보장)
            - _sass/ : 일종의 CSS 조각파일 저장 폴더
            - _js/ : JavaScript 저장 폴더
            - _plugins/ : 플로그인 저장 폴더(크롬 정책상 어차피 사용안함)
            - LICENSE.md : 테마 개발자의 라이센스 설명
            - index.html : 블로그 최초 접속 페이지
            - googlea0d1f22cc8208170.html : 구글 검색엔진에 블로그를 등록하는 과정의 소유권 확인 파일
            - feed.xml : RSS Feed 활용을 위한 XML
            - browserconfig.xml : 윈도우8 이상 IE11 접속 시 클라이언트가 요청하는 환경설정 파일
            - 404.md : 404 Not Found Page(블로그에 없는 페이지 요청 시 등장하는 페이지)
            - .eslintrc : EcmaScript Lint(자바스크립트 협업 개발을 위한 규칙 정의) 환경설정 파일
            - .eslintignore : EcmaScript Lint 무시할 규칙 지정(전역변수 에러표시 예외처리 등)
            - .babelrc : Babel(자바스크립트 컴파일러) 설정파일

<br>

# Jekyll 블로그 작성
- Jekyll은 MarkDown으로 작성

### Markdown 문법
> [참고자료](https://heropy.blog/2017/09/30/markdown/)

### img 업로드 방법
1. Jekyll Repository 내 assets/img 디렉토리를 사용하여 이미지를 업로드 할 수 있다. 
    - 다만 github repository에 용량이 제한되어있다고 하는 등 나중의 용량에 대한 걱정이 있다. 

2. Google Drive를 활용하여 이미지를 업로드할 수 있다. 
    - 아래 사진과 같이 구글 드라이브에 폴더를 생성하고 `공유`기능을 통해 링크를 통해 뷰어 역할로 접근 가능하게 설정한다.
    ![img](https://drive.google.com/uc?id=1vr4H4JUaY4hASozMLOxFf3-c3RcOi7N3)
    ![img](https://drive.google.com/uc?id=1Sgl0HGkspo2yUHjKyIClC-I7GgvfhPXN)
    
    <br>

    - 해당 폴더에 이미지를 업로드하고 우클릭 후 `링크생성`을 누르면 공유가능한 링크를 복사할 수 있는 박스가 팝업된다.
    - 이 때 링크 복사를 누르면 `https://drive.google.com/file/d/[File Name]/view?usp=sharing`형태로 복사된다.
    - 복사된 위 주소를 `https://drive.google.com/uc?id=[File Name]`의 형태로 변경하여 작성하는 md파일에서 링크로 사용하면 된다.
        > 구글 드라이브 링크에 대한 자세한 내용은 아래 블로그에서 자세히 설명해두었다.
        https://namhoon.kim/2020/07/02/topic/001/index.html
        <br>
        https://seogilang.tistory.com/1253

<br>

# Jekyll 블로그에 Jupyter Notebook 업로드
## nbconvert를 이용하여 md파일로 변환
1. ipynb 파일이 저장된 경로에서 cmd를 통해 `jupyter nbconvert --to markdown (File_Name).ipynb`를 입력
2. (File_Name).md 파일이 변환된 것을 확인
3. (File_Name).md 파일 이름을 jekyll `_post`형식에 맞춰 변형하여 업로드
- 발생할 수 있는 문제
    01. Jupyter Notebook 내 markdown으로 작성한 표가 nbconvert를 거치면서 줄 사이사이 공백이 생김 -> 표 모양 망가진다.
    02. Jupyter Notebook 내에서 불러온 이미지에 대해 경로가 달라지므로 이미지를 불러오지 못한다.


