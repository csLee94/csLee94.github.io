---
title: vscode에서 가상환경 jupyter notebook 사용하기(with. venv)
author: csLee94
date: 2023-02-01 00:00:00 +0900
categories: [others]
tags: [jupyter notebook, python, visual studio code, venv]
---

> 저는 EDA를 진행할 때는 jupyter notebook을 선호합니다. 다만 브라우저에서 사용하는 것이 아닌, 다양한 편의성을 제공하는 vscode에서 사용하고 싶었습니다. 또한, 프로젝트별로 library를 별도로 관리하기 위해서 venv까지 함께 사용하는 법을 정리합니다! <br>
> 사용 환경
> - Mac, Win 10 *(이 포스트는 Mac 기준으로 작성합니다.)*
> - Python 3.8
> - VS code

<br>

## INIT 
---

vscode에서 준비된 프로젝트를 엽니다. 만약 Jupyter notebook 관련 vscode extension이 설치되어 있지 않다면, 아래 항목들을 설치해줍니다. 
- Jupyter
- Jupyter Keymap
- Jupyter Notebook Renderers

그리고 venv 환경을 생성하고 활성화합니다. 이 때 `.venv` 부분이 생성될 가상환경이 이름입니다.

```vim
$ python3 -m venv .venv
$ source .venv/bin/activate
$ (.venv) 
```

그리고 필요한 library와 `ipykernel`을 pip을 이용해 설치합니다.

```vim
$ (.venv) pip3 install pandas
$ (.venv) pip3 install ipykernel
```

마지막으로, 아래 명령어를 통해 jupyter notebook 커널 목록에 생성한 venv를 추가합니다. 이 때 `--name` 뒤에는 생성한 가상환경의 이름이, `--display-name` 뒤에는 표시할 이름을 쌍따옴표(")와 함께 작성합니다.

```vim
$ (.venv) python3 -m ipykernel install --user --name .venv --display-name ".venv"
Installed kernelspec .venv in /Users/user/Library/Jupyter/kernels/.venv
```

<br> 

## Connet kernel
---
위 작업들이 성공적으로 진행됐다면, VS code에서 jupyter와 연결할 차례입니다. 먼저 `ctrl+shift+p`를 누른 다음, 입력창에서 `Developer: Reload Window`를 입력해 vscode window를 한번 새로 고침 합니다.

그 다음, 우측 상단에 `Python {$version}`영역을 클릭하면, 연결할 수 있는 커널 목록이 나타납니다. 위 커맨드에서 작성했던 venv 이름과 경로를 참고해 클릭하면 연결됩니다.

![img](/assets/img/others/jupyter_0.png)
_적용 전_

그러면, 아까 `Python {$version}`였던 부분이 `.venv`로 변경되는 것을 확인할 수 있습니다. 그 다음 하나의 셀을 실행시켜 방금 venv에 설치한 pandas의 version을 확인해보겠습니다. 

![img](/assets/img/others/jupyter_1.png)
_적용 후_

잘 실행되네요!

<br>

## + Optional 
---
> 저는 보통 위 방식으로 진행하는데 가끔 에러가 발생하며, 보다 안정적인 방법이 있다고 합니다. 위 방법이 안된다면 아래 방식을 추가 진행해보세요.
{: .prompt-tip}

kernel 추가하는 방식이 아닌 Jupyter의 리모트 서버를 vscode로 연결하는 방법도 있습니다! venv에 jupyer를 추가로 설치해주세요. 그리고, **가상환경 디렉토리 폴더** *(eg./.venv/)*로 이동해 Jupyter 서버를 실행시킵니다.

```vim
$ (.venv) pip3 install jupyer
$ (.venv) cd ./.venv/
$ (.venv) ~/.venv/ jupyter 
```

그러면 jupyter server 주소가 나오면서 실행됩니다. vscode에서 `ctrl+shift+p`를 누르고, `Jupyter: Specify Jupyter Server for Connet`를 찾아 선택한 다음 Terminal에서 확인한 주소를 입력합니다. 

이 후 위와 동일하게 사진 내 영역을 클릭해 연결을 진행해주면 됩니다!