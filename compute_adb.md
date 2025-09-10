# OCI Hands-on: Compute (Windows) + Autonomous Database (Private Endpoint)

## 目的
- OCI上に **Windows Compute** と **Autonomous Database (ATP)** を構築  
- **Private Endpoint** を利用し、Windows から ADB へ接続確認  

---

## アーキテクチャ

<img width="100%" height="100%" alt="CleanShot 2025-09-09 at 09 22 59@2x" src="https://github.com/user-attachments/assets/9e2abd15-45a9-4b38-b46c-9bdd7c9637f7" />

---

## 前提条件
- OCIテナンシ／コンパートメントが利用可能  
- ローカルPCから Windows への RDP が可能（Public IP で接続）  
- ブラウザで OCI コンソールにログイン可能（ADB作成権限あり）  

---

## 手順

### Step 1. VCN 作成
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


5. 作成完了後、作成したVCNを表示

6. パブリックサブネットに、RDPのポートを登録
[仮想クラウド・ネットワーク]
→ [handon-vcn]※今回作成したVCN
→ [セキュリティ]タブ
→ [Default Security List for handson-vcn]※ウィザードで作成された、Publicサブネット用のセキュリティリスト
→ [セキュリティ・ルール]
→ [イングレス・ルールの追加]

項目|設定値|備考|
|---|---|---|
|ソースCIDR|0.0.0.0/0|RDP接続元のIPアドレス、要件によって絞ってください|
|宛先ポート範囲|3389||

<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 09 29 42" src="https://github.com/user-attachments/assets/32ffea30-4de3-4bde-83a0-2aedf788c941" />
<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 09 31 01" src="https://github.com/user-attachments/assets/aefeb07f-601a-431d-98e2-7b58d3d7dc1b" />
<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 09 38 02" src="https://github.com/user-attachments/assets/639c4bd4-a018-46de-890a-f5c0bd7ea289" />

6. プライベートサブネットに、ATPへの接続用のポートを登録
[仮想クラウド・ネットワーク]
→ [handon-vcn]※今回作成したVCN
→ [セキュリティ]タブ
→ [Default Security List for handson-vcn]※ウィザードで作成された、Publicサブネット用のセキュリティリスト
→ [セキュリティ・ルール]
→ [イングレス・ルールの追加]

項目|設定値|備考|
|---|---|---|
|ソースCIDR|10.0.0.0/16|RDP接続元のIPアドレス、要件によって絞ってください|
|宛先ポート範囲|1521||

<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 18 47 33" src="https://github.com/user-attachments/assets/98bada82-8096-4da8-9af5-6a711b760257" />
<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 09 31 01" src="https://github.com/user-attachments/assets/aefeb07f-601a-431d-98e2-7b58d3d7dc1b" />
<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 18 49 40" src="https://github.com/user-attachments/assets/0456577f-fe9d-415f-829c-ab29c3a424b3" />

---

### Step 2. Windows Compute 作成 & RDP
1. コンソールのナビゲーションメニューから [コンピュート]→ [インスタンス]を選択
2. 右ペインにて、**対象のコンパートメントが選ばれていること**を確認
3. 右ペインの「アクション」から、[インスタンスの作成]をクリック

|項目|設定値|備考|
|---|---|---|
|名前|handson-win|任意の名前でOK|
|イメージの変更|Windows Server 2022 Standard|追加のライセンス料金|
|シェイプの変更|||
|シェイプ名|VM.Standard.E4.Flex||
|OCPUの数|2||
|メモリー量|32||
|ネットワーキング|先ほど作成したVCNの`パブリックサブネット`||


2. 起動後、インスタンス・アクセス部分を確認し、**パブリックIP へ RDP** で接続
   
ユーザー名と初期パスワードもインスタンス・アクセス部分を確認

<img width="50%" height="50%" alt="CleanShot 2025-09-08 at 09 40 25" src="https://github.com/user-attachments/assets/83c36a68-6b64-41b2-8ae2-6e79052a4400" />

最初に、パスワードの変更が求められますので、OKを押してパスワード変更してください
※英語キーボードと日本語キーボードで、記号の違いがある場合がありますのでご注意ください

<img width="50%" height="50%" alt="CleanShot 2025-09-08 at 10 42 27" src="https://github.com/user-attachments/assets/72e5f508-67c4-4006-a8cd-b81df1fb199e" />

<img width="50%" height="50%" alt="CleanShot 2025-09-08 at 10 47 20" src="https://github.com/user-attachments/assets/5b69c9a5-81b1-4dde-8720-8eadf8ba444f" />

---

