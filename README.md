# OCIハンズオンについて

Oracle Cloud Infrastructure (OCI) の主要サービスを実践的に学習できるハンズオン集です。ネットワーク監視、セキュリティ、データベース移行・レプリケーション、プライベート接続など、実務で重要な分野をカバーしています。

## ハンズオンサマリ

| ハンズオン | 難易度 | 所要時間 | 主な学習内容 |
|------------|--------|----------|--------------|
| VTAP編 | ⭐⭐ | 60分 | ネットワーク監視、パケット分析 |
| Cloud Guard編 | ⭐ | 30分 | セキュリティ監視、アラート設定 |
| HeatWave Migration編 | ⭐⭐⭐ | 90分 | データベース移行、MySQL Shell |
| HeatWave Replication編 | ⭐⭐⭐⭐ | 120分 | レプリケーション、マルチクラウド |
| Compute + ADB編 | ⭐⭐ | 75分 | プライベート接続、Wallet設定 |


## ハンズオン一覧

### 1. VTAP編 (`VTAP.md`)
**ネットワーク監視・パケット分析**
- VTAP（Virtual Test Access Point）環境の構築
- ping(ICMP)のキャプチャ確認
- ネットワークトラフィックの監視・分析手法

### 2. Cloud Guard編 (`CloudGuard.md`)
**セキュリティ監視・脅威検知**
- Cloud Guardディテクタの動作確認
- セキュリティアラートのメール通知設定
- Object Storage公開設定の検知デモ

### 3. HeatWave Migration編 (`HeatWave_migration.md`)
**データベース移行**
- MySQL Shellを利用したDumpファイル作成
- AWS RDS for MySQLからOCI MySQL HeatWave Database Serviceへの移行
- Object Storageを経由した大容量データ移行

### 4. HeatWave Replication編 (`HeatWave_replication.md`)
**データベースレプリケーション**
- AWS RDS for MySQLからHeatWave MySQLへのリアルタイムレプリケーション
- AWSとOCI間でのVPN接続環境の構築
- GTIDを使用したMySQL レプリケーション設定

### 5. Compute + Autonomous Database編 (`wip_compute_adb.md`) 🚧
**プライベートデータベース接続**
- Windows ComputeとAutonomous Database (ATP)の構築
- Private Endpointを利用した安全な接続
- SQL Developerでの接続確認とWallet設定

## 学習目標

このハンズオン集を通じて、以下のスキルを習得できます：

- **ネットワーク監視**: VTAPを使ったパケットキャプチャと分析
- **セキュリティ運用**: Cloud Guardによる脅威検知とアラート設定
- **データベース移行**: 実際のワークロードを想定した移行手順
- **レプリケーション**: マルチクラウド環境でのデータ同期
- **プライベート接続**: セキュアなデータベース接続の構築

## 前提条件

### 共通要件
- OCIテナンシへのアクセス権限
- 適切なIAMポリシーの設定
- VCN（仮想クラウドネットワーク）の基本知識

### ハンズオン別要件
- **VTAP編**: Compute インスタンス作成権限、ネットワーク設定権限
- **Cloud Guard編**: Cloud Guard管理権限、Notification設定権限
- **HeatWave Migration編**: AWS環境へのアクセス、MySQL Shell利用経験
- **HeatWave Replication編**: AWS/OCIマルチクラウド環境、VPN設定権限
- **Compute + ADB編**: Windows Server操作経験、SQL Developer利用経験

## 開始方法

1. **学習したいハンズオンを選択**
   - 各`.md`ファイルに詳細な手順が記載されています
   - 順番に実施することで段階的にスキルアップできます

2. **前提条件の確認**
   - 必要な権限とリソースが揃っていることを確認してください

## コントリビューション

改善提案やバグ報告は、Issueまたはプルリクエストでお寄せください。

