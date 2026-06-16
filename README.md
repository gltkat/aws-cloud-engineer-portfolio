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
<img width="717" height="427" alt="image" src="https://github.com/user-attachments/assets/3e3eb2e7-14e5-4067-8cdd-f279a4e17389" />
<img width="1458" height="771" alt="image" src="https://github.com/user-attachments/assets/586b1ff4-8bb7-4b3c-bf0e-c790568933f8" />

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

<img width="657" height="423" alt="image" src="https://github.com/user-attachments/assets/9694c1ea-449c-4d9a-8f60-39e8afb205b5" />

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

### Terraform実装メモ
<img width="537" height="302" alt="image" src="https://github.com/user-attachments/assets/d282b085-0213-404a-a2e7-edcd1ef95a83" />

for_eachを利用した場合は、
同じ設定内容から複数の管理対象が作成される。

# Project 02

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

プライベートEC2から外部への通信が出来ないことをEC2のコンソール操作にて確認する

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
```

NAT Gatewayを利用せず、
Systems Manager Interface Endpointを利用することで、

インターネット接続を持たない
プライベートサブネット上のEC2へ
SSM接続できることを確認する。

Private DNSを利用することで名前解決が
SSM接続に重要であることを確認する。
<img width="688" height="421" alt="image" src="https://github.com/user-attachments/assets/7daffea6-6ab8-4621-85f1-3ea15d0529ad" />

### 学んだこと

- SSM接続によってインターネット通信を経由することなくプライベートインスタンスへの接続が達成できる

- SSM接続はEC2からSystems Managerへの
  アウトバウンド通信で成立する

- Interface EndpointにはSecurity Groupを設定する
- S3ゲートウェイエンドポイントはルートテーブルで行き先を決めるが
  インターフェイス型のエンドポイントはIP（名前解決）により通信先が見つける仕様となっている

- Endpoint側でEC2からのTCP443通信を許可する必要がある

- Private DNSを有効化することで
  AWSサービスのDNS名をEndpointの
  プライベートIPへ解決できる

- VPCの設定でDNSによる名前解決をプライベートIPに対しても可能になるように設定することが必要


### つまずきポイント

<img width="1842" height="850" alt="image" src="https://github.com/user-attachments/assets/ed0f490f-a646-4b62-8cb7-85324996f31b" />


#### SSM接続によるインスタンスへの接続は、インスタンスから能動的に管理側へアクセスしてくることで成立する

管理側はつねにポーリングによりインスタンスからの通信を待ち受けているだけなので中間に位置するインターフェイス型のエンドポイントへのルール設定はingressとなる。
実装当初はingressなのかegressかで迷うこともあったが演習により基準が明確になった。

#### ｓｓｍ接続するにはインターフェイス型のエンドポイントが４つ必要

ｓｓｍ接続には管理用（ssmmaneaged）・ec2→ssm・ssm→ec2への専用のエンドポイントが必要という認識しかなかったが
実際には暗号化通信（kms）を利用するためエンドポイントも必要であることを理解・確認できた。
<img width="681" height="323" alt="image" src="https://github.com/user-attachments/assets/8917c794-896d-46dd-93ed-3b4ca3b4a00c" />

#### Private DNS設定

SSM Interface Endpointを作成しただけでは
SSM接続できなかった。

原因はVPC側のDNS設定および
Endpoint側のPrivate DNS設定であった。

Private DNSを有効化することで
ssm.ap-northeast-1.amazonaws.com が
EndpointのプライベートIPへ解決される仕組みを理解した。

当初はSecurity GroupやIAMロールの問題と考えていたため
原因の切り分けに時間を要した。



### Terraform実装メモ

Security Groupルールは
  aws_vpc_security_group_ingress_ruleを利用して
  個別管理する方法が現在推奨されている

<img width="626" height="516" alt="image" src="https://github.com/user-attachments/assets/c199c840-aff9-404f-93d9-f91426f51978" />







<img width="1240" height="318" alt="image" src="https://github.com/user-attachments/assets/565a2357-1f2a-453c-ae1c-c36dd4e24194" />

ｓ３へのｓ３エンドポイントを使った接続検証において「aws s3 ls」ではエラーとなったしまった。
他のリージョンなどを別の作業で使ったりすると場合によってはオプションでリージョンを指定しないと上手く行かない場合があるということが確認できた。




### 以下作成中エリア

- NATあり構成
- NATなし構成
- SSM接続確認
- S3接続確認
- ALB負荷分散確認
- ASG自動復旧確認

#### 検証図

#### 学んだこと

- EndpointによるAWSサービス接続
- NAT Gatewayとの違い
- Private DNSの重要性







