# OCIハンズオン-HeatWave Migration編

MySQL Shellを利用したDumpの作成と、Dumpファイルを元にしたOCI MySQL HeatWave Database Serviceの作成手順をガイドします。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/e58a455d-ed6a-b337-ac9a-9667ed2f2126.png)


## 1. 前提

- AWSとOCI、それぞれに接続するための踏み台・作業用サーバがパブリックサブネット上に構築されていること。

MySQL公式サイトにて案内されていた以下ドキュメント内の手順を参照しています。

https://www.mysql.com/jp/why-mysql/presentations/amazon-rds-aurora-mysql-heatwave-database-service-jp/

## 動作検証用データベース
本ハンズオンでは、以下サンプルのemployees databaseを利用して移行前後のテーブル内容を確認します。

https://github.com/datacharmer/test_db

## Amazon RDS for MySQLでの動作確認結果
<details><summary>動作確認結果（長文のため折りたたみ）</summary><div>

```
mysql> 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.02 sec)

mysql> use employees;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> show tables;
+----------------------+
| Tables_in_employees  |
+----------------------+
| current_dept_emp     |
| departments          |
| dept_emp             |
| dept_emp_latest_date |
| dept_manager         |
| employees            |
| salaries             |
| titles               |
+----------------------+
8 rows in set (0.00 sec)

mysql> select * from employees limit 30;
+--------+------------+------------+-------------+--------+------------+
| emp_no | birth_date | first_name | last_name   | gender | hire_date  |
+--------+------------+------------+-------------+--------+------------+
|  10001 | 1953-09-02 | Georgi     | Facello     | M      | 1986-06-26 |
|  10002 | 1964-06-02 | Bezalel    | Simmel      | F      | 1985-11-21 |
|  10003 | 1959-12-03 | Parto      | Bamford     | M      | 1986-08-28 |
|  10004 | 1954-05-01 | Chirstian  | Koblick     | M      | 1986-12-01 |
|  10005 | 1955-01-21 | Kyoichi    | Maliniak    | M      | 1989-09-12 |
|  10006 | 1953-04-20 | Anneke     | Preusig     | F      | 1989-06-02 |
|  10007 | 1957-05-23 | Tzvetan    | Zielinski   | F      | 1989-02-10 |
|  10008 | 1958-02-19 | Saniya     | Kalloufi    | M      | 1994-09-15 |
|  10009 | 1952-04-19 | Sumant     | Peac        | F      | 1985-02-18 |
|  10010 | 1963-06-01 | Duangkaew  | Piveteau    | F      | 1989-08-24 |
|  10011 | 1953-11-07 | Mary       | Sluis       | F      | 1990-01-22 |
|  10012 | 1960-10-04 | Patricio   | Bridgland   | M      | 1992-12-18 |
|  10013 | 1963-06-07 | Eberhardt  | Terkki      | M      | 1985-10-20 |
|  10014 | 1956-02-12 | Berni      | Genin       | M      | 1987-03-11 |
|  10015 | 1959-08-19 | Guoxiang   | Nooteboom   | M      | 1987-07-02 |
|  10016 | 1961-05-02 | Kazuhito   | Cappelletti | M      | 1995-01-27 |
|  10017 | 1958-07-06 | Cristinel  | Bouloucos   | F      | 1993-08-03 |
|  10018 | 1954-06-19 | Kazuhide   | Peha        | F      | 1987-04-03 |
|  10019 | 1953-01-23 | Lillian    | Haddadi     | M      | 1999-04-30 |
|  10020 | 1952-12-24 | Mayuko     | Warwick     | M      | 1991-01-26 |
|  10021 | 1960-02-20 | Ramzi      | Erde        | M      | 1988-02-10 |
|  10022 | 1952-07-08 | Shahaf     | Famili      | M      | 1995-08-22 |
|  10023 | 1953-09-29 | Bojan      | Montemayor  | F      | 1989-12-17 |
|  10024 | 1958-09-05 | Suzette    | Pettey      | F      | 1997-05-19 |
|  10025 | 1958-10-31 | Prasadram  | Heyers      | M      | 1987-08-17 |
|  10026 | 1953-04-03 | Yongqiao   | Berztiss    | M      | 1995-03-20 |
|  10027 | 1962-07-10 | Divier     | Reistad     | F      | 1989-07-07 |
|  10028 | 1963-11-26 | Domenick   | Tempesti    | M      | 1991-10-22 |
|  10029 | 1956-12-13 | Otmar      | Herbst      | M      | 1985-11-20 |
|  10030 | 1958-07-14 | Elvis      | Demeyer     | M      | 1994-02-17 |
+--------+------------+------------+-------------+--------+------------+
30 rows in set (0.00 sec)

mysql> 
mysql> select count(*) from employees group by gender;
+----------+
| count(*) |
+----------+
|   179973 |
|   120051 |
+----------+
2 rows in set (0.21 sec)

mysql> select gender,count(*) from employees group by gender;
+--------+----------+
| gender | count(*) |
+--------+----------+
| M      |   179973 |
| F      |   120051 |
+--------+----------+
2 rows in set (0.24 sec)

mysql> show columns from employees;
+------------+---------------+------+-----+---------+-------+
| Field      | Type          | Null | Key | Default | Extra |
+------------+---------------+------+-----+---------+-------+
| emp_no     | int           | NO   | PRI | NULL    |       |
| birth_date | date          | NO   |     | NULL    |       |
| first_name | varchar(14)   | NO   |     | NULL    |       |
| last_name  | varchar(16)   | NO   |     | NULL    |       |
| gender     | enum('M','F') | NO   |     | NULL    |       |
| hire_date  | date          | NO   |     | NULL    |       |
+------------+---------------+------+-----+---------+-------+
6 rows in set (0.02 sec)
```

