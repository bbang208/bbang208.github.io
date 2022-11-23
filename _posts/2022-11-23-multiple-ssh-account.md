---
layout: post
title: SSH를 여러 Git 계정에 사용하는 방법
subtitle: config 관리로 쉽게 !
gh-badge: [follow]
comments: true
tags: [git, ssh, 여러 계정, 깃]
---

개발을 하다 보면 여러 이메일을 갖고 Git 플랫폼을 이용할 때가 있다. 이런 경우 같은 ssh를 사용할 수 없기 때문에, 다른 ssh를 사용하여 Git을 이용하여야 한다.

이 때 사용 가능한 방법에 대해 글을 작성하려 한다.

<br>

## 추가 ssh 생성

새롭게 사용하기 위한 ssh를 만들어준다. ssh폴더 경로로 이동한 뒤, 터미널에 아래와 같은 명령어를 입력한다.

```shell
$ ssh-keygen -t ed25519 -C "your_email@example.com"
```

그럼 아래와 같은 문구가 뜰 탠데,

```shell
Generating public/private ed25519 key pair.

Enter file in which to save the key (/Users/junseo/.ssh/id_ed25519): bbang-new
```

여기서 파일의 이름을 정한다. 그냥 enter 시에는 id_ed25519로 된다. 나는 중복방지와 쉬운 구분을 위해 bbang-new로 생성하였다.

이름 정하고 enter 누른 뒤 자신이 사용하고자 하는 비밀번호 입력 및 확인입력 까지 한다.

`ls` 커맨드를 사용하여 제대로 생성됐는지 확인할 수 있다.

<br>

## 새로운 ssh 등록

아래 명령어를 입력하여 새로 만든 ssh의 공개키를 복사한다.

```shell
$ pbcopy < ~/.ssh/bbang-new.pub
```

그 다음 각 서비스의 SSH Key 등록 창에서 붙여넣기 하여 key를 등록한다.

Github은 프로필 > Settings > SSH and GPG Keys에서 등록할 수 있다.

<br>

## 계정에 따라 Git에서 구분하여 사용하려면...

ssh추가가 되었다면 config파일을 하나 생성하여 계정에 따라 ssh key를 사용할 수 있도록 구분해 주어야 한다.

config파일을 만들어서 아래와 같이 작성해준다.

만약.. 다른 여러개의 ssh key를 Github에서 사용하기 위한다면, Account1과 Account3을 참고하여 등록한다.

```
# Account1
Host github.com
	hostName github.com
	User git
	IdentityFile ~/.ssh/id_ed25519

# Account2
Host bitbucket.com
	hostName bitbucket.com
	User git
	IdentityFile ~/.ssh/bbang-new
	
# Account3
Host github.com-new
	hostName github.com
	User git
	IdentityFile ~/.ssh/bbang-new-github
```

hostName은 동일하게 github.com이지만 Host가 다른 것을 볼 수 있다.

그럼 각각의 key로 다른 계정을 사용하여 Clone하려 할 때 아래와 같이 사용할 수 있겠다.



1번 Account를 사용

```shell
$ git clone git@github.com:android/sunflower.git
```

3번 Account 를 사용

```shell
$ git clone git@github.com-new:android/sunflower.git
```

 hostName은 같고 Host만 다른 것을 확인할 수 있다.



해피-코딩 !
