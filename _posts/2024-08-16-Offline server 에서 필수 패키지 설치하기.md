---
title: "Offline server 에서 필수 패키지 설치하기"
categories:
  - Dev
tags:
  -
---

On-premise 환경에서 일하다 보면 인터넷에 접속할 수 없는 서버에서 작업할때가 있다. 인프라를 제공해준 측에서 솔루션에 필요한 Dependency들을 설치한 상태로 제공해 주면 좋겠지만 항상 그렇지만은 않은데. 이럴경우 직접 빈 서버에 필수 패키지들을 설치해야 하는데.
이럴경우 사용 할 수 있는 방법에 대해 알아보겠습니다.

각 필요 패키지마다 RPM파일을 하나하나 가져가는 방법도 있지만. RPM에 Dependency가 많을경우 그리고 각 RPM이 동일 패키지 다른 버전에 대해 Dependency가 있는경우 등이 발생하면 머리가 아파지기 시작합니다. 그렇기 때문에 Local disk 에 repository를 생성하고 해당 repository에서 필요 패키지를 받아 설치하는 방법을 사용하겠습니다.

오프라인 타깃 서버와 같은 아키텍쳐를 가진 온라인 서버를 한대 준비합니다. Docker등을 이용해도 괜찮습니다.

> 아래 예시는 RHEL8-arch64을 기준으로 작성 했습니다. \*

### 필요 파일 준비

아래 1~3 절차는 온라인접근 가능 서버에서 진행합니다.

1. 온라인 서버에서 createrepo 패키지 rpm 파일을 download

```shell
$ yum install -y --downloadonly --downloaddir=/root/rpm-list/ createrepo
```

오프라인 타겟 서버에서 local dir 을 repository로 만들기 위한 패키지 rpm files.

2. yumdownloader install

```shell
$ yum install yum-utils
```

yumdownloader 를 통해 repo 에서 rpm 파일을 다운로드 하기 위한 패키지.

3. 오프라인 서버 필요 패키지 다운로드

```shell
$ yumdownloader --downloadonly --resolve --installroot={currentAbsolutePath} -downloaddir=~/{packagename} {packagename}
```

오프라인 타겟 서버에서 yum install 할 package RPM file with dependency.
각 패키지 별로 폴더에 다운로드.

--downloadonly : rpm File download
--resolve : with dependency
-- installroot : without local cache (이 옵션이 없을 경우, 로컬에 이미 설치된 dependency rpm은 다운로드 하지 않음. )

### 준비된 파일 오프라인 타깃 서버에서 설치.

`필요파일 준비` 에서 download 한 파일을 offline target server로 보내줍니다.
여기서부터는 오프라인 타겟서버에서 진행한다.

1. install all rpm in createRepo
   필요파일준비-1 에서 download 한 createrepo 패키지 파일 install

```shell
$ yum install drpm-0.4.1-3.el8.x86_64.rpm
$ yum install createrepo_c-libs-0.17.7-5.el8.x86_64.rpm
$ yum install createrepo_c-0.17.7-5.el8.x86_64.rpm
```

> RHEL의 경우 yum 명령어를 사용 하기 위해서 등록 과정을 거쳐야 하고 아래 명령어로 등록 가능합니다(어차피 실행되지 않겠지만). 그렇지 않을 경우 경고 문구가 출력 될 수 있습니다. 그러나 오프라인 설치 명령어는 유효하게 작동. \*

```
#subscription-manager register --username <username> --password <password> --auto-attach 명령어 사용(rhel 서브스크립 등록된 계정)
```

2. make folder to local repo
   local repo가 될 폴더({targetDir})를 생성, local repo 초기화.

```shell
$ mkdir {targetDir}
$ createrepo {targetDir}
```

3. move built-in repos
   기본 repository 제거(및 백업)

```shell
$ mkdir ~/yum.repos.d_backup
$ mv /etc/yum.repos.d/* ~/yum.repos.d_backup
```

4. create local repo files
   local dir 을 가리키는 repo file 생성

```shell
$ vi /etc/yum.repos.d/local-repo.repo
```

```
[local-repo]
name=local-repo
baseurl=file:///repodisk
gpgcheck=0
enabled=1
priority=1
```

5. move all package files into {targetDir}
   필요파일준비-3 에서 download 한 package

```shell
$ yes | cp -rf {python39 packageDir} {targetDir}
$ yes | cp -rf {wget packageDir} {targetDir}
```

6. update localrepo
   새 package 파일이 들어온 repository meta 업데이트.

```shell
$ createrepo --update {targetDir}
```

7. clean yum cache and update
   yum 의 repository cache 정보 제거후 package list 다시 받아옴.

```shell
$ yum clean all
$ yum update
```

8. install Packages.
   local repository 로부터 yum 을 이용한 package install

```shell
$ yum install {packageName}
```