</div></details>

# 移行フロー

1. OCIのObject Storageに、Dumpファイル保存用バケットを作成
1. AWSの作業用EC2に、OCI接続用設定を実施
1. AWSの作業用EC2に、MySQL Shellをインストール
1. MySQL Shellを利用して、RDS for MySQLのDumpファイルを作成
1. OCIのMDSを、Dumpファイルを用いて作成

# 1. OCI Object Storage: Dumpファイル保存用バケットを作成
Dumpファイル保存用のバケットを作成します。

OCIコンソール画面
→ 左上のハンバーガーメニュー
→ 「ストレージ」→ 「オブジェクト・ストレージとアーカイブ・ストレージ」→ 「バケット」を選択
「バケットの作成」を押下


| 項目名 | 設定 |
|:-----------|------------|
| バケット名 | （任意のバケット名） |
| デフォルト・ストレージ層 | 標準 |
| 暗号化 | Oracle管理キーを使用した暗号化 |

その他、デフォルトのままで作成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/3fa8adf9-3ccf-33c2-a32f-f9d2b3331a50.png)

# 2. AWSの作業用EC2に、OCI接続用設定を実施
EC2の作業ユーザーのホームディレクトリに、 **./.oci/config** ファイルを作成します。

# 2-1. OCIコンソールから、APIキーを作成
OCIコンソール画面
→ 「ユーザー設定」を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/e3da84b0-01bb-d721-8716-19d5a36e14e8.png)

左下、リソースの「APIキー」→「APIキーの追加」を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/e237f4eb-456f-c4a0-df9d-a085d86cbea1.png)

「秘密キーのダウンロード」を押下し、保存後に、「追加」を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/dfdd5590-27cc-1e75-3c85-57339fdb318b.png)

構成ファイルのプレビューの内容をコピーします。
後で利用しますので、テキストエディタ等に保存してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/b24dd44d-c6ae-2be7-1121-49d14309654b.png)

## 2-2. 作業用EC2に、OCI構成ファイルを作成
作業用EC2にログインします。

ログイン後に、Touchコマンドで./.oci/config作成します。
```
touch ~/.oci/config
```

