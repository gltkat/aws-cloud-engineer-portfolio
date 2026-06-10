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

- NAT Gatewayを削除すると
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

当初はEC2と同様にSecurity Groupを設定するものと考えていた。

しかしNAT GatewayはSecurity Groupを持たず、
通信制御はルートテーブルによって行われることを確認した。

### S3 Gateway Endpointの通信制御

当初はSecurity Groupによる制御を想定していた。

しかしGateway EndpointはSecurity Groupを持たず、
ルートテーブルへ自動追加される経路によって
S3への通信が制御されることを理解した。

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







