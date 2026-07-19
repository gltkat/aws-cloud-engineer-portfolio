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

<br>

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
【NATあり】

EC2
 ↓
Internet

【NATなし】

EC2
 ↓
× Internet

EC2
 ↓
○ S3 Endpoint
```

<img width="490" height="301" alt="image" src="https://github.com/user-attachments/assets/1f78d48c-5d9b-4e03-8996-db0d307be882" />

### 学んだこと

・NAT Gatewayはプライベートサブネットからインターネットへのアウトバウンド通信を実現する

・S3 Gateway Endpointを利用すると、NAT Gatewayを経由せずにS3へアクセスできる

・NAT GatewayとS3 Gateway Endpointの役割の違いを実機で検証できた

・Terraformではresource単位で状態管理が行われることを理解できた

<br><br><br>

### つまずきポイント

#### その１：NAT GatewayにSecurity Groupを設定しようとした点

**【気付き】**
 NAT Gatewayは
Route Tableで通信制御を行う

<br>

**【詳細】**

実装当初EC2と同様にSecurity Groupを設定しようとしてしまっていた（座学での知識が使えていなかった）。

しかしNAT GatewayはSecurity Groupを持たず、
通信制御はルートテーブルによって行われることを確認した。

当初はNAT Gatewayの削除をブラウザ上のUIで気軽に行ってしまったりもしたが
stateファイルとの差異が深刻な状況を招いてしまい躓く点になっていた。

現在ではリソースそのものや該当箇所をコメントアウトするなどした後にTerraform上からapplyすることを心掛けるようになった。

Terraform stateとの不整合が発生した場合は、
stateの修正やimportなどによる復旧が必要になることを学んだ。

<br>

---

#### その２：S3 Gateway Endpointの設定への誤解

**【気付き】**
 Gateway Endpointには
Security Groupは設定できない

<br>

**【詳細】**

当初は座学で得た知識を活かせずにSecurity Groupによる制御を行おうとしていた。

しかしGateway EndpointはSecurity Groupを持たず、
ルートテーブルへ自動追加される経路によって
S3への通信が制御されることを改めて確認・理解できた。

<br>

---

#### その3：for_eachによる複数リソース管理を正しく理解できていなかった

**【気付き】**
・リソース中にfor_eachの構文を使うと
複数リソースが生成される

<br>

**【詳細】**
当初はfor_eachの構文を使う方がシンプルに感じていた。

構成を整理する中で各Subnetを個別のresourceとして記述し直したことで、
Terraformがリソース単位で状態を管理していることをより意識できるようになった。 

Terraformでは通常、1つのresourceブロックが1つの管理対象としてTerraform Stateへ登録される。

一方、for_eachを利用した場合は、
1つのresourceブロックから複数の管理対象が生成され、それぞれがTerraform Stateへ独立したリソースとして登録される。



<br><br><br>

---
# Project 02

## ssm接続の追加

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

- SSM Endpointによりインターネット接続なしでEC2へ管理アクセスできる

- Private DNSによりAWSサービス名をEndpointへ解決できる

- SSM Interface Endpointを利用することで
  プライベートサブネット上のEC2へ
  インターネット接続なしで管理アクセスできることを
  実際に検証できた


<br><br><br>
  

### つまずきポイント

#### その１： エンドポイントに付けるSecurity GroupをEgressで設定していた誤り

**【気付き】**
 Iacにおいては理解していたつもりでも勘違いで真逆で設定してしまう危険がある

<br>


**【詳細】**

<img width="600" height="240" alt="image" src="https://github.com/user-attachments/assets/652ca74e-8f35-41c3-9564-74172a6f4d17" />

実装当初においてInterface Endpointに設定する
Security GroupのルールをEgressで設定してしまっていた。

AWSの管理領域とEC2インスタンの中間にエンドポイントは位置している。

そのためInterface Endpoint側では
EC2からのHTTPS通信を受け付けるための
Ingressルールが必要である。

実装作業において不意に勘違いしてコードを入力してしまう怖さを知ると共に
SSM接続は管理端末からEC2へ接続するのではなく、
EC2側からSystems Managerへ通信を開始する仕組みであることを確認できた。

<br>

---

#### その2：必要となるインターフェイス型エンドポイントの作成数を間違っていたこと

**【気付き】**
 SSM接続にはインターフェイス型エンドポイントが4つ必要

<br>

**【詳細】**

<img width="681" height="323" alt="image" src="https://github.com/user-attachments/assets/8917c794-896d-46dd-93ed-3b4ca3b4a00c" />

当初はssm接続には３つのエンドポイントが必要だと思い込んでいた。

しかし実際にはそれだけでは接続を成功させることが出来なかった。

管理用（ssmmaneaged）・ec2→ssm・ssm→ec2への用途ごとの専用エンドポイント以外に、
暗号化通信（kms）を利用するためのエンドポイントが必要となることを理解した。

<br>

---

#### その3： AWS内部における名前解決の重要性を認識出来ていなかった点 

**【気付き】**
 Private DNS設定を有効化しておく必要がある

<br>

**【詳細】**

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

- Terraformにて構築後、ALBへのブラウザによるアクセス画面の確認をする
- AWSのUI操作によって片側インスタンスを停止しALBヘルスチェック内容を確認する

<img width="1195" height="626" alt="image" src="https://github.com/user-attachments/assets/af41c145-1a34-4fc0-b8a3-654c0c3609a5" />


### 学んだこと

- ALBはリクエストを受け付ける入口となっているためALBにはパブリックアドレスが設定される

- ターゲットグループがヘルスチェックを実施する

- ターゲットグループが正常なEC2のみにトラフィックを転送する

- EC2起動中でもWebサーバが応答しなければ異常判定となる

- NAT Gatewayが無い場合、UserDataによるWebサーバ導入に失敗する場合がある

- ALBヘルスチェックにより
  EC2の起動状態だけではなく、
  アプリケーションの応答状況まで
  監視できることを実際に検証できた


### つまずきポイント
UI上でインスタンスは正常に動いているにもかかわらずヘルスチェックがすべて異常となっていた

```text