作成した **.oci/config** を編集します。
```
[DEFAULT]
user=(2-1でコピーした内容)
fingerprint=(2-1でコピーした内容)
tenancy=(2-1でコピーした内容)
region=(2-1でコピーした内容)
key_file=oci_api_key.pem
```
## 2-2. 作業用EC2に、OCI APIキーをアップロード
手順2-1でダウンロードした秘密キーをアップロードします。
- ファイル名: **oci_api_key.pem**とします。
- 保存先フォルダ: **~/.oci/** 

```
$ ls -l ~/.oci/
total 12
-rw------- 1 ubuntu ubuntu  301 Oct  1 08:26 config
-rw------- 1 ubuntu ubuntu 1704 Oct  1 08:26 oci_api_key.pem
$ 
```

# 3. AWSの作業用EC2に、MySQL Shellをインストール
:::note info
既にインストール済の場合は手順4に進んでください。
:::

以下公式の手順を参照してインストールします。
https://dev.mysql.com/doc/mysql-shell/8.0/ja/mysql-shell-install-linux-quick.html

インストール出来ていることを確認
```
$ mysqlsh --version
mysqlsh   Ver 8.0.34 for Linux on x86_64 - for MySQL 8.0.34 (MySQL Community Server (GPL))
$ 
```

# 4. EC2にて、MySQL Shellを利用して、RDS for MySQLのDumpファイルを作成

## 4-1. EC2にて、MySQL Shellを実行

```
$ mysqlsh

MySQL Shell 8.0.34

Copyright (c) 2016, 2023, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.
```

## 4-2. MySQL Shellから、対象のRDS for MySQLのエンドポイントに接続

```
 \connect RDSのユーザ名@rdsのエンドポイント
```

途中でRDSユーザのパスワードを聞かれます。

```
 MySQL  JS > \connect xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com

Creating a session to 'xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com'
Please provide the password for 'xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com': ********
Save password for 'xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com'? [Y]es/[N]o/Ne[v]er (default No): Y
Fetching schema names for auto-completion... Press ^C to stop.
Your MySQL connection id is 319
Server version: 8.0.33 Source distribution
No default schema selected; type \use <schema> to set one.

 MySQL  xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306 ssl  JS > 
```

## 4-3. RDS for MySQLのDumpファイルをOCIのObject Storageに作成

以下のコマンドでDumpファイルを作成
```
> util.dumpInstance("outputurl", {osBucketName: "bucketname", osNamespace: "namespace", threads:1, ocimds: true, compatibility: ["strip_restricted_grants", "strip_definers"]})
```
| 項目 | 説明 |
|:-:|:-|
| outputurl | バケット直下に作成される親ディレクトリ名。任意でOK  |
| osBucketName | OCIで作成したObject Storageのバケット名を入力  |
| osNamespace | OCIテナントのNameSpace名を入力。OCIコンソールの右上人型アイコンのテナンシをクリックすることで、オブジェクト・ストレージ・ネームスペースが確認出来ます。  |
| threads | EC2のvCPU数を指定 |
| ocimds | このオプションを true に設定すると、MySQL Database Service との互換性のチェックおよび変更が可能になります。 |
| compatibility | MySQL Database Service との互換性のために指定された要件を適用します。  |
| strip_restricted_grants | MySQL Database Service によって制限されている特定の権限を GRANT ステートメントから削除して、ユーザーおよびそのロールにこれらの権限を付与できないようにします  |
| strip_definers | ビュー、ルーチン、イベントおよびトリガーから DEFINER 句を削除して、これらのオブジェクトがデフォルト定義者 (スキーマを起動するユーザー) で作成されるようにし、ビューおよびルーチンの SQL SECURITY 句を変更して、DEFINER のかわりに INVOKER を指定します。 |

項目の詳細は以下公式サイトを参照してください。
https://dev.mysql.com/doc/mysql-shell/8.0/ja/mysql-shell-utilities-dump-instance-schema.html

実行結果
```
 MySQL  xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306 ssl  JS > util.dumpInstance("outputurl", {osBucketName: "bucketname", osNamespace: "namespace", threads:1, ocimds: true, compatibility: ["strip_restricted_grants", "strip_definers"]})

NOTE: Backup lock is not available to the account 'admin'@'%' and DDL changes will not be blocked. The dump may fail with an error if schema changes are made while dumping.

(中略）

Dump duration: 00:00:05s                                                                      
Total duration: 00:00:05s                                                                     
Schemas dumped: 1                                                                             
Tables dumped: 6                                                                              
Uncompressed data size: 141.50 MB                                                             
Compressed data size: 36.82 MB                                                                
Compression ratio: 3.8                                                                        
Rows written: 3919015                                                                         
Bytes written: 36.82 MB                                                                       
Average uncompressed throughput: 25.41 MB/s                                                   
Average compressed throughput: 6.61 MB/s                                                      

MySQL  xxxxxx@xxxxxx.xxxxxxxxx.ap-northeast-1.rds.amazonaws.com:3306 ssl  JS >
```

OCIのバケットに作成された事を確認
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/0d05a2ae-869c-44ab-2c80-a60619395cdb.png)

# 5. OCIのMDSを、Dumpファイルを用いて作成
OCIコンソールから、新規MySQL HeatWave Database Serviceを作成する。

OCIコンソール画面
→ 左上のハンバーガーメニュー
→ 「データベース」→ 「MySQL HeatWave」→ 「DBシステム」を選択

「DBシステムの作成」を選択

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/c590f277-251a-3452-2ad1-9d948380991e.png)

作成メニューの下部、拡張オプションを表示。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/eb5d43c5-6a1b-9854-2316-f6504d048d52.png)

「データのインポート」タブを選択し、「既存のバケットに対するPAR URLを作成～」を選択。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/ececfece-410b-7a49-50d1-f6679d5de97d.png)

Dumpファイルが保存してある、バケットとDump取得時にurlとして指定した接頭辞を指定し、作成。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/f234af8f-6880-aad4-aeeb-a20a7180f6ab.png)

URLが入っている事を確認して作成をクリック。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/6bc76afc-7f20-462d-30c3-e472389c806e.png)

作成完了を待ちます。（Inportするデータ容量に時間が左右します。20分～ ）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/276568/7f243592-bd24-ccb0-12d3-8e200851595a.png)

------------------------


# 動作確認
MySQL HeatWave Database Serviceのエンドポイントにログインし、動作確認をします。

<details><summary>動作確認結果（長文のため折りたたみ）</summary><div>

```
mysql> 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| mysql_audit        |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

mysql> 
mysql> 
mysql> use employees;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> 
mysql> show tables;
+----------------------+
| Tables_in_employees  |
+----------------------+
| current_dept_emp     |
| departments          |
| dept_emp             |
| dept_emp_latest_date |
| dept_manager         |
| employees            |
| salaries             |
| titles               |
+----------------------+
8 rows in set (0.00 sec)

mysql> 
mysql> select * from employees limit 30;
+--------+------------+------------+-------------+--------+------------+
| emp_no | birth_date | first_name | last_name   | gender | hire_date  |
+--------+------------+------------+-------------+--------+------------+
|  10001 | 1953-09-02 | Georgi     | Facello     | M      | 1986-06-26 |
|  10002 | 1964-06-02 | Bezalel    | Simmel      | F      | 1985-11-21 |
|  10003 | 1959-12-03 | Parto      | Bamford     | M      | 1986-08-28 |
|  10004 | 1954-05-01 | Chirstian  | Koblick     | M      | 1986-12-01 |
|  10005 | 1955-01-21 | Kyoichi    | Maliniak    | M      | 1989-09-12 |
|  10006 | 1953-04-20 | Anneke     | Preusig     | F      | 1989-06-02 |
|  10007 | 1957-05-23 | Tzvetan    | Zielinski   | F      | 1989-02-10 |
|  10008 | 1958-02-19 | Saniya     | Kalloufi    | M      | 1994-09-15 |
|  10009 | 1952-04-19 | Sumant     | Peac        | F      | 1985-02-18 |
|  10010 | 1963-06-01 | Duangkaew  | Piveteau    | F      | 1989-08-24 |
|  10011 | 1953-11-07 | Mary       | Sluis       | F      | 1990-01-22 |
|  10012 | 1960-10-04 | Patricio   | Bridgland   | M      | 1992-12-18 |
|  10013 | 1963-06-07 | Eberhardt  | Terkki      | M      | 1985-10-20 |
|  10014 | 1956-02-12 | Berni      | Genin       | M      | 1987-03-11 |
|  10015 | 1959-08-19 | Guoxiang   | Nooteboom   | M      | 1987-07-02 |
|  10016 | 1961-05-02 | Kazuhito   | Cappelletti | M      | 1995-01-27 |
|  10017 | 1958-07-06 | Cristinel  | Bouloucos   | F      | 1993-08-03 |
|  10018 | 1954-06-19 | Kazuhide   | Peha        | F      | 1987-04-03 |
|  10019 | 1953-01-23 | Lillian    | Haddadi     | M      | 1999-04-30 |
|  10020 | 1952-12-24 | Mayuko     | Warwick     | M      | 1991-01-26 |
|  10021 | 1960-02-20 | Ramzi      | Erde        | M      | 1988-02-10 |
|  10022 | 1952-07-08 | Shahaf     | Famili      | M      | 1995-08-22 |
|  10023 | 1953-09-29 | Bojan      | Montemayor  | F      | 1989-12-17 |
|  10024 | 1958-09-05 | Suzette    | Pettey      | F      | 1997-05-19 |
|  10025 | 1958-10-31 | Prasadram  | Heyers      | M      | 1987-08-17 |
|  10026 | 1953-04-03 | Yongqiao   | Berztiss    | M      | 1995-03-20 |
|  10027 | 1962-07-10 | Divier     | Reistad     | F      | 1989-07-07 |
|  10028 | 1963-11-26 | Domenick   | Tempesti    | M      | 1991-10-22 |
|  10029 | 1956-12-13 | Otmar      | Herbst      | M      | 1985-11-20 |
|  10030 | 1958-07-14 | Elvis      | Demeyer     | M      | 1994-02-17 |
+--------+------------+------------+-------------+--------+------------+
30 rows in set (0.00 sec)

mysql> 
mysql> select gender,count(*) from employees group by gender;
+--------+----------+
| gender | count(*) |
+--------+----------+
| M      |   179973 |
| F      |   120051 |
+--------+----------+
2 rows in set (0.27 sec)

mysql> 
mysql> show columns from employees;
+------------+---------------+------+-----+---------+-------+
| Field      | Type          | Null | Key | Default | Extra |
+------------+---------------+------+-----+---------+-------+
| emp_no     | int           | NO   | PRI | NULL    |       |
| birth_date | date          | NO   |     | NULL    |       |
| first_name | varchar(14)   | NO   |     | NULL    |       |
| last_name  | varchar(16)   | NO   |     | NULL    |       |
| gender     | enum('M','F') | NO   |     | NULL    |       |
| hire_date  | date          | NO   |     | NULL    |       |
+------------+---------------+------+-----+---------+-------+
6 rows in set (0.00 sec)

mysql> 
mysql> exit
Bye
```

</div></details>




------------------------


