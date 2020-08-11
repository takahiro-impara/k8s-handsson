# 目的
書籍「Kubernetes完全ガイド」を参考に、GKEを利用してk8sを入門してみた


# 環境準備
## 前提
本ハンズオンはGoogle Cloud Platform(GCP)を利用します。

GCPには300ドルまで無料試用できるクレジットがあるので、まだGCPのアカウントを持っていない方は以下のURLを参考にアカウント取得から初めてください。

https://cloud.google.com/free?hl=ja

## GCPアカウント準備
### プロジェクト作成
GCPでは、プロジェクトと呼ばれる単位で課金アカウントを分けて、その中で各種サービスを実行します。（Azureのリソースグループのようなもの）

プロジェクト作成手順  
https://www.magellanic-clouds.com/blocks/guide/create-gcp-project/

### セキュリティ
アカウント情報の漏洩を防ぐため、2段階認証の設定を推奨します
https://support.google.com/accounts/answer/6103523

## GCP SDKをローカルPCにインストール
SDKをローカルPCにインストールすることで、GKE上のk8sクラスタの操作や定義ファイルの修正がターミナル上で簡単に実行できるようになります。（GCP上のCloud Shellを使えば、ブラウザ上で上記操作を行うことも可能）

SDKインストールとセットアップ手順（win/mac）
https://cloud.google.com/sdk/docs/quickstarts?hl=ja

## GKEクラスタ作成
### CLIでクラスタ作成
```
gcloud container clusters create gke-handson2 --zone=asia-northeast1-a --num-nodes=3
```

### kubectl install（ローカルPC実行時のみ）
k8sではクラスタに対してのコマンドライン実行をクライアントツールであるkubectlを利用して行う。
※cloud shellで実行していた場合はデフォルトインストール済み。

kubectlインストールとセットアップ（win/mac）
https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

### クラスタ作成時の注意事項
クラスタ作成時にip addressのquota制限に引っ掛かりデプロイできない場合がある。その時はquota制限を上げる

エラー例:
```
ERROR: (gcloud.container.clusters.create) ResponseError: code=403, message=Insufficient regional quota to satisfy request: resource "IN_USE_ADDRESSES": request requires '3.0' and is short '2.0'. project has a quota of '8.0' with '1.0' available
. View and manage quotas at https://console.cloud.google.com/iam-admin/quotas?usage=USED&project=handson-pj.
```
参考：

https://qiita.com/kkitase/items/83269f56c7259feede52