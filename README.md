# AWS Cloud Engineer Portfolio

## About

TerraformとCloudFormationを利用してAWS環境の構築・検証を行った成果物をまとめています。

## 保有資格

- CCNA
- AWS Solutions Architect Associate

## Skills

- AWS
- Terraform
- CloudFormation
- VPC
- ALB
- Auto Scaling Group
- SSM Endpoint
- S3 Gateway Endpoint

# 概要

TerraformによるAWS 2AZ Web環境構築

## 構成概要

```text
Internet
    │
    ▼
ALB
    │
    ▼
EC2 × 2 (ASG)

    ├─ SSM Endpoint
    └─ S3 Gateway Endpoint

NAT Gateway
```

<br><br><br>
# Project 01
## TerraformによるAWS 2AZ高可用性環境構築

### 構築内容

- VPC
- Public Subnet
- Private Subnet
- NAT Gateway
- S3 Gateway Endpoint

###  詳細構成図

```text
VPC 10.0.0.0/16

Public-A
  NAT

Public-C
  NAT

Private-A
  EC2

Private-C
  EC2

S3 Endpoint
```
<img width="506" height="438" alt="image" src="https://github.com/user-attachments/assets/8052798a-6fe8-409a-98d0-3b3f310ce013" />

<img width="1150" height="616" alt="image" src="https://github.com/user-attachments/assets/d6315007-1421-48d1-b1d9-05d5d2cf262e" />

<br><br><br>
### 検証目的

プライベートサブネット上のEC2が
NAT Gatewayを利用した場合と
利用しない場合で通信経路が
どのように変化するか確認する。

またS3 Gateway Endpointにより
インターネット接続なしで
S3へアクセス可能であることを確認する。

### 検証内容

プライベート上のEC2インスタンスからインターネットへの接続

```text
NATあり

EC2
 ↓
Internet

NATなし

EC2
 ↓
× Internet

EC2
 ↓
○ S3 Endpoint
```

<img width="490" height="301" alt="image" src="https://github.com/user-attachments/assets/1f78d48c-5d9b-4e03-8996-db0d307be882" />


### 学んだこと

- NAT Gatewayはプライベートサブネットから
  インターネットへのアウトバウンド通信を実現する

- NAT Gatewayの削除やルートテーブルからの除外により
  EC2から外部インターネットへの通信が停止する

- S3 Gateway Endpointを利用することで
  NAT Gatewayが存在しなくても
  S3へアクセスできる

- S3 Gateway Endpointを利用することで
  S3向け通信をNAT Gateway経由にしなくて済む

- NAT Gatewayのデータ処理料金を削減できる

- S3利用料金自体は別途発生する

- NAT Gatewayの有無により
プライベートサブネットからの
インターネット接続可否が決まることを
実際に検証できた

<br><br><br>

### つまずきポイント

#### NAT GatewayにSecurity Groupは設定できない

実装当初EC2と同様にSecurity Groupを設定しようとしてしまっていた（座学での知識が使えていなかった）。

しかしNAT GatewayはSecurity Groupを持たず、
通信制御はルートテーブルによって行われることを確認した。

当初はNAT Gatewayの削除をブラウザ上のUIで気軽に行ってしまったりもしたが
stateファイルとの差異が深刻な状況を招いてしまい躓く点になっていた。

現在ではリソースそのものや記載部分をコメントアウトするなどした後にTerraform上からapplyすることを心掛けるようになった。

Terraform stateとの不整合が発生した場合は、
stateの修正やimportなどによる復旧が必要になることを学んだ。

#### S3 Gateway Endpointの通信制御

当初は座学で得た知識を活かせずにSecurity Groupによる制御を行おうとしていた。

しかしGateway EndpointはSecurity Groupを持たず、
ルートテーブルへ自動追加される経路によって
S3への通信が制御されることを改めて確認・理解できた。

#### Terrafomにおけるresouceブロック

Terraformでは通常、
1つのresourceブロックから
1つの管理対象が作成される。

ただしfor_eachを利用した場合は、
1つのresourceブロックから
複数の管理対象が作成される。

当初はresourceブロック内で複数SubnetやRouteTableを
指定していたため、それぞれが個別リソースとして
管理されるものと誤解していた。

<br><br><br>


---
# Project 02
##　ssm接続の追加

## 詳細構成図

### SSM Endpointを追加した構成

```text
VPC 10.0.0.0/16

Public-A
  NAT

Public-C
  NAT

Private-A
  EC2

Private-C
  EC2

SSM Endpoint
S3 Endpoint
```
## 検証目的

SSM Endpointを使ったEC2へのログイン確認

プライベート上のEC2インスタンスからインターネットへの接続可否を確認する

## 検証内容


```text
① NAT Gateway削除

Private Subnet
 ↓
× Internet

② SSM Endpoint追加

Private Subnet
 ↓
○ SSM Endpoint
 ↓
Systems Manager

③ Session Manager接続成功

AWS Console
 ↓
Systems Manager
 ↓
EC2

④ インターネット接続不可を確認

EC2
 ↓
curl google.com
 ↓
× 接続不可
```

① NAT Gatewayを削除し、
   プライベートサブネットから
   インターネットへ接続できないことを確認

② Systems Manager Interface Endpointを追加

③ Session Managerによる接続を確認

④ NAT Gatewayが存在しなくても
   EC2へ管理アクセスできることを確認

<img width="1779" height="756" alt="image" src="https://github.com/user-attachments/assets/5cef0b7b-f7dd-4baa-91df-97e32f7e0192" />




### 学んだこと

- Interface Endpointを利用すると
  AWSサービスへプライベート接続できる

- Private DNSによりAWSサービス名をEndpointへ解決できる

- SSM Endpointによりインターネット接続なしでEC2へ管理アクセスできる

