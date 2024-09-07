# OCIハンズオン-VTAP編

1. ハンズオン-作業イメージ&前提
2. サーバインスタンス作成&ping疎通確認
3. バックエンドインスタンスの作成
4. NLBの作成
5. VTAPの作成
6. 確認

## 1. ハンズオン-前提と構成イメージ
- VCNの作成が完了しており、そのVCN内にデフォルトのプライベート・サブネットとパブリック・サブネットが作成されていること
- パブリック・サブネットにコンピュート・インスタンスの作成が完了していること
- パブリック・サブネットにSSH接続が可能なこと

※※※ここに、前提のイメージを入れる※※※
（クライアント-----SSH------OCI-VCN-PublicSubnet-Compute

下記がこのハンズオンで作成できる構成イメージです。
![image](https://github.com/user-attachments/assets/21dafe69-7513-4374-9eb8-55c5d75dbb60)

## 2. サーバインスタンス作成&ping疎通確認
プライベートサブネットにすでにインスタンスがある前提

（すでに自動割り当てされてるプライベートIPアドレスを要確認）プライベートIPアドレスの手動割当 10.0.1.2

拡張オプションの表示は不要

セキュリティリストのpingポートを開ける ← いったんなし

## 3. バックエンドインスタンスの作成

手順と画像は2−1のまんまでOK
同じ設定の49152を作る、をもうちょっと丁寧に書く

ncatコマンドとは
ネットワークの疎通確認コマンド

## 4. NLBの作成
orasejapanはMAX NLB制限にひっかかって断念。

Flexible Network Load Balancer Count	max-nlb-flexible-count の制限解除リクエスト必須

## 5. VTAPの作成

基本的にチュートリアルの通りでOK
要所要所で画像いれないとわけわかんなくなるので作る
最初にVTAPについて詳細を解説する。
解説元はSpeakerdeckで
https://speakerdeck.com/ocise/oci-jia-xiang-tesutoakusesupointo-vtap-gai-yao



## 6. 確認
デモにする

## 確認
このコマンドの意味

（ターゲット・インスタンスに送信されるヘルスチェックのリクエストを受け入れ、レスポンスを返すため、ここではncatコマンドを用います。まずは以下のコマンドでnmap-ncatパッケージをインストールします）
ncコマンドとは

–l
リモートホストへの接続を開始せずに、着信する接続を待機します。

–u
デフォルトのオプションである TCP の代わりに UDP を使用します。

–i interval
テキスト行の送受信間隔 (interval) の遅延時間を指定します。

echoをつけて、responseを受け取ったときの動作とするサーバにする。
responseは、NLBで指定している。

(UDPサーバを建て、NLBのヘルスチェックのリクエストを待機します)
 while true; do (echo "response") | nc -lu 49152 -i 1; done > /dev/null 2>&1 &

(サーバ・インスタンスからミラーリングされて流れてくるパケットを待ち受けます)
 sudo tcpdump src host 10.0.1.2 -vv -i ens3

> src host HOST	パケットの送信元ホストがHOSTであれば真
> -vv	-vよりも詳細に出力する（NFS応答パケットの追加フィールドなども表示される）
> -i INTERFACE	ネットワーク・インターフェースINTERFACEを監視する
> linuxのデフォルトのNIC名はens3

