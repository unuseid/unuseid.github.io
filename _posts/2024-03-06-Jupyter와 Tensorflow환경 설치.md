---
title: "Jupyter와 Tensorflow환경 설치"
categories:
  - Dev
tags:
  - Tensorflow
  - Jupyter
  - AI
---

이번 시간에는 Ubuntu에 Conda와 Jupyter 를 활용한 원격 Tensorflow 테스트 환경을 구축 해보고자 합니다.

# Conda

다양한 버전의 Python이 필요하거나 진행하는 프로젝트에 따른 격리를 위해
Tensorflow 와 Jupyter 가 작동하는 환경을 conda 를 사용해 분리할 것이다.

## Conda 설치

아래 명령으로 miniconda 를 설치한다

```bash
# 가장 최신의 miniconda3 설치 스크립트 다운로드와 스크립트 실행.
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
&& sh Miniconda3-latest-Linux-x86_64.sh
```

설치 완료후 쉘에 재접속하면 conda 명령어가 동작한다.
설치중 문제가 생기거나 bash쉘 이외의 쉘을 사용한다면

```
~/miniconda/bin/conda init"
```

명령을 수동으로 수행.

## Conda env 생성

```bash
#jupyter lab을 위한 가상환경 생성.
$ conda create -n jupyter python=3.10
#tensorflow 를 위한 가상환경 생성.
$ conda create -n tensorflow python=3.10
```

Python3.10을 사용하는 가상환경이지만 둘은 분리된 환경으로 각각의 환경에 맞는 모듈을 별도 설치하여 사용한다.

conda env list 로 현재 가상 환경 리스트를 확인한다.

```bash
(base) sd@sd-HP-Laptop:~$ conda env list
# conda environments:
#
base                  *  /home/sd/miniconda3
jupyter                  /home/sd/miniconda3/envs/jupyter
tensorflow               /home/sd/miniconda3/envs/tensorflow
```

# Jupyter & Tensorflow

## env1에 Jupyter lab 설치

첫번째 conda 가상환경 jupyter를 활성화 시키고 jupyterlab 을 설치합니다.

```bash
conda activate jupyter
pip install jupyterlab
```

설치가 완료되면 "jupyter-lab" 명령으로 jupyter를 실행할 수 있습니다.

## env2에 Tensorflow 설치

첫번째 conda 가상환경 tensorflow 활성화 시키고 tensorflow  설치합니다.

```bash
conda activate tensorflow
pip install tensorflow
```

이때 Tensorflow와 함께 필요한 numpy, keras등의 모듈이 함께 설치됩니다.

## Jupyter lab에 커널추가

위에서 생성한 jupyter lab 에는 기본 커널만 생성되어 있습니다.
Tensorflow 를위해 생성한 Conda 환경에서 작업하려면 jupyter lab에 생성한 환경의 Kernel을 추가하여 사용합니다.

```bash
#추가 하고자 하는 conda env(tensorflow env) 에서 진행 합니다

# ipykernel 설치
pip install ipykernel

# JupyterLab 안에 Kernel 설치
python -m ipykernel install --user --name<가상환경 이름> --display-name<표시될 커널 이름>
```

Jupyter env 로 돌아와서 Kernel list 를 확인해 봅니다.

```bash
# Jupyter env 에서
(jupyter) sd@sd-HP-Laptop:~$ jupyter kernelspec list
Available kernels:
  python3       /home/sd/miniconda3/envs/jupyter/share/jupyter/kernels/python3
  tensorflow    /home/sd/.local/share/jupyter/kernels/tensorflow
```

반대로 Kernel 제거를 위해서는 아래 명령어를 이용 합니다.

```bash
jupyter kernelspec uninstall <가상환경 이름>
```

이제 기본적인 Jupyter lab 을 활용한 Tensorflow 테스트 환경이 구성 됐습니다.

## Jupyter lab 원격 접속

Jupyter가 참조할 설정파일을 생성합니다.

```
jupyter-lab --generate-config
```

위 명령어는 "~/.jupyter/" 위치에 "jupyter_lab_config.py" 파일을 생성합니다.

해당 파일을 열고 아래 내용을 넣어주세요.

```
c = get_config()  #noqa
c.JupyterApp.config_file_name = 'jupyter_lab_config.py'
c.NotebookApp.allow_origin = '*'
c.NotebookApp.ip = 'IP주소'
c.NotebookApp.open_browser = False

c.NotebookApp.password = 'passwd Hash 값'
c.NotebookApp.notebook_dir = 'Jupyter lab의 home dir'
```

c.NotebookApp.allow_origin
이 옵션은 JupyterLab 서버에 접속을 허용하는 IP를 지정하는 옵션입니다, 모든 곳에서 접속이 가능한 \* 로 설정하였습니다.

c.NotebookApp.ip
옵션은 본인 IP 주소를 적어주세요.

c.NotebookApp.open_browser
옵션은 브라우저를 자동으로 실행할지 여부 입니다.

c.NotebookApp.password
옵션은 서버에 접속하기 위해 사용할 암호를 설정합니다.
패스워드는 파이썬 쉘에서 다음 명령어로 생성합니다.

```
>>> from jupyter_server.auth import passwd
>>> passwd()
Enter password:  # 암호 입력
Verify password:  # 암호 확인
'password' # 암호 해시 값 출력
```

생성된 Hash를 복사하여 위의 옵션에 넣어주면 됩니다.
