---
title: githubリポジトリへのpushする際の設定の流れ
tags: Git GitHub Docker Linux SSH
author: t_kyn
slide: false
---
※2024/1/28 補足を追記しています。

## 背景
githubというソースコードを管理するバージョン管理システムがあります。
githubのfreeアカウントでプライベートリポジトリを作成し、ソースコードやdockerイメージをパッケージとして保管することを目的とします。

## 前提条件
githubへのpushは2つの方法で行っています。
* LinuxOSの場合、既存の仮想マシンにgitをインストールしています。
    * スペック：OS：CentOS7.6、CPU2コア、メモリ3GB、ディスク：20GB、
* Windowsの場合、VidualStudioCodeにより、gitプラグインをインストールしています。
    * スペック：OS：Windows10、CPU Inter i5 Core、メモリ8GB、ディスクHDD：300GB
## 実施内容

### githubのアカウント作成
* メールアドレスの登録が必要ですので、メールアドレスを登録します。
* パスワードを登録します。
* 「🚀 Your GitHub launch code」
というメールが届くので、内容に従いコード入力します。
https://docs.github.com/ja/get-started/quickstart/creating-an-account-on-github

### githubのプライベートリポジトリ作成
プライベートリポジトリを作成します。
githubのページで簡単に作れます。
パブリックリポジトリでも同様に作成できますが、インターネットへ公開されますので、
公開したくない場合は注意しましょう。

### gitのインストール
gitをインストールします。
```
yum install git
```

### gitを利用するための設定
* .ssh/config
* .ssh/github（鍵名はなんでもいい）
* .ssh/github.pub
（鍵名はなんでもいい。github側でも公開鍵の内容を登録しておく）
```
[root@homeserver ~]# ssh-keygen -t ed25519 -N "" -f ~/.ssh/github
Generating public/private ed25519 key pair.
/root/.ssh/github already exists.
Overwrite (y/n)? y
Your identification has been saved in /root/.ssh/github.
Your public key has been saved in /root/.ssh/github.pub.
The key fingerprint is:
SHA256:HOGEHOGEHOGEEEEEEEEEEEEEEEEEEEEEEEE root@homeserver
The key's randomart image is:
+--[ED25519 256]--+
|   hogehogehoge  |
|   hogehogehoge  |
|   hogehogehoge  |
|   hogehogehoge  |
|   hogehogehoge  |
|   hogehogehoge  |
|   hogehogehoge  |
|                 |
|                 |
+----[SHA256]-----+
[root@homeserver ~]# ls -lrt /root/.ssh/*

[root@homeserver ~]# ls -lrt ~/.ssh
合計 24
-rw-r--r--. 1 root root 1268 12月 23 22:25 known_hosts
-rw-r--r--. 1 root root   95 12月 23 22:27 github.pub
-rw-------. 1 root root  399 12月 23 22:27 github
-rw-r--r--. 1 root root  395 12月 23 22:42 id_rsa.pub
-rw-------. 1 root root 1679 12月 23 22:42 id_rsa
-rw-------. 1 root root   80 12月 23 23:09 config
[root@homeserver ~]# 


[root@homeserver ~]# ssh-add -l
2048 SHA256:HOGEHOGEHOGEEEEEEEEEEEEEEEEEEEEEEEE root@homeserver
[root@homeserver ~]# ssh-add ^/.~/.ssh/github
Identity added: /root/.ssh/github (root@homeserver)
[root@homeserver ~]# 
[root@homeserver ~]# ssh-add ~/.ssh/github
2048 SHA256:HOGEHOGEHOGEEEEEEEEEEEEEEEEEEEEEEEE root@homeserver
[root@homeserver ~]# 

[root@homeserver ~]# cat .ssh/config 
Host github
  User git
  Hostname github.com
  Port 22
  IdentityFile /root/.ssh/github
[root@homeserver ~]# 
※キーの中身は秘密とさせて頂きます。
```

これらの設定が完了したらgithub.comにsshしてみます。
```
[root@homeserver ~]# ssh -T git@github.com
Hi ●●●! You've successfully authenticated, but GitHub does not provide shell access.
※●●●はgithubのユーザになります。
```

githubは、gitユーザでないとsshできないので、gitユーザ指定にしています。

SSH接続でsuccessfullyが出力されていれば、pushやpullができるようになります。

### dockerのパッケージのpush
自作のdockerイメージファイルをクラウド上に保管したい場合があると思います。
その場合、ghcr.ioパッケージとしてアップロードすることが可能です。

* ghcr.ioのアクセストークンを作成
https://github.com/settings/tokens
よりアクセストークンを作成します。パスワードを求められたらパスワードを入力し、作成します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/551020/dd9a0642-9f3b-408f-2f35-7bcb37e74220.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/551020/cdb7a7c5-a151-4313-294f-51e344e29922.png)

* docker tagをつけた上で、docker pushが可能
```
docker tag 9bbd6f10291f ghcr.io/●●●/docker_homeserver_infrastructures/t_kyn_python:latest
[root@homeserver ~]# docker images
REPOSITORY                                                           TAG          IMAGE ID       CREATED        SIZE
ghcr.io/t-kyn-git/docker_homeserver_infrastructures/t_kyn_python       latest       9bbd6f10291f   6 months ago   468MB
t_kyn_python                                                         latest       9bbd6f10291f   6 months ago   468MB
[root@homeserver ~]# docker push ghcr.io/t-kyn-git/docker_homeserver_infrastructures/t_kyn_python:latest
The push refers to repository [ghcr.io/t-kyn-git/docker_homeserver_infrastructures/t_kyn_python]

3c1057617da6: Preparing 
be1357297e43: Preparing 
b2f9b7786585: Preparing 
9d8b0c05e1f1: Preparing 
b78f89681ec5: Preparing 
e70a7d356758: Preparing e70a7d356758: 
～～～略～～～
[2023-12-23 18:10:55.578] [root@homeserver ~]# 

```

## 補足：パブリックリポジトリの履歴削除
Githubを運用していく中で、誤操作をした場合に有効化と思いますが、
（プライベートリポジトリを利用するのが最善ですが、誤ってパブリックリポジトリにコミットしたくないファイルを含めてしまうと履歴削除はできません。
履歴にこだわらないならば、履歴自体をバックアップし、
履歴上書きコミット＆強制プッシュ（-fオプション指定）すれば消してくれます。

```
git clone <URL>
git checkout --orphan tmp
git commit -m "Initial Commit"
git checkout -B master
git branch -d tmp
git push origin master -f
```

参考：

https://qiita.com/sea_mountain/items/d70216a5bc16a88ed932

https://zenn.dev/giba/articles/7eb5e4c4446769

## あとがき
※セキュリティの都合上、一部をマスキング（内容を書き換えて記載）しておりますのでご了承ください。

