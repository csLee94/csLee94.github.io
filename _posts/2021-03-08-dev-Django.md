---
layout: post
title:  "Django"
subtitle:   Django
categories: dev
tags: Django webframework 
comments: true
---

# Content 
1. [Django란?](#1.-Django란?)
2. [Django의 구성요소](#2.-Django의-구성요소)
3. [Django의 동작순서]()
---

<br>

---
# 1. Django란?
Djagno란 파이썬으로 만들어진 무료 오픈소스 웹 애플리케이션 프레임워크(Web Application Framework)입니다. 

<br>

# 2. Django의 구성요소
> django 설치 후 `django-admin startproject mysite` 명령을 통해 프로젝트를 시작할 수 있다. 이 때 mysite는 프로젝트의 이름이다. 

- 프로젝트 시작 명령을 통해 현재 디렉토리에서 mysite라는 디렉토리가 생성된다. 
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
    - **manage.py**: Django 프로젝트와 다양한 방법으로 상호작용하는 커맨드라인의 유틸리티. 이 파일을 실행시켜 웹 서버를 확인한다.
    - **mysite/**: 디렉토리 내부에 프로젝트를 위한 실제 Python 패키지들이 저장된다. mysite.url와 같은 식으로 이 디렉토리 내의 이름을 이용하여 프로젝트의 어디서나 Python 패키지들을 임포트할 수 있다. 
    - **mysite/__init__**: 디렉토리를 패키지처럼 다루라고 알려주는 용도의 빈 파일
    - **mysite/setting.py**: 현재 Django 프로젝트의 환경 및 구성을 저장
    - **mysite/urls.py**: 현재 Django 프로젝트의 URL 선언을 저장

- 작업을 위한 프로젝트 환경을 설치하고 나서 앱(APP)을 생성해준다. 
> `python manage.py startapp app_name`















# 참고자료
- [HonKit](#https://tutorial.djangogirls.org/ko/django_start_project/)
- [docs.djangoproject](#https://docs.djangoproject.com/ko/3.1/intro/tutorial01/)