---
published: false
---

Docker: Getting Started With CentOS
==============================

서버 OS로 CentOS 6.5를 쓸 수 있게 되면서 최근 유행하고 있는 [Docker][docker.io]를 적용해보기로 했다.

Docker에 대해서는 개념적으로 알고는 있었지만 실제 사용해보는 것은 처음이었기 때문에, 사용법 위주로 간단히 기록을 남기려고 한다.


Installation
-----------------

#### 1. Setup EPEL Repository

우리가 설치할 docker-io 패키지는 EPEL에 포함되어 있으므로 아래 링크를 참조하여 설치해준다.

- http://www.rackspace.com/knowledge_center/article/installing-rhel-epel-repo-on-centos-5x-or-6x
```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
sudo rpm -Uvh epel-release-6*.rpm
```

#### 2. Install Docker Package

docker-io 패키지를 설치한다.
```
sudo yum install docker-io
```

#### 3. Setup Docker (optional)

docker-io 패키지 설치가 끝나면 바로 실행할 수 있다. 하지만 어떤 이유로 실행 옵션 등을 변경해야 한다면 약간의 작업을 해주어야 한다.

##### 3.1. Docker 작업 디렉토리를 변경하고 싶은 경우

어떤 서버 환경에서는 root 파티션의 크기를 작게 유지하면서 사용자 디렉토리에 큰 파티션을 할당하여 사용하도록 한다. 이 경우에는 Docker가 사용하는 기본 작업 디렉토리를 변경하여 root 파티션을 가득 차는 것을 방지해야 한다.

- Docker의 기본 작업 디렉토리는 `/var/lib/docker` 이다.

- 작업 디렉토리를 변경하는 방법은 여러 가지가 있지만 -g (--graph) 실행 옵션을 이용하는 방법이 가장 확실하다.
  - bind mount를 이용하거나 symbolic link를 사용할 수도 있지만 추천하지 않는다.

- `/etc/sysconfig/docker` 파일을 수정하여 원하는 디렉토리로 지정한다.
```
$ sudo vi /etc/sysconfig/docker
# /etc/sysconfig/docker
#
# Other arguments to pass to the docker daemon process
# These will be parsed by the sysv initscript and appended
# to the arguments list passed to docker -d

other_args="-g /<your_directory>/docker"
```

- Docker 대몬을 재시작한다.
```
sudo /sbin/service docker restart
```

Prepare Images
----------------------

Docker는 이미지(image)라는 단위로 컨테이너(container=docker process) 환경을 관리한다.
다시 말해 Docker가 실행시킨 프로세스는 특정 이미지 위에서 동작하며 각각 독립된 환경을 지니게 된다.

> 내가 처음 Docker를 접했을 때 가장 감명을 받은 부분이기도 한데, 이 특징 하나만으로도 Docker를 도입할 가치가 충분하다고 생각한다. 기본 설치된 호스트 OS를 전혀 건드리지 않고 Docker 이미지를 이용하여 각종 프로그램들을 설치하고 호스트에서 실행, 필요가 없어지면 깨끗하게 삭제도 가능하다. 심지어 CentOS에서 Ubuntu 이미지를 사용해 프로세스를 동작시킬 수도 있다.

Docker 이미지는 소위 말하는 `상속` 구조를 지원한다. 즉, 이미지를 매번 바닥부터 만들 필요 없이 특정 상태의 이미지를 기반으로 새로운 이미지를 생성할 수 있다.

만약 ubuntu:14.04 이미지를 기준으로 새로운 이미지를 생성한다면 다음과 같은 모습이 될 것이다.

```
└─511136ea3c5a Virtual Size: 0 B
 ...
  └─c4ff7513909d Virtual Size: 213 MB Tags: ubuntu:14.04 // 공식 Docker 이미지
    └─62e7b55d31d3 Virtual Size: 213 MB // 중간 중간 파일 시스템에 변경이 발생하면 commit 된다.
     ...
      └─c235471015e1 Virtual Size: 677.8 MB Tags: piwik:latest // 이미지 변경이 완료되면 tag가 붙는다.
```

[docker.io]: http://docker.io