ALB異常（ブラウザでの表示不可）
↓
Security Groupか？
↓
ターゲットグループか？
↓
UserDataか？
↓
Apache/Nginxか？
↓
NATが無くてダウンロード失敗

```
ALBのヘルスチェックがアプリケーションレベルでのチェックであることへの意識が不具合に直面するまで曖昧だった。

座学で学んだ知識のおさらいや試行錯誤を繰り返したためインスタンス上のWebサーバーが応答しない原因を特定するのに苦労した。

調査の結果、NAT Gatewayが存在しなかったためUserDataによるWebサーバー導入処理が失敗していたことが判明した。

そのためEC2自体は起動していたものの、
Webサーバーが動作しておらず
ALBヘルスチェックが異常となっていた原因を特定できた。

<img width="803" height="430" alt="image" src="https://github.com/user-attachments/assets/15d1fd60-a207-4a7c-8159-81a35f94c01e" />

#### 再検証が必要となったケース

```text

① ALBヘルスチェックの正常化

   ↓

④ NAT削除

⑤ なぜか再び異常化

⑥ 理屈と合わない

⑦ UI操作でEC2停止・削除・再作成などを繰り返す

⑧ Terraform管理との整合性が怪しくなる

⑨ 後日Terraformのみで再構築

⑩ NAT削除後も正常

⑪ 理論通りの結果を確認

```

Webサーバー導入後にNAT Gatewayを削除した場合において、
本来であればALBヘルスチェックは正常を維持するはずだった。

しかし初期の検証においてはヘルスチェックが異常となる現象が発生した。

後日同じTerraformのplanを実行すると
NAT Gatewayを削除しても、
既にWebサーバーが起動済みであれば
ALBヘルスチェックは正常を維持することを確認できた。

不可解な現象が起こってしまった際にはAWSコンソールからEC2の停止・削除・再作成などを繰り返していたため、
Terraform管理外の変更が多数発生していた。

AWSコンソールによる変更や複数の検証が重なったことで、Terraform Stateと実環境の整合性を含め、原因の切り分けが困難になった。

不可解な現象そのものについては真因を特定するに至らなかったが、再検証によって理論通りの挙動を確認できた。

```text

Webサーバ導入済
＋
NAT Gateway削除
↓
ALBヘルスチェック正常

```

この経験から、Terraform管理下のリソースをコンソールから直接変更すると検証結果の再現性が失われる可能性を常に意識する必要があることを学んだ。

結局なぜ初期の実装においてNATゲートウェイを削除するとヘルスチェックが異常となってしまうのか分からない怖さを実感できた。

そして構成管理ツールのみで状態を管理することの重要性を学ぶことができた。

<br><br><br>



---

# Project 04

## Launch Templateを中心としたEC2の自動展開基盤の構築

## 詳細構成図

### Auto Scaling Groupを追加した構成
```text

                Internet
                    │
                    │
             ┌────────────┐
             │     ALB    │
             └─────┬──────┘
                   │
             Target Group
                   │
      ┌────────────┴────────────┐
      │                         │
┌─────────────┐          ┌─────────────┐
│   EC2(AZ-a) │          │   EC2(AZ-c) │
│ Auto Scaling│          │ Auto Scaling│
└──────┬──────┘          └──────┬──────┘
       │                         │
       └────────────┬────────────┘
                    │
          Auto Scaling Group
                    │
           Launch Template
                    │
        UserData / IAM Role /
           Instance Profile
```


### 学んだこと

- Launch TemplateにはAMI・UserData・IAM設定など、EC2の構成情報をまとめて定義できる

- Auto Scaling GroupはLaunch Templateを利用してEC2を起動・終了する

- Auto Scaling Groupは起動したEC2をTarget Groupへ自動登録する

- EC2へIAMロールを付与する際は、TerraformではLaunch TemplateにInstance Profileを指定する必要がある

- Launch TemplateとAuto Scaling Groupを組み合わせることで、
同一構成のEC2を自動展開できることを実際に検証できた

<br><br>


### つまずきポイント
```text

          Auto Scaling Group					NATなし
                   │                           　 ↓
         Launch Template					    UserData失敗
                   │                            　↓
        ┌──────────┴──────────┐		          　Apache導入失敗
        │                     │                　 ↓
   EC2 Instance A        EC2 Instance C			Target Group 
        │                     │                	Unhealthy
   UserData実行          UserData実行				
        │                     │
 Apache導入成功        Apache導入成功				 
        │                     │
        └──────────┬──────────┘		
                   │					
             Target Group					
                   │
                  ALB
