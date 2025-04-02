# OCIハンズオン-HeatWave Replication編

AWS RDS for MySQLからHeatWave MySQLに移行を実施する際、様々な手法/手段が考えられますが今回はMySQLの機能を利用したレプリケーションを実施します。
HeatWave MySQLは仕様上、RDSとは異なりパブリックエンドポイントを持つことは現時点(2025年4月時点)ではできません。

# 作業の流れ
1. AWSとOCI間でVPN接続した環境の構築
1. OCI ComputeにてMySQLクライアントのセットアップ
1. RDSの構築及び、レプリケーションに向けたセットアップ
1. RDS→HeatWave MySQLへのレプリケーション設定
1. 同期確認

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/c4c81e14-d3ad-6662-7b46-e60091fb2544.png)

# 1. AWSとOCI間でVPN接続した環境の構築
OCI HeatWave MySQLはパブリックエンドポイントを持つことができないため、AWSとOCIをVPN接続する必要があります。
今回、VPN接続の設定方法は以下記事を参考にしました。

https://qiita.com/yama6/items/c188de191269cb604341

無事にVPN接続ができれば以降の工程に進みます。

# 2. OCI ComputeにてMySQLクライアントサーバのセットアップ
## 2-1.MySQLのインストール
MySQLチームが提供しているyumの公式リポジトリをセットアップします
リポジトリ追加
```
 sudo yum install https://dev.mysql.com/get/mysql80-community-release-el8-4.noarch.rpm
```
RHEL8系のOSの場合、デフォルトで有効になっているMySQLモジュールを無効化する必要があるため、以下のコマンドを実行
デフォルトMySQLモジュールの無効化
```
sudo yum module disable mysql
```
mysqlのインストール
```
sudo yum install mysql-community-server
```

mysqlshellのインストール
```
sudo yum install mysql-shell
```
# 3. RDSの構築及び、レプリケーションに向けたセットアップ
## 3-1 RDSの構築
RDS構築の際、以下を実施する必要があります。
①カスタムパラメータグループ にて以下3オプションの有効化を実施	
- GTID-MODE=ON
- ENFORCE_GTID_CONSISTENCY=ON
- binlog_format=ROW
 
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/79a1a3f7-194e-fb67-ecc7-ffb092949c3a.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/7f8a25e3-3dd8-e75a-d5c4-fba0006db67b.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/be958285-ead2-984b-4b8f-74c8d029b9f8.png)

②構築時のオプションにて①で作成したカスタムパラメータグループの指定

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/f974cc54-1a59-cf49-9be3-f82d0a115ec0.png)


③パブリックアクセスの無効化
※VPN経由の場合、パブリックアクセスを有効化するとエンドポイント指定時にグローバルIPが返ってきて接続エラーとなるので注意ください

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/4caae9eb-78f7-658d-973f-2f4a331bed63.png)

④自動バックアップの有効化

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/ffb78cae-bf81-f95f-0c0c-47a2a8ab46a3.png)

## 3-2 RDSのセットアップ
踏み台サーバからRDSにアクセスを実施
```
mysql -u admin -p -h エンドポイント
```

レプリケーションユーザ作成を実施
※後ほどHeatWave MySQLのチャネル設定で利用するのですが、以下のパスワードポリシーがあるためそれに準拠した任意のパスワードとしてください
パスワードは8文字から32文字までの長さで、大文字、小文字、数字および特殊文字をそれぞれ1つ以上含める必要があります。

```
create user usertemp@'%' identified  by 'Manager@123';
```
```
grant replication slave on *.* to usertemp@'%';
```


次にRDSのGTID_executedをメモしておきます。
```
mysql> show global variables like 'GTID%';
+----------------------------------+-------------------------------------------+
| Variable_name                    | Value                                     |
+----------------------------------+-------------------------------------------+
| gtid_executed                    | f8fc4d14-79d8-11ee-a0fa-06d25f27ce47:1-14 |
| gtid_executed_compression_period | 0                                         |
| gtid_mode                        | ON                                        |
| gtid_owned                       |                                           |
| gtid_purged                      | f8fc4d14-79d8-11ee-a0fa-06d25f27ce47:1-10 |
+----------------------------------+-------------------------------------------+
5 rows in set (0.01 sec)
```

