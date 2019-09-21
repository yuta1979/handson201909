### 事前準備

* ゲスト用wifiにつないでいること
* ソフトウェアダウンロード
 * TeraTerm https://ja.osdn.net/projects/ttssh2/
 * テキストエディタ

* ハンズオン資料・template.ymlのダウンロード
OneDrive [LINK]
(https://tdcsoft-my.sharepoint.com/:f:/g/personal/shimada_yuu_tdc_co_jp/EkuHPECj0wBAhcbo5WsilOYBT0DPA3VT8qfrAhoLKC126Q?e=ve4h0q)

---
## ハンズオンシナリオ CloudFormation編
### TDC AWS WorkingGroup

---
### 講師について
* 名前：島田雄太　(2003年入社)
* 所属：インテグレーションサービス部
* 大まかな経歴：　ほぼインフラ屋
 - 当社サービスのインフラ設計/構築/運用 10年
 - 客先インフラ構築支援 3年
 - AWS環境構築 (当社ホームページ、エンタープライズ向け顧客など) 2年
* 資格：PMP、AWSソリューションアーキテクト アソシエイト
* 好きなAWSサービス：Route53


---
### CloudFormationとは
AWS上に自動でリソースを作成する定義ファイルを実行するサービスになります。

---
定義ファイルの書き方は2種類あり、JSON形式とYAML形式があります。
本ハンズオンでは、ファイルにコメントを残せるYAML形式を採用します。

### 補足) YAMLの書き方について
https://qiita.com/fkana/items/21f7cc3b327445483d5c

---
### CloudFormationのメリット
* すべてのモデル化

AWS CloudFormation では、お客様のインフラストラクチャ全体をテキストファイルでモデル化できます。
このテンプレートは、インフラストラクチャにおける真の単一ソースとなります。
これにより、組織全体にわたって、使用されるインフラストラクチャコンポーネントを標準化し、
構成の準拠とトラブルシューティングの時間短縮を実現します。

---
* 自動化とデプロイ

AWS CloudFormation では、安全で繰り返し可能な方法でリソースがプロビジョニングされるため、
手作業やカスタムスクリプト作成を必要とせずにインフラストラクチャとアプリケーションの構築と再構築が可能になります。
スタックの管理時には、実行に適した操作が CloudFormation によって自動的に決定され、エラーが検出された場合は
変更が自動的にロールバックされます。

---
* 単なるコード

インフラストラクチャをコード化することで、インフラストラクチャを単なるコードとして扱えるようになります。
任意のコードエディタで作成してバージョン管理システムにチェックインし、プロダクションにデプロイする前に
チームメンバーとファイルを確認することができます。
(AWSのドキュメントより引用)

---
### CloudFormationの概念

AWS CloudFormation を使用する際には、テンプレートとスタックの作業を行います。
テンプレートは、AWS リソースとそのプロパティを記述するために作成します。
スタックを作成するたびに、AWS CloudFormation はテンプレートに記述されているリソースを
プロビジョニングします。
実行されたテンプレートはS3バケット上に保存されます。

---
### テンプレート
AWS CloudFormation テンプレートは JSON または YAML 形式のテキストファイルです。
これらのファイルは、.json、.yaml、.template、.txt などの拡張子を使用して保存できます。
AWS CloudFormation はこれらのテンプレートを AWS リソースを作成する際の設計図として使用します。

---
```
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template
Resources:
  MyEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: "ami-0ff8a91507f77f867"
      InstanceType: t2.micro
      KeyName: testkey
      BlockDeviceMappings:
        -
          DeviceName: /dev/sdm
          Ebs:
            VolumeType: io1
            Iops: 200
            DeleteOnTermination: false
            VolumeSize: 20
```

---
また単独のテンプレートに複数のリソースを指定し、これらのリソースが連携するように設定できます。
たとえば、Elastic IP (EIP) を含み、Amazon EC2 インスタンスとの関連付けを持つように、
上記のテンプレートを変更することができます。

---
```
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template
Resources:
  MyEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: "ami-0ff8a91507f77f867"
      InstanceType: t2.micro
      KeyName: testkey
      BlockDeviceMappings:
        -
          DeviceName: /dev/sdm
          Ebs:
            VolumeType: io1
            Iops: 200
            DeleteOnTermination: false
            VolumeSize: 20
  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyEC2Instance
```
---
### スタック
AWS CloudFormation を使用する際、関連リソースはスタックと呼ばれる単一のユニットとして
管理します。スタックを作成、更新、削除することで、リソースのコレクションを作成、更新、削除します。
スタック内のすべてのリソースは、スタックの AWS CloudFormation テンプレートで定義されます。

---
削除が出来るため、1つのスタックにまとめすぎると運用が回らなくなる場合もあります。
サービスカット(VPC,セキュリティーグループ,EC2,ALBRDS)でテンプレートを分割するパターンが多くみられます。

---
### 変更セット
スタックで実行中のリソースに変更を加える必要がある場合は、スタックを更新します。
リソースに変更を加える前に、変更案の概要である変更セットを生成できます。
変更セットで、変更が実行中のリソース、特に重要なリソースに与える可能性のある影響を、
実装前に確認できます。

---
Amazon RDS データベースインスタンスの名前を変更すると、AWS CloudFormation によって
新しいデータベースが作成され、古いものは削除されます。
古いデータベースのデータは、バックアップしていない限り、失われます。
変更セットを生成すると、変更によってデータベースが置き換えられることがわかり、
スタックを更新する前に対応策を立てることができます

---
### ハンズオン
実際に触ってみよう！

---
### アカウント

ハンズオン用AWSアカウントのURL
https://743559742203.signin.aws.amazon.com/console
ログインIDはメールアドレス
パスは当日説明

---

ハンズオン資料格納OneDrive
[LINK] 
(https://tdcsoft-my.sharepoint.com/:f:/g/personal/shimada_yuu_tdc_co_jp/EkuHPECj0wBAhcbo5WsilOYBT0DPA3VT8qfrAhoLKC126Q?e=ve4h0q)

キーペアを作成する
 EC2 -> ネットワーク＆セキュリティ -> キーペア
キーペアの作成 -> key-社員番号 (key-t2003006)

---
## シナリオ
今回は同一のVPC上にWEBサーバとDBサーバを構築します

---
### リソースを定義するためのTemplateを書く

管理するAWSリソースを定義するテンプレートです。(yml)
まずはこんな形のymlを用意しましょう

```template0.yml
AWSTemplateFormatVersion: '2010-09-09'
Description: CFn hands On 201909
Parameters:
# Parameters以下に変数を定義する
Resources:
# Resources以下に管理したいResourceを管理する
Outputs:
# エクスポートしたい値を書く
```

---
### Parameters (変数)
環境変数を定義します。

```template1.yml
Parameters:
  Prefix:
    Type: String
    Default: UserName
    Description: Prefix Name

  ServiceName:
    Type: String
    Default: CFn-handson

  KEYNAME:
    Type: String
    Default: KeyName
    Description: YourKeyName

  VPCID:
    Type: String
    Default: vpc-xxxxxxxxxxxxxx
    Description: VPC ID(Do not change)

  SubnetID:
    Type: String
    Default: subnet-xxxxxxxxxxxxxxx
    Description: Subnet ID(Do not change)
```

---
型指定とDefault値を指定できます。
詳しいオプションは

公式ドキュメント Parameters [LINK](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) を参照してください。

---
### Resouces
EC2やRDS、セキュリティグループ等AWSで使うサービスの定義をすべてしていきます。

ではまずはWEBサーバーを作成するにあたってのAWSリソースで最低限必要なものは下記です。
* SecurityGroup
* EC2
これを定義していきましょう

---
```template1.yml
Resources:
  Ec2SecurityGroupApp:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: !Sub "${Prefix}_${ServiceName}_app"
      GroupDescription: "web server security group"
      VpcId: !Ref VPCID
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: "0.0.0.0/0"
      Tags:
        - Key: Name
          Value: !Sub "${Prefix}_${ServiceName}_app"

  Ec2InstanceServer:
    Type: "AWS::EC2::Instance"
    Properties:
      # AmazonLinux2を指定しています
      ImageId: "ami-0ff21806645c5e492"
      InstanceType: "t2.micro"
      KeyName: !Ref KEYNAME
      SecurityGroupIds:
        - !Ref Ec2SecurityGroupApp
      SubnetId: !Ref SubnetID
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 8
            VolumeType: gp2
      Tags:
        - Key: Name
          Value: !Sub "${Prefix}_${ServiceName}_app"
```

---
上から順番に説明していきます

Ec2SecurityGroupApp, Ec2InstanceServer (ResourceKeyName)
リソースの名前です。任意の名前をつけてください。

---
### Type
どういったリソースを配置するかを定義する。

### Properties:xxxxxxxx
指定したリソースのプロパティーです。

詳細は以下を参照
公式CloudFormation AWS リソースおよびプロパティタイプのリファレンス [LINK](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

---
### !Sub,!Ref (CloudFormationの組込関数)
CloudFormationでは組込み関数が用意されています。

公式CloudFormation 組込み関数 [LINK](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

書き方に少し癖があるのですが下記を覚えてしまえば難しくないです。

---
### !{関数名}はショート記法
例えばショート記法を使わない場合でRefを使用したい場合だと

```
SecurityGroupIds:
 - Fn::Ref: Ec2SecurityGroupApp
```
と書きます。

---
:を段落区切りを表すyamlの場合この方式で書くと少々可読性が悪くなります。
なのでショート記法で書いている人のほうが多いです。

---
### Ref
!Ref {パラメーター名}
でパラメーターの値を参照できます。

!Ref {リソース名}
で配置するリソースの返り値を取得できます。

---
大概の場合リソースを参照したい場合は乱暴に
!Ref {リソース名}
と宣言すればよしなに参照できます。

---
### Sub
Parameterと文字列のテンプレートリテラルです。
文字列 + Parameterで値を作成したいときにつかってください。

---
### 実際に作ってみる
ではAWS マネジメントコンソール上からCloudFormationを実際に動かしてみましょう。

ここまでのtemplateファイルは下記です。
template1.yml

EC2のGUIを確認してください。t2.microで {Prefix}-{ServiceName}_appというサーバーが立ち上がっているのが確認できます。
セキュリティグループもできていることを確認します。

---
### RDSを作成してみる
ここまでできたら要領は把握したのでRDSも作ってみましょう

---
```(抜粋)template2.yml
  RDSDBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: "DBSubnets for RDS"
      SubnetIds:
        - !Ref DBSubnetID1
        - !Ref DBSubnetID2
  # RDSのセキュリティグループ
  RdsSecurityGroupApp:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: !Sub "${Prefix}_${ServiceName}_rds"
      GroupDescription: "dbserver security group"
      VpcId: !Ref VPCID
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 3306
        ToPort: 3306
        SourceSecurityGroupId: !Ref Ec2SecurityGroupApp
      Tags:
        - Key: Name
          Value: !Sub "${Prefix}_${ServiceName}_rds"
```
---
```
 RdsInstace:
    Type: "AWS::RDS::DBInstance"
    DeletionPolicy: Delete
    Properties:
      DBInstanceIdentifier: !Sub "${Prefix}-${ServiceName}-db"
      DBSubnetGroupName: !Ref RDSDBSubnetGroup
      DBInstanceClass: "db.t2.micro"
      AllocatedStorage: 6
      StorageType: gp2
      VPCSecurityGroups:
        - !Ref RdsSecurityGroupApp
      Engine: MySQL
      EngineVersion: 5.7.23
      DBName: cfntest
      MasterUsername: cfntest
      MasterUserPassword: !Ref DBPASSWORD
      Tags:
        - Key: Name
          Value: !Sub "${Prefix}_${ServiceName}_rds"
```

---
ここまでのtemplate.ymlは下記です

template2.yml

---
では再度マネジメントコンソールから実行します。

再度実行するときに冒頭説明した変更セットを利用します。
変更セットを利用することでtemplateやパラメーターをスタックに送信しますが
リソースの変更を行わずにリソース変更部分の差分を出してくれます。

dry-runをイメージしてもらえるとわかりやすいと思います。
では実際にやってみましょう。

---
スタックの状況が

UPDATE_COMPLETE

になったら正しく反映できています。

---
### EC2インスタンスにSSHポートを追加する
EC2のセキュリティグループに22番ポートが開いていなくてsshログインできないですね…
出来上がったEC2の内容を確認できません。
ですがセキュリティグループは慎重にやらないと事故のもとです。

template2.ymlに追記しましょう

```
Resources:
  Ec2SecurityGroupApp:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: !Sub "${Prefix}_${ServiceName}_app"
      GroupDescription: "web server security group"
      VpcId:
        "{your VPCID}"
```

---
```
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: "0.0.0.0/0"
        # 追加内容
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: "0.0.0.0/0"
      Tags:
        - Key: Name
          Value: !Sub "${Prefix}_${ServiceName}_app"
```

---
再度変更セットを実行します。

変更した内容がEC2のセキュリティグループだけなのがわかりますね。
では変更セットの詳細画面右上の実行ボタンを押して反映させてみます。

RDSに接続できるか確認してみましょう
TeraTermを利用して作成したインスタンスにログインします
(ユーザ名:ec2-user)

---
```
sudo yum localinstall -y https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo yum -y install mysql-community-client
mysql -u cfntest -p -h RDSエンドポイント
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.7.23-log Source distribution

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

```

---
```
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cfntest            |
| innodb             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)
```

---
### スタックの掃除をしてみましょう
これでCloudFormationの流れがつかめてきたと思います。
では最後にCloudFormationの削除を実行してみましょう。

CloudFormationで定義されたリソースの削除は非常に簡単にできます。

aws cloudformation delete-stack --stack-name {任意のスタック名}
これだけです。

定義したリソースすべてをキレイサッパリ削除してくれます。
リソースの消し忘れがなくて良いですね!!

---
### クロージング
* AWSなどクラウドサービスの困りごとや気になったこと
* Teams紹介 (コード: zi1ntls )
* アンケートについて

---
ありがとうございました！