```


**【現象】**
Auto Scaling Groupを作成しただけでは
期待したEC2は作成されなかった

**【原因】**
Launch Templateに
Instance Profileを設定していなかった

**【解決】**
Launch Templateへ
Instance Profileを追加

<br><br>


>#### 【解決詳細】：**Launch TemplateとInstance Profileの関係**

調査を進める中で、以下のような役割分担を理解することができた。
```text
IAM Policy

↓

IAM Role

↓

Instance Profile

↓

Launch Template

↓

Auto Scaling Group

↓

EC2
```




Auto Scaling Groupを追加すれば自動的にEC2が起動すると考えていたが、
実際にはLaunch TemplateにAMI・UserData・IAMなど必要な設定を定義しておかなければ期待したEC2は作成されなかった。

特にIAMロールはAuto Scaling Groupへ直接指定するのではなく、
Launch Template内でInstance Profileを指定する必要がある点の理解に時間を要してしまった。


<br><br>

### 考察：CloudFormationとTerraformの相違

**【現象】** 
同様の構成をTerraformで実装するにあたって想定以上に時間を要してしまった

**【原因】**
tfファイルを複数に分けて作成しはじめたことによる依存関係の混乱

**【まとめ】**
独立して存在できるリソースと他のリソースありきで作成可能なリソースの違いに注目すること

<br><br>

>#### 【考察詳細】
#####  CloudFormationは「完成した構成」を記述する

CloudFormationではAuto Scaling Groupの設定項目として
Launch TemplateやTarget Groupなどを記述していたため
それぞれの役割や責務について注意が行き届いていなかった。

YAMLという書式の特徴であるインデントの不備などに注意が向けられていた。
<img width="542" height="467" alt="image" src="https://github.com/user-attachments/assets/40e652bf-6c1c-4963-99f9-06f8cdcb589c" />


##### Terraformは「構成部品」を組み立てる

Terraformでは様々なAPIに個別に対応した形でリソースを記述する。

Launch Template・Target Group・Instance Profileなどを独立したリソースとして実装しなくてはならない。

<img width="526" height="351" alt="image" src="https://github.com/user-attachments/assets/2f50926b-035f-4608-b5f9-79ce64def866" />

実装を進める中で、

- Launch TemplateがEC2の構成を定義すること

- Auto Scaling GroupがLaunch Templateを利用してEC2を管理すること

- Target GroupがALBとAuto Scaling Group双方から利用されること

など、各リソースの責務や依存関係を理解することができた。

<br><br>
### 考察を終えて

```text
CloudFormationでは完成したサービスとして扱っていた構成を

Terraformではリソース単位で実装したことで

各リソースの責務や依存関係に以前より注目できるようになった
```
<br><br><br>

# AWS学習ログ

### ALBとターゲットグループ

Terraform実装を進める中で、
ALBは単純にEC2へ負荷分散しているわけではなく、
ターゲットグループを通じて
正常なターゲットへ振り分けていることを理解した。

当初はEC2単位の振り分けのみを想定していたが、
AWS資格学習を通じてECSでは
コンテナ単位で負荷分散できることも学んだ。

- ALBが振り分ける相手はターゲット
- コンテナ実装時にターゲットグループへ登録するのは誰？　➡ ECSサービス

```text
load_balancer {
  target_group_arn = aws_lb_target_group.web.arn

  container_name = "nginx"

  container_port = 80
}

```
実装と資格学習を往復することで、サービスの仕組みへの理解を深めることができた。

<br><br><br>

#### その他１



<img width="626" height="516" alt="image" src="https://github.com/user-attachments/assets/c199c840-aff9-404f-93d9-f91426f51978" />

Security Groupルールは
  aws_vpc_security_group_ingress_ruleを利用して
  個別管理する方法が現在推奨されている

<br><br><br>

#### その他２


<img width="537" height="302" alt="image" src="https://github.com/user-attachments/assets/d282b085-0213-404a-a2e7-edcd1ef95a83" />


for_eachを利用した場合は、
同じ設定内容から複数の管理対象が作成される。

<br><br><br>

#### その他３

<img width="769" height="304" alt="image" src="https://github.com/user-attachments/assets/e84d60f9-0ec8-4031-8a9a-9067df0e1f88" />

ｓ３へのｓ３エンドポイントを使った接続検証において「aws s3 ls」ではエラーとなったしまった。

他のリージョンなどを別の作業で使ったりすると場合によっては
オプションでリージョンを指定しないと上手く行かない場合があることが確認できた。

#### その他4

- Instance Profileには1つのIAMロールを関連付けることができ、
そのIAMロールには複数のIAMポリシーをアタッチできる
