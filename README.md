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

## 構築内容

- VPC
- Public Subnet
- Private Subnet
- NAT Gateway
- S3 Gateway Endpoint

##  詳細構成図

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

## 検証目的

プライベートサブネット上のEC2が
NAT Gatewayを利用した場合と
利用しない場合で通信経路が
どのように変化するか確認する。

またS3 Gateway Endpointにより
インターネット接続なしで
S3へアクセス可能であることを確認する。

## 検証内容

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

## 学んだこと

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

## つまずきポイント

### NAT GatewayにSecurity Groupは設定できない

実装当初EC2と同様にSecurity Groupを設定しようとしてしまっていた（座学での知識が使えていなかった）。

しかしNAT GatewayはSecurity Groupを持たず、
通信制御はルートテーブルによって行われることを確認した。

当初はNAT Gatewayの削除をブラウザ上のUIで気軽に行ってしまったりもしたが
stateファイルとの差異が時に深刻な状況を招くことを後々の作業での躓きく点になってしまっていた。

現在ではリソースそのものや記載部分をコメントアウトするなどした後にTerraform上からapplyすることを心掛けるようになった。

Terraform stateとの不整合が発生した場合は、
stateの修正やimportなどによる復旧が必要になることを学んだ。

### S3 Gateway Endpointの通信制御

当初は座学で得た知識を活かせずにSecurity Groupによる制御を行おうとしていた。

しかしGateway EndpointはSecurity Groupを持たず、
ルートテーブルへ自動追加される経路によって
S3への通信が制御されることを改めて確認・理解できた。

### Terrafomにおけるresouceブロックとfor each構文の理解
<img width="719" height="456" alt="image" src="https://github.com/user-attachments/assets/060de4b5-6ee9-4ab8-86c4-d5a976653620" />

resourceブロック内に記述された複数の構築物はそれらすべてが１つのグループ（リソース）としてTerraformで管理される
resource中にforeach構文で繰り返せば別グループのリソースとして構築される
<img width="574" height="86" alt="image" src="https://github.com/user-attachments/assets/ae52ea85-6504-4a5b-ac8c-c3e5805f8e5b" />
foreachを使わずリスト化しての表記だと1つのリソース内に2つのルートテーブルが含まれる


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
プライベートEC2から外部への通信が出来ないことをEC2のコンソール操作で確認する

## 検証内容

プライベート上のEC2インスタンスからインターネットへの接続







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







