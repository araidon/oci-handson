[English version is available below](#english-version)

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
| ブートボリュームバックアップ編 | ⭐⭐ | 45分 | ブートボリュームバックアップ、インスタンス復元 |


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

### 5. Compute + Autonomous Database編 (`compute_adb.md`)
**プライベートデータベース接続**
- Windows ComputeとAutonomous Database (ATP)の構築
- Private Endpointを利用した安全な接続
- SQL Developerでの接続確認とWallet設定

### 6. ブートボリュームバックアップ編 (`boot_volume_backup.md`)
**インスタンス復元・クローン**
- Computeインスタンスのブートボリュームバックアップ作成
- バックアップからのブートボリュームリストア
- リストアしたブートボリュームから新規インスタンス作成

## 学習目標

このハンズオン集を通じて、以下のスキルを習得できます：

- **ネットワーク監視**: VTAPを使ったパケットキャプチャと分析
- **セキュリティ運用**: Cloud Guardによる脅威検知とアラート設定
- **データベース移行**: 実際のワークロードを想定した移行手順
- **レプリケーション**: マルチクラウド環境でのデータ同期
- **プライベート接続**: セキュアなデータベース接続の構築
- **バックアップ・リストア**: ブートボリュームバックアップによるインスタンス復元

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
- **ブートボリュームバックアップ編**: 稼働中のComputeインスタンス、SSHキーペア

## 開始方法

1. **学習したいハンズオンを選択**
   - 各`.md`ファイルに詳細な手順が記載されています
   - 順番に実施することで段階的にスキルアップできます

2. **前提条件の確認**
   - 必要な権限とリソースが揃っていることを確認してください

## コントリビューション

改善提案やバグ報告は、Issueまたはプルリクエストでお寄せください。

---

# English Version

## About OCI Hands-on Collection

This is a comprehensive hands-on collection for learning Oracle Cloud Infrastructure (OCI) key services practically. It covers important areas for real-world implementations including network monitoring, security, database migration/replication, and private connectivity.

## Hands-on Summary

| Hands-on | Difficulty | Duration | Key Learning Topics |
|----------|------------|----------|---------------------|
| VTAP | ⭐⭐ | 60 min | Network monitoring, packet analysis |
| Cloud Guard | ⭐ | 30 min | Security monitoring, alert configuration |
| HeatWave Migration | ⭐⭐⭐ | 90 min | Database migration, MySQL Shell |
| HeatWave Replication | ⭐⭐⭐⭐ | 120 min | Replication, multi-cloud |
| Compute + ADB | ⭐⭐ | 75 min | Private connectivity, Wallet configuration |
| Boot Volume Backup | ⭐⭐ | 45 min | Boot volume backup, instance restoration |

## Hands-on List

### 1. VTAP (`VTAP.md`)
**Network Monitoring & Packet Analysis**
- Building VTAP (Virtual Test Access Point) environment
- Capturing and verifying ping (ICMP) traffic
- Network traffic monitoring and analysis techniques

### 2. Cloud Guard (`CloudGuard.md`)
**Security Monitoring & Threat Detection**
- Cloud Guard detector operation verification
- Security alert email notification setup
- Object Storage public access detection demo

### 3. HeatWave Migration (`HeatWave_migration.md`)
**Database Migration**
- Creating dump files using MySQL Shell
- Migration from AWS RDS for MySQL to OCI MySQL HeatWave Database Service
- Large-scale data migration via Object Storage

### 4. HeatWave Replication (`HeatWave_replication.md`)
**Database Replication**
- Real-time replication from AWS RDS for MySQL to HeatWave MySQL
- Building VPN connection environment between AWS and OCI
- MySQL replication configuration using GTID

### 5. Compute + Autonomous Database (`compute_adb.md`)
**Private Database Connectivity**
- Building Windows Compute and Autonomous Database (ATP)
- Secure connectivity using Private Endpoint
- Connection verification and Wallet configuration with SQL Developer

### 6. Boot Volume Backup (`boot_volume_backup.md`)
**Instance Restoration & Cloning**
- Creating boot volume backup of Compute instance
- Restoring boot volume from backup
- Creating new instance from restored boot volume

## Learning Objectives

Through this hands-on collection, you can acquire the following skills:

- **Network Monitoring**: Packet capture and analysis using VTAP
- **Security Operations**: Threat detection and alert configuration with Cloud Guard
- **Database Migration**: Migration procedures for actual workloads
- **Replication**: Data synchronization in multi-cloud environments
- **Private Connectivity**: Building secure database connections
- **Backup & Restore**: Instance restoration using boot volume backup

## Prerequisites

### Common Requirements
- Access permissions to OCI tenancy
- Appropriate IAM policy configuration
- Basic knowledge of VCN (Virtual Cloud Network)

### Hands-on Specific Requirements
- **VTAP**: Compute instance creation permissions, network configuration permissions
- **Cloud Guard**: Cloud Guard management permissions, Notification configuration permissions
- **HeatWave Migration**: AWS environment access, MySQL Shell usage experience
- **HeatWave Replication**: AWS/OCI multi-cloud environment, VPN configuration permissions
- **Compute + ADB**: Windows Server operation experience, SQL Developer usage experience
- **Boot Volume Backup**: Running Compute instance, SSH key pair

## Getting Started

1. **Select the hands-on you want to learn**
   - Detailed procedures are documented in each `.md` file
   - Progressive skill development by following them in order

2. **Verify prerequisites**
   - Ensure you have the necessary permissions and resources

## Contribution

Please submit improvement suggestions or bug reports via Issues or Pull Requests.

---

**Happy Learning with OCI! 🎉**
