---
layout: post
title:  "서버리스 개념과 Serverless Framework"
subtitle:   서버리스 개념과 Serverless Framework
categories: datascience
tags: dev serverless server aws lambda
comments: true
---

# 목차
1. [서버리스(Serverless)의 개념](#serverless의-개념)
2. [AWS Lambda](#aws-lambda)
3. [Serverless Framework](#serverless-framework) <br>
    - [Serverless Framework Basic Components](#serverless-framework-basic-components) 
4. [Reference](#reference)

---
<br>

# Serverless의 개념

- **Serverless는 백엔드에 서버가 없다?** [ X ]
    > **Serverless**는 백엔드인데, 직접 서버를 관리하지 않는 경우를 뜻함 <br>
*Backend without Server management* *by. [Nomad Coders](https://www.youtube.com/channel/UCUpJs89fSBXNolQGOYKn0YQ)*


- Serverless의 탄생 배경
    <br>
    1. 기존에는 서버의 하드웨어와 소프트웨어를 둘 다 관리 (물리적 서버 및 메모리 관리 등)
    2. 아마존의 EC2 서비스를 시작으로 서버를 '설치'하는 대신 아마존에 돈을 지불하고 서버를  대여.<br> 아마존과 같은 업체들이 돈을 받고, 서버의 하드웨어 부분을 관리
    3. 그러나 여전히, 업데이트/보안/데이터 백업 등 소프트웨어 관리가 필요
    4. **<mark style='background-color: #ffdce0'>서버리스의 등장</mark>**<br>
    서버리스는 백엔드를 서버에 올리는 것이 아니라, **백엔드를 작은 함수 단위로 쪼개서 직접  관리하지 않는 서버로 업로드**(eg, AWS Lambda)

<br>

- Serverless의 특징
    > 서버리스가 아닌 경우, 서버는 24식나 돌아가며 항상 요청에 응답할 준비 대기<br> **서버리스의 경우** 내가 업로드한 함수들은 잠들어 있고, request가 오는 순간 AWS가 함수를 깨워 작동. 수행을 마치고 다시 Sleep
    - 장점
    1. 저렴한 사용 가격 (항시 서버 운영이 아닌 Request에 반응해 작동)
    2. 스케일 조정이 용이 (유저에 따라 AWS에서 함수를 복제해 작동)
    3. 빠른 제품 출시 가능, 물적 자원 절약 가능
    4. 사이드프로젝트 & 프로토타입 등 서버관리에 시간을 아끼고 싶을 때 유용


    - 단점
    1. Request에 반응해 깨어나는 함수이기 때문에 아주 약간의 지연 시간이 발생
    2. AWS와 같은 서버 제공자에게 의존도가 높아진다.
        > 서버리스로 작업하면 어플리케이션의 구조자체가 바뀌게 되어, 다른 서버로 migration하기 어렵다.
    3. 서버에 대한 통제를 잃고, 구조를 변경해줘야함

<br>

- 학습 시 참고 사이트
    - SERVERLESS.COM
    - AWS Lambda
    - Google Cloud Functions
    - Apex
    - Terraform

> 위 글은 유튜버 <mark style='background-color: #fff5b1'>노마드 코더 Nomad Coders</mark>의 ['서버러시는 서버가 없는걸까? 8분 개념 설명](https://www.youtube.com/watch?v=ufLmReluPww&t=448s) 영상을 정리했습니다.

<br><br>

# AWS Lambda
> ***Lambda**는 서버를 프로비저닝하거나 관리하지 않고도, 코드를 실행할 수 있는 컴퓨팅 서비스입니다. Lambda는 고가용성 컴퓨팅 인프라에서 코드를 실행하고 서버 및 운영 체제 유지 관리/자동확장/코드 모니터링 및 로깅을 비롯한 모든 컴퓨팅 리소스 관리를 수행합니다.<br> Lambda API를 사용하여 Lambda 함수를 호출하거나 다른 AWS 서비스의 이벤트에 응답하여 Lambda 함수를 실행할 수 있습니다. <br>by. [Amazon Guide](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/welcome.html)* 

- 사용가능 언어<br>
 **Python**/ Node.js / Ruby / Java / Go / C# 

<br>

- AWS Lambda Process Summary<br>
    1. Lambda Function 정의<br>
        - 수행할 함수를 정의한다. 
        - Source Code를 작성할 때, 필요한 Library를 함께 Lambda Environment에 업로드해줘야한다.
            > **~에서 업로드** 버튼에 zip 파일 업로드할 수 있다.
    2. Lambda Function's Trigger 설정
        - **트리거 추가**버튼을 통해 Lambda Function을 실행시킬 트리거를 설정한다.
        - 트리거를 **API 게이트웨이**로 설정함으로써 외부 서비스에서 Lambda Function을 실행시킬 호출가능한 API Endpoint를 설정한다.
    3. API Gateway Console에서 생성한 API를 Lambda Function과 통합해주고, HTTP Method를 설정 
        >**POST** Method의 경우 '통합 요청'에서 '매핑 템플릿'설정을 통해 'application/json' 이름의 '메서드 요청 패스쓰루' 템플릿을 지정해준다.


<br><br>

# Serverless Framework
- Serverless Framework는 AWS에서 제공하는 AWS Lambda에서 애플리케이션을 구축하기 위해 개발된 첫 번째 Framework다. AWS Lambda, API Gateway와 같은 Serverless 자원들을 사용하여 Serverless 아키택처를 쉽게 설계, 구현, 배포, 관리할 수 있게 도와준다. 현재 AWS, Google Cloud Platform, Azure, Knative 등 다양한 Cloud 제공 업체에서 지원한다.

- 기본 언어는 Node.js이나 이 외에도 swift, ruby, Go, python, PHP 언어를 지원한다. 또한 활성화되어 있는 Framework 중 하나이기 때문에 관련 Plugin이 굉장히 많다. 이 Plugin을 활용해 HTTP, Tracing, Step Functions 등의 다양한 기능을 활용할 수 있다.

    > - 기본 언어가 Node.js이기 때문에 Serverless Framework를 python으로 사용하기 위해서는 많은 환경설정이 필요하다고 한다.
    > - 특히 사용되는 Library 관련해 Packing하는 작업 관련해서 어려움이 많다고 한다. ~~이를 해결하기 위해 **serverless-python-requirement**라는 Plugin이 나왔다고 하는데 여전히 나에게는 어렵다.~~
    > - [serverless-python-requirement plugins Rerference](https://serverless.com/blog/serverless-python-packaging/)

- **Faas의 단점 보완**: 수정 및 배포 과정 축약<br>
    Faas의 가장 큰 단점은, **코드 수정 및 배포**가 어렵다는 것이다. 코드 한 줄을 수정하더라도 다시 첨부해 업로드해야하고, 코드의 크기가 일정 크기를 초과할 경우 AWS Console에서 수정할 수 없는 상황이 초래되기도 한다. <br>
    Serverless Framework는 CLI 환경에서 간단한 설정 후 명령어 하나로 쉽게 배포할 수 있어, Faas의 배포 문제를 완화시켜 줄 수 있다.

    > Faas(Function-as-a-Service)는 서버리스 컴퓨팅을 구현하는 방식으로 AWS Lambda, Google Cloud Functions, Microsoft Azure Functions 등이 있다.

<br>

## Serverless Framework 다루기
1. 사전준비 <br>
    - IAM 계정 준비 <br>
    AWS Console 외부에서 자유롭게 서버를 배포하고 생성할 수 있도록 IAM계정을 생성한다.
        > 이때 생성되는 액세스 키 ID에 대한 키는 **절대** 공유되어선 안된다. 과금에 대한 부분도 건드릴 수 있기 때문에 Github이나, 블로그에 배포하지 않도록 유념한다.

        IAM 계정이 준비되었다면, 로컬에 배포할 수 있도록 serverless framework 환경 설정을 해준다. (Configure the **default** profile)
        ```vim
        serverless config credentials --provider aws --key 액세스키ID --secret 비밀액세스키
        ```


    - 가상환경 준비 <br>
    AWS Lambda에 업로드할 때 zip 파일을 좀 더 쉽게 생성하기 위해서 가상환경 사용한다. 나중에 'python-requirements' Plugin과 함께 사용 <br>
    여기서는 **pipenv**를 사용한다.
        > pipenv의 대표적인 기능
        > - pip과 virtualenv의 기능 동시 사용
        > - requirements.txt 대신 Pipfile, Pipfile.lock 사용
        > - 해쉬의 자동 생성
        > - .env 자동 인식
    
2. 설치 <br>
    - Serverless Framework를 설치하기 위해서 NPM을 먼저 설치해야한다. [Node JS 사이트](https://nodejs.org/en/download/)에 접속해 자신의 운영 체제에 맞는 버전을 선택해 설치한다. 대부분 `LTS` 버전을 선택한다.
    - Node 및 NPM이 설치됐다면, 아래 명령어로 Serverless Framework를 설치한다.
        ```vim
        npm install -g serverless
        ```
3. 로컬에서 서비스 만들기
    ```vim
    serverless create --template aws-python3 --name ProjectName --path ProjectPath
    ```
    - `--template`은 serverless에서 사용할 수 있는 template이다.
    - `--name`은 serverless.yml 파일 속 서비스 이름이다.
    - `--path`는 서비스가 생성되어야하는 path다. 

    위 명령어를 실행시키면 해당 경로 아래 `--path`에 설정한 이름을 가진 폴더가 생성된다. 해당 폴더 아래 **handler.py**와 **serverless.yml**파일이 생성됐다. **serverless.yml** 파일을 열어보면 service에 `--name`에 설정한대로 설정되었음을 알 수 있다.
    ```yaml
    service: ProjectName
    ```

    > 이후 해당 폴더에서 **pipenv**를 통해 가상환경을 활성화하고 Pipefile.lock파일도 만들어주기 위해 명령어를 한번 더 친다.
    > ```vim
    > pipenv shell
    > pipenv lock
    > ```

    1. **handler.py** <br>
    handler는 이벤트를 처리하는 Lambda 함수의 메서드다. 이 안에 정의하는 함수대로 Lambda가 생성된다.python에서 AWS Lamb Function Handler를 정의하는 기본 구조는 다음과 같다.
        ```python
        def handler_name(event, context):
            ...
            return value
        ```
        이 때 handler를 실행시키기 위해 별도 Library가 필요하다면 설치해줘야한다. AWS Lambda 환경에 handler.py와 함께 zip파일로 올려주기 위해 가상환경에 설치한다.
        ```vim
        pipenv install (library)
        ```
    
    2. **serverless.yml 파일 수정** <br>
    serverless.yml은 서비스 설정 값들이 관리되고 있는 파일이다. 
        > serverless.yml의 역할
        > - 서버리스 서비스 선언
        > - 서비스에 들어갈 한 개 이상의 함수를 정의
        > - Provider 정의 (AWS, GCP, Azure 등)
        > - Plugin 정의
        > - 함수들을 실행할 이벤트들을 정의
        > - 함수에서 사용되는 리소스 정의
        > - 이벤트 섹션에 나열된 이벤트들이 개발 시 이벤트들이 요구하는 리소스들을 자동적으로   생성하도록 허용
        > - 서버리스의 변수들을 사용해서 유연한 설정값을 만들도록 허용
    
        <br>

        serverless.yml은 사용되지 않는 기능들의 예시를 주석으로 채워놨다. 주석을 제거한 가장 기본 형태는 다음과 같다.
        
        ```yaml
        service: ServiceName

        provider:
            name: aws
            runtime: python3.8
        
        functions:
            hello:
              handler: handler.hello
        ```

        ### Serverless Framework Basic Components
        - service <br>
            service는 Lambda에서 표시할 Prefix(접두사)이다. 실제로 서버리스가 배포되면 service의 이름이 앞에 붙은 이름으로 배포된다.
        - provider <br>
            name: 서버를 제공하는 곳<br>
            runtime: 언어<br>
            region: 배포하고자 하는 지역 (서울은 ap-northeast-2)<br>
            stage: API Gateway의 지점 (API endpoint URL에 반영)
        - functions <br>
            hello: function의 경우 첫 부분에 함수 이름 <br>
            handler: `handler.` 뒤에 `handler.py`에서 정의한 함수의 이름<br>
            event: Lambda에서 트리거가 될 event 정의
            > 작성 시 참고 사항<br>
            > - function 속성은 provider에서 상속받은 속성을 덮어쓴다.
            > - events는 handler의 바로 밑에 위치해야한다.
            > - 코드의 인덴트(들여쓰기)가 망가지지 않도록 주의하자.(yml 파일 특성)
        
        <br>

        ```yaml
        service: ServiceName

        provider:
            name: aws
            runtime: python3.8
            region: ap-northeast-2 
            stage: dev 

        functions:
            hello:
              handler: handler.hello
              events:
                - http:
                    path: hello
                    method: get
        ```

        `ap-northeast-2`(서울)로 지역 설정하고, `dev` 스테이지로 배포한다. 또한 `hello`라는 함수는 `handler.hello`와 연결되어 잇고, `http`의 `GET`방식을 통해 실행할 수 있다.

        <br>

    3. **deploy**
        Deploy 하기 전 serverless-python-requirements plugins를 추가해야한다. 이를 위해서 패키지에 대한 정보와 버전에 대한 정보가 있는 package.json 파일을 만들어야한다. 생성한 프로젝트 경로에서 아래 코드를 입력한다.
        ```vim
        sls plugin install -n serverless-python-requirements        
        ```
        serverless framework에서 library의 정보를 읽을 수 있도록 `pipenv`에 설치한 목록을 **requirements.txt**로 내보낸다.
        ```vim
        pipenv lock -r > requirements.txt
        ```

        이후 plugin을 사용하기 위해서 serverless.yml에 해당 코드를 추가한다.
        ```yaml
        plugins:
            - serverless-python-requirements
        
        custom:
            pythonRequirements:
                dockerizePip: non-linux
        ```
        > **dockerizePip**는 aws lambda가 돌아가는 OS환경인 Linux 환경에서 컴파일하기 위한 옵션이다. 사용자의 OS가 Linux여서 굳이 docker를 사용하지 않아도 된다면, `pythonRequirements`이하를 지우면 된다.

        이후 다음 명령어를 통해 배포한다.
        ```vim
        serverless deploy
        ```

        - serverless deploy를 했을 때 동작 process
            1. serverless.yml 설정 파일로부터 AWS CloudFormation 템플릿 파일 생성
            2. Lambda function으로 실행될 코드들을 Zip 파일로 압축
            3. 이전 배포된 모든 파일에 대한 hash를 가져온 뒤 현재 로컬에 있는 파일들의 hash와 비교
            4. 만약 hash결과가 같으면 배포 프로세스는 종료
            5. hash 결과가 같지 않으면, zip파일을 s3 bucket에 업로드
            6. 모든 IAM Roles, Lambda Function, Events 및 그 외 자원들이 AWS CloudFormation 템플릿에 추가
            7. 새로운 CloudFormation 템플릿으로 Stack을 업데이트
            8. 각각의 배포는 각 Lambda function을 새로운 버전으로 발행





<br><br>
# Reference
> -  [노마드 코더 Nomad Coders](https://www.youtube.com/channel/UCUpJs89fSBXNolQGOYKn0YQ) 유튜브: [서버리스 기초개념](https://www.youtube.com/watch?v=ufLmReluPww&t=448s)
> - [Gyullbb님의 블로그](https://velog.io/@_gyullbb/Serverless-Framework-VS-Chalice-4) : Serverless 개념과 Python에서 serverless framework 다루기
> - [Neon K.I.D님의 블로그](https://blog.neonkid.xyz/140): Serverless Framework를 사용하여 더 쉽게 서버 배포하기
> - [velopert님의 블로그](https://velopert.com/3549): Serverless 프레임워크로 서버리스 애플리케이션 생성 및 배포하기
> - [hello-bryan님의 블로그](https://hello-bryan.tistory.com/95): NPM 설치하기
> - [chullino님의 블로그](https://medium.com/@chullino/serverless-python-requirements-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0-8c93fdf43c9a): python-requirement 활용하기
> - [changhoi님의블로그](https://changhoi.github.io/posts/serverless/serverless-framework-quicklearn-(1)/): serverless.yml 기초개념
> - [drgabrielharris님의 블로그](https://drgabrielharris.medium.com/python-how-create-requirements-txt-using-pipenv-2c22bbb533af): pipenv 사용법