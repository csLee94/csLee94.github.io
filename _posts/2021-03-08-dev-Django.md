---
layout: post
title:  "Django"
subtitle:   Django
categories: datascience
tags: Django webframework dev
comments: true
---

# Content 
1. [Django란?](#1-django)
2. [Django의 구성요소](#2-django의-구성요소)

4. [참고자료](#참고-자료)
---

<br>

---
# 1. Django
Djagno란 파이썬으로 만들어진 무료 오픈소스 웹 애플리케이션 프레임워크(Web Application Framework)입니다. 

<br>

## Django 시작하기
> django 설치 후 `django-admin startproject mysite` 명령을 통해 프로젝트를 시작할 수 있다. 이 때 mysite는 프로젝트의 이름입니다. 

- 프로젝트 시작 명령을 통해 현재 디렉토리에서 mysite라는 디렉토리가 생성됩니다. 
    ~~~~
    mysite/
        manage.py
        mysite/
            __init__.py
            setting.py
            urls.py
            asgi.py
            wsgi.py
    ~~~~
    - **manage.py**: Django 프로젝트와 다양한 방법으로 상호작용하는 커맨드라인의 유틸리티. 이 파일을 실행시켜 웹 서버를 확인합니다.
    - **mysite/**: 디렉토리 내부에 프로젝트를 위한 실제 Python 패키지들이 저장된다. mysite.url와 같은 식으로 이 디렉토리 내의 이름을 이용하여 프로젝트의 어디서나 Python 패키지들을 임포트할 수 있습니다. 
    - **mysite/__init__**: 디렉토리를 패키지처럼 다루라고 알려주는 용도의 빈 파일
    - **mysite/setting.py**: 현재 Django 프로젝트의 환경 및 구성을 저장
    - **mysite/urls.py**: 현재 Django 프로젝트의 URL 선언을 저장

- 작업을 위한 프로젝트 환경을 설치하고 나서 앱(APP)을 생성해줍니다. 
```vim
$ python manage.py startapp app_name
```
<br><br>

# 2. Django의 구성요소

- ## Django의 구조
    |||
    |---|---|
    |1. 프로젝트 생성||
    |2. 앱 생성||
    |[3. 프로젝트와 앱 연결](#1-settings.py) |**setting.py**|
    |[4. template 생성](#2-template-생성) |**.html**|
    |[5. 앱 기능 구현](#3-앱-기능-구현) |**views.py**|
    |[6. URL 요청을 views에 연결](#4-URL과-View-연결) |**urls.py**|
    |[7. 서버 실행](#5-서버-실행)|**manage.py**|

<br><br>



## 1. settings.py
app을 만들었지만 project는 아직 그 app의 존재를 모릅니다. 그래서 우리는 app을 만들었다고 등록해주는 절차가 필요합니다.
>**project_name/settings.py** 수정

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
INSTALLED_APPS 영역에 '`app_name`.apps.`app-name`Config'를 추가합니다. `app_name`/apps.py 파일을 열어보면
```python
from django.apps import AppConfig


class app_nameConfig(AppConfig):
    name = 'app_name'
```
이렇게 `app_name`Config라는 클래스가 정의되어 있는데 이것을 등록해주는 절차입니다. 이제 Django는 app이 포함된 것을 알게 되었습니다. 
```vim
$ python manage.py makemigrations app_name
```
**makemigrations**를 실행시킴으로서 우리가 모델을 변경시킨 사실과 이 변경 사항을 *migration*으로 저장시키고 싶다는 것을 Django에게 알려줍니다.

<br>

## 2. template 생성
template은 유저가 보는 화면입니다. app 폴더 안에 templates 폴더를 만들고 templates 내부에 HTML 파일들을 넣어줍니다.

>**app_name/templates/helloworld.html**
```html
<h1>
    Hello, World!
</h1>
```

<br>

## 3. 앱 기능 구현
view.py 파일에서 앱의 기능을 구현할 수 있습니다. 

<br>

## 4. URL과 View 연결
>**app_name/urls.py**

1. app 폴더 안에 있는 views.py의 함수를 import해옵니다.
```python
from . import views
```
2. urlpatterns에 path를 추가합니다. 
```python
urlpatterns = [
    path('',views.index, name='index'),
]
```

### +)urlpattern 작성법
path() 함수는 3가지 인수를 받습니다.
```python
path(경로, view에 정의된 함수, 이름)
```
- **경로**<br> 도메인 뒤에 붙는 url 부분입니다. 예를 들어 `http://127.0.0.1:8000/` 뒤에 `admin/`을 입력하면 어드민 페이지가 나오게 됩니다. 위에서 추가한 helloworld.html과 연결되는 path는 `''`로 아무것도 적지 않았습니다. 그러므로 HTML이 기본 홈 화면이 됩니다.

- **View에 정의된 함수**<br> app_name 폴더 속 views.py 파일 속 helloworld라고 정의된 함수를 실행시키겠다는 의미입니다. 

- **이름**<br> `name='helloworld`는 이 path의 이름을 helloworld라고 정의합니다. 이렇게 정의된 이름은 Django 프로젝트 어디에서든 helloworld라고 불러서 호출할 수 있습니다. 
    

## 5. 서버 실행
이제 터미널에서 아래 명령어를 통해 어떤 웹사이트가 만들어졌는지 확인할 수 있습니다. 
```python
$ python manage.py runserver
```
<br>

이렇게 Django는 `settings.py`, `html`, `views.py`, `urls.py`, `manage.py` 이 다섯 가지 파일들끼리 서로 데이터를 어떻게 처리하고 연결하는지 흐름을 이해하면 됩니다.



<br><br>

# 참고 자료
- [HonKit](https://tutorial.djangogirls.org/ko/django_start_project/)
- [docs.djangoproject](https://docs.djangoproject.com/ko/3.1/intro/tutorial01/)
- [tothefullest08님의 블로그](https://tothefullest08.github.io/django/2019/02/11/django01/)