- SSM Interface Endpointを利用することで
  プライベートサブネット上のEC2へ
  インターネット接続なしで管理アクセスできることを
  実際に検証できた


<br><br><br>
  

### つまずきポイント

#### Interface EndpointのSecurity Group設定

<img width="600" height="240" alt="image" src="https://github.com/user-attachments/assets/652ca74e-8f35-41c3-9564-74172a6f4d17" />


実装当初はInterface Endpointに設定する
Security Groupのルールを
IngressにするべきかEgressにするべきか迷ってしまった。

調査を進める中で、
SSM接続は管理端末からEC2へ接続するのではなく、
EC2側からSystems Managerへ通信を開始する仕組みであることを理解した。

そのためInterface Endpoint側では
EC2からのHTTPS通信を受け付けるための
Ingressルールが必要であることを確認できた。

---

#### SSM接続にはインターフェイス型エンドポイントが4つ必要

<img width="681" height="323" alt="image" src="https://github.com/user-attachments/assets/8917c794-896d-46dd-93ed-3b4ca3b4a00c" />

当初はssm接続には３つのエンドポイントが必要だと思い込んでいたが
それだけでは接続を成功させることが出来なかった。

管理用（ssmmaneaged）・ec2→ssm・ssm→ec2への用途ごとの専用エンドポイント以外に、
暗号化通信（kms）を利用するためのエンドポイントが必要となることを理解した。

---

#### Private DNS設定

<img width="534" height="266" alt="image" src="https://github.com/user-attachments/assets/fd0e9794-80d9-4146-bac3-0a2482c966dc" />

<img width="537" height="258" alt="image" src="https://github.com/user-attachments/assets/b9fc0ac1-27d2-4373-b942-be924cdacca5" />


Interface Endpointを正しく設定しただけでは
SSM接続を成功させられなかった。

原因はVPC側のDNS設定および
Endpoint側のPrivate DNS設定であった。

Private DNSを有効化することで
ssm.ap-northeast-1.amazonaws.com が
EndpointのプライベートIPへ解決される仕組みを理解した。

当初はSecurity GroupやIAMロールの問題と考えていたため
原因の切り分けに時間を要してしまった。


<br><br><br>


---
# Project 03

## ALBヘルスチェック失敗時の原因調査と解決

## 詳細構成図

### ALBを追加した構成

```text
ALB
 ↓
EC2
SSM Endpoint

```

## 検証目的

ALBヘルスチェック機能の確認

## 検証内容

Terraformにて構築後、UI操作で片側インスタンスを停止しALBヘルスチェック内容を確認する

<img width="1195" height="626" alt="image" src="https://github.com/user-attachments/assets/af41c145-1a34-4fc0-b8a3-654c0c3609a5" />


### 学んだこと

- ALBはリクエストを受け付ける入口となっている

- ターゲットグループがヘルスチェックを実施する

- ターゲットグループが正常なEC2のみにトラフィックを転送する

- EC2起動中でもWebサーバが応答しなければ異常判定となる

- NAT Gatewayが無い場合、UserDataによるWebサーバ導入に失敗する場合がある

- ALBヘルスチェックにより
  EC2の起動状態だけではなく、
  アプリケーションの応答状況まで
  監視できることを実際に検証できた


### つまずきポイント
Terraformにて構築した際にUI上でインスタンスは正常に動いているにもかかわらずヘルスチェックがすべて異常となっていた

```text

ALB異常
↓
SGか？
↓
ターゲットグループか？
↓
UserDataか？
↓
Apache/Nginxか？
↓
NATが無くてダウンロード失敗

```
ALBのヘルスチェックがアプリケーションレベルでのチェックであることへの意識が実装作業で躓くまで曖昧だった。

座学で学んだ知識のおさらいや試行錯誤を繰り返したためインスタンス上のWebサーバーが応答しない原因を特定するのに苦労した。

インターネットを使いインスタンスへWebサーバーをダウンロードすることでサーバーが立ち上がりヘルスチェックの問題を解決できた。

<img width="803" height="430" alt="image" src="https://github.com/user-attachments/assets/15d1fd60-a207-4a7c-8159-81a35f94c01e" />


Webサーバーの導入後にNATゲートウェイを削除してもALBのヘルスチェックには影響がないことも確認したため時間を費やしてしまった。

Terraformにて構築せずUIでインスタンスを弄ったりしたため何が原因でヘルスチェックが通らないか分からなくなってしまったこともあった

後日あらためてTerraformのみで実装や削除を行いようやくすべてを確信できるまで時間がかかってしまった。

インスタンス上でWebサーバーが立ち上がっていることが確認できた後にNATゲートウェイを削除してしまってもALBのヘルスチェックは正常に反応し続けることを確認できた。



<br><br><br>

# AWS学習ログ

#### その１



<img width="626" height="516" alt="image" src="https://github.com/user-attachments/assets/c199c840-aff9-404f-93d9-f91426f51978" />

Security Groupルールは
  aws_vpc_security_group_ingress_ruleを利用して
  個別管理する方法が現在推奨されている

<br><br><br>

#### その２


<img width="537" height="302" alt="image" src="https://github.com/user-attachments/assets/d282b085-0213-404a-a2e7-edcd1ef95a83" />


for_eachを利用した場合は、
同じ設定内容から複数の管理対象が作成される。

<br><br><br>

#### その３

<img width="769" height="304" alt="image" src="https://github.com/user-attachments/assets/e84d60f9-0ec8-4031-8a9a-9067df0e1f88" />

ｓ３へのｓ３エンドポイントを使った接続検証において「aws s3 ls」ではエラーとなったしまった。

他のリージョンなどを別の作業で使ったりすると場合によっては
オプションでリージョンを指定しないと上手く行かない場合があることが確認できた。



