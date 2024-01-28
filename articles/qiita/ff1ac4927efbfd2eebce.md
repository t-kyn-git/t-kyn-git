---
title: ovfファイルの作成方法
tags: ovftool vmware バックアップ Linux コマンドプロンプト
author: t_kyn
slide: false
---
### 目的
システム障害では、障害対策として
・サーバのバックアップやスナップショットを取得する
・データベースをバックアップし、NASやS3などの外部のストレージにて保管する
・バックアップ運用がない場合、高可用性構成（High Availability）構成とし、ミッションクリティカルでもサービスを継続する
といった運用が必要となります。

一方、WindowsPCで1 台の仮想マシンを実行するユーティリティを利用しており、
自作の仮想マシンで何らかの不具合が発生した場合、
基本的には何も実装していないため、バックアップ運用を行う仕組みを構築する必要があります。

仕組みを検討した結果、VMWare Workstation Playerインストール時にovftoolを装備しているため、
ovftoolを用いてサーバテンプレートとしてエクスポートすることを目的としています。

### 前提
自作サーバの要件としては下記になります。
* ホストOS：Windows10 CPU、メモリ：8GB
* VMWare Workstation Player 17がインストール済
* <サーバ名>.vmxファイルがあり、vmdkという拡張子の他、サーバを稼働するために必要なファイルがある。
* 仮想サーバのファイルサイズは数10GB

### 実施内容

OVFファイルをエクスポートするコマンドが用意されています。
コマンドプロンプトを用いて、ovftool.exeを使用します。
ovftool.exe --noImageFiles [vmxファイルフルパス] [ovfファイルフルパス]
* イメージでマウントされているisoファイルを入れたくない場合、noImageFilesオプションを指定。
* vmxファイルフルパス：ovfファイルを取得したい仮想マシンのvmxファイルのフルパスを指定
* ovfファイルフルパス：作成したovfファイルのフルパスを指定します。事前にフォルダを作成する必要はありません。

```
※サーバ名や、パスに関しては、秘匿としたいので、仮名にしています。
C:\Users\USER>"C:\Program Files (x86)\VMware\VMware Player\OVFTool\ovftool.exe" --noImageFiles "F:\フニャフニャ～\homeserver_test\homeserver_test.vmx" "F:\フニャフニャ～\homeserver_test_ovf\homeserver_test.ovf"
Opening VMX source: F:\フニャフニャ～\homeserver_test\homeserver_test.vmx
Opening OVF target: F:\フニャフニャ～\homeserver_test_ovf\homeserver_test.ovf
Writing OVF package: F:\フニャフニャ～\homeserver_test_ovf\homeserver_test.ovf
Disk progress: 1%

```
上記実行完了するとDisk progressが100%となりますので、
該当のファイルがあるか確認してみてください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/551020/64b41337-29fe-f4ea-3aa0-6766ceffa49d.png)

### ovfファイルからサーバを起動する
ovfファイルを使用して仮想サーバを作成することが可能です。
注意点としては、コピー前の設定と変わらないため、
IPアドレスが固定になっているかつ、新しいサーバとして作成する場合、
変更が必要となります。

### あとがき
ovfファイル自体は容易に作成ができますが、
Windowsバッチファイルとして、仮想マシンが停止していたらovftoolを自動的に実行する仕組みがあればうれしいですね…
（停止するときのトリガーを検知する仕組みが難しいですがもう少し調べてみようと思います★）
```
@echo off
set VMX_FILE=%1
set OVF_OUTPUT_PATH=%2
set OVF_TOOL_PATH="C:\Program Files (x86)\VMware\VMware Player\OVFTool\ovftool.exe"

REM Check if ovftool is installed
if not exist %OVF_TOOL_PATH% (
    echo Error: VMware OVF Tool is not installed.
    exit /b 1
)

REM Check if the virtual machine is powered off
%OVF_TOOL_PATH% "vi://%VMX_FILE%" > nul 2>&1
if %errorlevel% equ 0 (
    echo Virtual machine is powered off. Creating OVF file...
    %OVF_TOOL_PATH% --noImageFiles %VMX_FILE% %OVF_OUTPUT_PATH%
) else (
    echo The virtual machine is running. ほげほげお!
    exit /b 1
)

```
```
create_ovf.bat vmx ovf
```

