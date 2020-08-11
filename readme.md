# 目的
書籍「Kubernetes完全ガイド」を参考に、GKEを利用してk8sを入門してみた

# そもそもkubernetesとは？？
以下記事などを参考  
https://qiita.com/MahoTakara/items/85096f8b2632c802ab22

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
SDKをローカルPCにインストールすることで、GKE上のk8sクラスタの操作や定義ファイルの修正がターミナル上で簡単に実行できるようになります  
（GCP上のCloud Shellを使えば、ブラウザ上で上記操作を行うことも可能）


SDKインストールとセットアップ手順（win/mac）  
https://cloud.google.com/sdk/docs/quickstarts?hl=ja


# GKEクラスタ作成
## CLIでクラスタ作成
```
gcloud container clusters create gke-handson --zone=asia-northeast1-a --num-nodes=2
```

## クラスタ情報の連携
```
gcloud container clusters get-credentials  gke-handson --zone asia-northeast1-a
```

## ノード情報の確認
```
$ kubectl get nodes
NAME                                         STATUS   ROLES    AGE   VERSION
gke-gke-handson-default-pool-213ac691-2m80   Ready    <none>   64s   v1.15.12-gke.2
gke-gke-handson-default-pool-213ac691-5jh1   Ready    <none>   66s   v1.15.12-gke.2
```

## kubectl install（ローカルPC実行時のみ）
k8sではクラスタに対してのコマンドライン実行をクライアントツールであるkubectlを利用して行う。
※cloud shellで実行していた場合はデフォルトインストール済み。

kubectlインストールとセットアップ（win/mac）  
https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

## クラスタ作成時の注意事項
クラスタ作成時にip addressのquota制限に引っ掛かりデプロイできない場合がある。その時はquota制限を上げる

エラー例:
```
ERROR: (gcloud.container.clusters.create) ResponseError: code=403, message=Insufficient regional quota to satisfy request: resource "IN_USE_ADDRESSES": request requires '3.0' and is short '2.0'. project has a quota of '8.0' with '1.0' available
. View and manage quotas at https://console.cloud.google.com/iam-admin/quotas?usage=USED&project=handson-pj.
```
参考：

https://qiita.com/kkitase/items/83269f56c7259feede52

# 各ハンズオンの進め方
k8sの各定義ファイルを以下のディレクトリに格納しています。最終的に[loadbalancing]の定義ファイルを適用するとnginxコンテナがパブリック公開されます（随時更新中です）

以下ではディレクトリの内容をハンズオンの順番に説明していきます  
また、各ディレクトリ内にあるshellスクリプトを実行してリソースの作成削除が行えます  
*_create.sh: マニュフェストを元に各リソース作成を行う  
*_delete.sh: マニュフェストを元に作成したリソースの削除を行う

## 1_simple_deploy
k8sの基本となるpodの作成について。
nginx-containerとredis-containerの構築を行います

## 2_replica_sets
k8sのスケール概念である「ReplicaSet」を利用してpodのレプリカを複数作成します
定義ファイルに常に動いて欲しいpodの数を定義することで、k8sがそのpod状態を管理し続けます。  
ReplicaSetを利用することで、スケーリングによるpodの耐障害性の向上や冗長性の確保が可能です。

## 3_deployment
k8sコンテナのアップデート戦略について記述します。  
コンテナのバージョンアップについて、コンテナ再作成もしくはローリングアップデートといった信頼性の高いサービスロールアウトを管理する仕組み

## 4_cronjob
k8sコンテナで定期実行するjobを定義、管理する。（linuxのcronjobのようなもの）

## 5_loadbalancing
pod間の負荷分散や外部ネットワークとの接続点について定義する。  
GKEではGCLBと組みわせることにより、GKEの各ノードをバックエンドとして負荷分散や外部接続を提供する。（GCLBの管理も定義ファイルによって行う）

※1_simple_deployなどでpodを作成した後で実施ください。  
 5_create.shではLB機能によりサービスをインターネット公開します。外部からの制限はかけていないのでセキュリティに注意して実施ください。