### Step 3. Autonomous Database（ATP, Private Endpoint）作成
1. コンソールのナビゲーションメニューから [Oracle Database]→ [Autonomous Database]を選択
2. 右ペインにて、**対象のコンパートメントが選ばれていること**を確認
3. 右ペインのAutonomous Databaseの作成]をクリック

|項目|設定値|備考|
|---|---|---|
|表示名|handson-atp|任意の名前でOK|
|データベース名|handson-atp|任意の名前でOK|
|ワークロード・タイプ|トランザクション処理||
|データベース・バージョンの選択|19c or 23ai|AIサービスを試すのであれば23ai|
|ECPU数|4|必要に応じて|
|ストレージ|1024GB||
|ADMINパスワード|任意に設定||
|ネットワーク・アクセス|||
|アクセス・タイプ|プライベート・エンドポイント・アクセスのみ|
|仮想クラウド・ネットワーク|handson-vcn|Step1で作成したVCN|
|サブネット|プライベート・サブネット|Step1で作成したプライベートサブネット|

   
  
2. 作成完了まで待機（数分）  

---

### Step 4. SQL Developer のインストール（Windows）
1. Windows 上のブラウザで **SQL Developer ダウンロードページ** を開く
   https://www.oracle.com/jp/database/sqldeveloper/technologies/download/
2. **Windows 64-bit (JDK 17 Included)** をダウンロード

<img width="50%" height="50%" alt="CleanShot 2025-09-08 at 10 52 51" src="https://github.com/user-attachments/assets/41e4b96a-acd1-4bd6-a69c-0bb9b65a3c50" />
     
3. Zipファイル解答後、`sqldeveloper.exe` を実行して起動  
   - 初回起動で SmartScreen が出たら「詳細情報→実行」を選択

 <img width="50%" height="50%" alt="CleanShot 2025-09-08 at 14 48 46" src="https://github.com/user-attachments/assets/3eac6d08-45f9-4a0b-b2d6-e38e0f99df28" />

- インポート・プリファレンスの確認 → いいえ
- Oracle使用状況トラッキング → OK

---

### Step 5. Wallet（Client Credentials）のダウンロード
1. OCI コンソールで **作成した ADB の詳細ページ**を開く  
2. **DB 接続（Database connection）** をクリック  
3. **Download Wallet** をクリック  
4. ダイアログで **Wallet パスワード**を入力して **Download**  
   - 例: ダウンロードファイル名 `Wallet_handson-atp.zip`  

---

### Step 6. Wallet を Windows Compute にコピー（クリップボード経由）
1. コンソールのナビゲーションメニューから [Oracle Database]→ [Autonomous Database]を選択
2. 作成したATPを選択
3. コンソール上部の[データベース接続]をクリック
4. `インスタンス・ウォレット`が選択されていることを確認し、[ウォレットのダウンロード]をクリック
5. 任意のパスワードを設定し、ダウンロード
6. RDP接続時に **クリップボード共有を有効化**  
7. 自分のPCでダウンロードした Wallet ZIP をコピー（Ctrl+C）  
8. RDP で接続している Windows 内で貼り付け（Ctrl+V）  
   - 例: `C:\wallet_handson\Wallet_handson-atp.zip` に保存  

#### トラブルシュート
| 現象 | 確認/対処 |
|-------|------------|
| RDP内で貼り付けできない | RDP接続オプションで「クリップボード共有」が有効か確認 |
| Wallet ZIP が開けない/破損 | 自分のPCでダウンロードしたZIPが正常か確認し、再ダウンロード |

---

### Step 7. SQL Developer で接続
1. SQL Developer を起動 → 左の *接続* で **+ボタンをクリック**  
2. 入力例（Cloud Wallet 方式）

|項目|設定値|備考|
|---|---|---|
|Name|handson|任意の名前でOK|
|データベースのタイプ|Oracle||
|ユーザー名|Ademin||
|パスワード|ATP作成時に設定したパスワード||
|接続タイプ|クラウド・ウォレット||
|構成ファイル|参照にて、対象のウォレットファイルを指定||
|ワークロード・タイプ|トランザクション処理||

3. [テスト]をクリック
4. テストが成功したら、[保存]をクリック
5. [接続]をクリック

<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 19 07 20" src="https://github.com/user-attachments/assets/d6c5538b-e3db-4575-bdb9-375b9591480b" />

#### 注意点
- SQL Developer 起動後に **Test** を必ず実行  

---

### Step 8. 動作確認（SQL 実行）
左ペインにSQLを入力してみて、三角矢印で実行し、結果が返ってくることを確認
```sql
SELECT sysdate FROM dual;
```

<img width="80%" height="80%" alt="CleanShot 2025-09-08 at 19 11 28" src="https://github.com/user-attachments/assets/6f9176a3-f7d7-488d-aa5c-7bffcd224538" />