# 4.RDS→HeatWave MySQLへのレプリケーション設定
## 4-1 HeatWave MySQL側のレプリケーションセットアップ
先ほどRDSで確認したGTIDをパージします
```
call sys.set_gtid_purged("RDSで確認したGTID");
call sys.set_gtid_purged("f8fc4d14-79d8-11ee-a0fa-06d25f27ce47:1-14");
```

## 4-2 RDSからMySQLへのレプリケーションを設定します：OCI コンソールに移動
作成したHeatWave MySQLの画面より"チャネル"タブへ移動し、チャネルの作成を実施します。

| 項目                          | 設定値                                                                 |
|-------------------------------|----------------------------------------------------------------------|
| **チャネル作成**               |                                                                      |
| 名前                          | rds_channel (任意項目)                                               |
| 説明                          | rds_channel (任意項目)                                               |
| **ソース接続**                 |                                                                      |
| ホスト名                      | RDSのエンドポイント                                                  |
| MySQLポート                   | 3306                                                                 |
| ユーザ名                      | RDSで作成したレプリケーション用ユーザ名                               |
| パスワード                    | RDSで作成したレプリケーション用ユーザのパスワード                     |
| SSLモード                     | 無効 (DISABLED)                                                     |
| レプリケーションのポジショニング | ソースで、GTID自動ポジションを使用できます (推奨)                     |
| **ターゲットDBシステム**        |                                                                      |
| Applierユーザ名               | 空白                                                                |
| チャネル名                    | replication_channel (任意項目)                                       |
| **主キーのない表**             | 許可 (ALLOW)                                                        |
| **チャネル・フィルタ・オプションの表示** |                                                                      |
| 共通フィルタ・テンプレート      | AWS RDS MySQL8.0                                                    |

※チャネル・フィルタ・オプションの設定を忘れるとチャネル作成後、エラーとなるので必ず設定してください
詳細背景については[こちら](https://blogs.oracle.com/mysql/post/successful-rds-to-oci-mysql-heatwave-migration-with-replication-channel-filters
)に記載がありますが、RDSで作成される特定のテーブルがエラー起因となるようでそれを回避するためになります

以下入力イメージ

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/51d25d6f-55a2-3f56-1fac-93ab6dbfa16e.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/f9e13886-46a5-ed0b-e5fa-a9e48e820655.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/3df8b765-ccce-2f00-0d56-837497ec07bb.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/edadc27d-4281-cfc3-a3d1-ebc98724a381.png)

チャネル設定完了後、以下のようにアクティブなっていれば問題ありません

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2471135/1fd0911b-546d-8dcc-0b25-0c22f2bc0f80.png)

# 5同期確認
## 5-1 RDSにてDB作成
同期確認のため、マスタであるRDSにアクセス、DB作成をします。
```
mysql -u admin -p -h エンドポイント
```
DBの作成
```
CREATE DATABASE rpl;
```
テーブル作成
```
CREATE TABLE rpl.test(id int AUTO_INCREMENT, col1 CHAR(10), PRIMARY KEY(id));
```
データ挿入
```
INSERT INTO rpl.test VALUES(1, "TEST");
```
結果確認
```
SELECT * FROM rpl.test;

以下実行結果
mysql> SELECT * FROM rpl.test;
+----+------+
| id | col1 |
+----+------+
|  1 | TEST |
+----+------+
1 row in set (0.01 sec)
```

## 5-2 HeatWave MySQLにてレプリケーションされているか確認
```
mysql -u admin -p -h ホスト名orプライベートIP
```

DB一覧を確認し、"rpl"があるこを確認
```
SHOW DATABASES;

以下実行結果
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mysql_audit        |
| performance_schema |
| rpl                |
| sys                |
+--------------------+
6 rows in set (0.00 sec)
```
RDSと同様の結果が返ってくること確認
```
SELECT * FROM rpl.test;
以下実行結果
mysql> SELECT * FROM rpl.test;
+----+------+
| id | col1 |
+----+------+
|  1 | TEST |
+----+------+
1 row in set (0.00 sec)

```
