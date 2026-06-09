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

## 構成図

![Architecture](images/architecture.png)

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
- SSM Endpoint
- S3 Gateway Endpoint

##  詳細構成図

```text
VPC 10.0.0.0/16

Public-A
  ALB
  NAT

Public-C
  ALB
  NAT

Private-A
  EC2

Private-C
  EC2

S3 Endpoint
```

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

## 学んだこと
NAT Gatewayによりプライベート上のインスタンスからインターネットへの接続が可能
NAT Gatewayの削除やルートテーブルからNAT Gatewayへのルートを削除するとインターネットとの接続が止まることの確認
S3 Endpointを設けるとインターネットに繋がらなくてもS3への接続が可能となること


### 以下作成中エリア

- NATあり構成
- NATなし構成
- SSM接続確認
- S3接続確認
- ALB負荷分散確認
- ASG自動復旧確認

## 検証図

## 学んだこと

- EndpointによるAWSサービス接続
- NAT Gatewayとの違い
- Private DNSの重要性







