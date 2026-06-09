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

# Project

## 概要

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
- ALB
- Auto Scaling Group
- NAT Gateway
- SSM Endpoint
- S3 Gateway Endpoint

##  詳細構成図

---
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

SSM Endpoint
S3 Endpoint
---

## 検証内容

- NATあり構成
- NATなし構成
- SSM接続確認
- S3接続確認
- ALB負荷分散確認
- ASG自動復旧確認

## 学んだこと

- EndpointによるAWSサービス接続
- NAT Gatewayとの違い
- Private DNSの重要性
