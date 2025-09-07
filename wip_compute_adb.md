# OCI Hands-on: Compute (Windows) + Autonomous Database (Private Endpoint)

## 目的
- OCI上に **Windows Compute** と **Autonomous Database (ATP)** を構築  
- **Private Endpoint** を利用し、Windows から ADB へ接続確認  

---

## アーキテクチャ

![図: ハンズオン全体構成（Public SubnetのWindowsからPrivate SubnetのADBへ接続）](image-placeholder)

---

## 前提条件
- OCIテナンシ／コンパートメントが利用可能  
- ローカルPCから Windows への RDP が可能（Public IP で接続）  
- ブラウザで OCI コンソールにログイン可能（ADB作成権限あり）  

---

## 手順

### Step 1. VCN 作成
※本手順は、VCN作成済の場合は省略
1. コンソールのナビゲーションメニューから [ネットワーキング]→ [仮想クラウド・ネットワーク]を選択
2. 右ペインにて、**対象のコンパートメントが選ばれていること**を確認
3. 右ペインの「アクション」から、[VCNウィザードの起動]をクリック

<img width="40%" height="40%" alt="CleanShot 2025-09-07 at 16 04 50" src="https://github.com/user-attachments/assets/0c82cbb1-007f-4f4d-87a4-a5c9966c6acf" />



4. VCNウィザードで以下を一括作成
   
|項目|設定値|備考|
|---|---|---|
|接続タイプ|インターネット接続性を持つVCNの作成||
|VCN名|handson-vcn|任意の名前でOK|
|VCN IPv4 CIDRブロック|10.0.0.0/16||
|Public Subnet|10.0.1.0/24|（Windows Server用）|
|Private Subnet|10.0.2.0/24|（ADB Private Endpoint 用）|

---

### Step 2. Windows Compute 作成 & RDP
1. コンソールのナビゲーションメニューから [コンピュート]→ [インスタンス]を選択
2. 右ペインにて、**対象のコンパートメントが選ばれていること**を確認
3. 右ペインの「アクション」から、[インスタンスの作成]をクリック

|項目|設定値|備考|
|---|---|---|
|名前|handson-win|任意の名前でOK|
|イメージの変更|Windows Server 2022 Standard Core|追加のライセンス料金|
|シェイプの変更|||
|シェイプ名|VM.Standard.E4.Flex||
|OCPUの数|2||
|メモリー量|32||
|ネットワーキング|先ほど作成したVCNの`パブリックサブネット`||

※以下確認途中

2. 起動後、**パブリックIP へ RDP** で接続し、管理者パスワードを設定  

![図: Computeインスタンス作成（Windows Server 2022 選択）](image-placeholder)

---

### Step 3. Autonomous Database（ATP, Private Endpoint）作成
1. **Autonomous Database → Create**  
   - DB名: `handson-atp`  
   - ワークロード: *Transaction Processing*  
   - OCPU: 2（Auto Scaling 有効のまま）  
   - **Subnet: Private Subnet** を選択  
   - **Private Endpoint: 有効** を選択  
   - ライセンス: 環境に応じて（BYOL/License Included）  
2. 作成完了まで待機（数分）  

![図: ADB作成（Private Endpoint 有効化の箇所を強調）](image-placeholder)

---

### Step 4. SQL Developer のインストール（Windows）
1. Windows 上のブラウザで **SQL Developer ダウンロードページ** を開く  
2. **Windows 64-bit (JDK 17 Included)** をダウンロード  
3. ZIP を任意のフォルダに展開（例: `C:\tools\sqldeveloper`）  
4. `sqldeveloper.exe` を実行して起動  
   - 初回起動で SmartScreen が出たら「詳細情報→実行」を選択  

![図: SQL DeveloperのZIP展開と初回起動](image-placeholder)

---

### Step 5. Wallet（Client Credentials）のダウンロード
1. OCI コンソールで **作成した ADB の詳細ページ**を開く  
2. **DB 接続（Database connection）** をクリック  
3. **Download Wallet** をクリック  
4. ダイアログで **Wallet パスワード**を入力して **Download**  
   - 例: ダウンロードファイル名 `Wallet_handson-atp.zip`  

![図: ADB → DB Connection → Download Wallet 画面（入力箇所を強調）](image-placeholder)

---

### Step 6. Wallet を Windows Compute にコピー（クリップボード経由）
1. RDP接続時に **クリップボード共有を有効化**  
   - 「ローカルリソース」タブ → 「クリップボード」にチェック  
2. 自分のPCでダウンロードした Wallet ZIP をコピー（Ctrl+C）  
3. RDP で接続している Windows 内で貼り付け（Ctrl+V）  
   - 例: `C:\wallet_handson\Wallet_handson-atp.zip` に保存  
4. ZIP を右クリック → 「すべて展開」で展開  
   - 中に `tnsnames.ora / sqlnet.ora / cwallet.sso / ewallet.p12` などが含まれる  

![図: Windows RDP にコピーして展開した Wallet フォルダ](image-placeholder)

#### 注意点
- RDP接続時にクリップボード共有が無効だとコピーできません  
- Wallet ZIP は展開前に中身を変更しない  
- 展開先フォルダは英数字のみで空白や日本語を含まないことを推奨  

#### トラブルシュート
| 現象 | 確認/対処 |
|-------|------------|
| RDP内で貼り付けできない | RDP接続オプションで「クリップボード共有」が有効か確認 |
| Wallet ZIP が開けない/破損 | 自分のPCでダウンロードしたZIPが正常か確認し、再ダウンロード |
| 展開後にファイルが足りない | ZIPを展開せずコピーしていないか確認。必ず「すべて展開」で解凍 |

---

### Step 7. SQL Developer で接続
1. SQL Developer を起動 → 左の *Connections* で **New**  
2. 入力例（Cloud Wallet 方式）  
   - **Connection Name**: `handson`  
   - **Username**: `ADMIN`  
   - **Password**: ADB作成時に設定したもの  
   - **Configuration File**: `C:\wallet_handson\Wallet_handson-atp.zip`  
3. **Test → Save → Connect**  

![図: SQL Developer 新規接続（Cloud Wallet ZIP 指定）](image-placeholder)

#### 注意点
- Cloud Wallet方式は ZIP を展開不要  
- 展開して使う場合は `TNS_ADMIN` に展開フォルダを指定する  
- SQL Developer 起動後に **Test** を必ず実行  

#### トラブルシュート
| 現象 | 確認/対処 |
|-------|------------|
| ORA-12154 / TNS:could not resolve | TNS_ADMIN の設定や tnsnames.ora の存在を確認 |
| ORA-29024 / invalid certificate | Wallet 内の証明書を改変していないか確認 |
| 接続できない | Cloud Wallet 方式で直接 ZIP 指定 or 展開フォルダの内容確認 |

---

### Step 8. 動作確認（SQL 実行）
```sql
SELECT sysdate FROM dual;
