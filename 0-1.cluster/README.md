まず初めにチュートリアルを行うためのGKEクラスタを準備します。

公式ドキュメント: https://cloud.google.com/kubernetes-engine/docs/how-to/creating-an-autopilot-cluster

## プロジェクトの準備

### プロジェクトの作成

最後にまとめて掃除できるように、また他のリソースと干渉しないように、チュートリアルを進めるためのプロジェクトを作成します。

例えば `m3-ai-team-k8s-tutorial-(ランダムな数字)` という名前でプロジェクトを作成します。

```console
export PROJECT_ID=m3-ai-team-k8s-tutorial-$RANDOM
```

```console
gcloud projects create $PROJECT_ID
```

### Billing Account の設定

GKEを使用するためには、プロジェクトに対して Billing Account を設定する必要があります。

有効になっているか確認

```console
gcloud billing projects describe $PROJECT_ID
```

```
billingAccountName: ''
billingEnabled: false
name: projects/m3-ai-team-k8s-tutorial-xxxxx/billingInfo
projectId: m3-ai-team-k8s-tutorial-xxxxx
```

作成直後であれば `billingEnabled: false` になっているはずなので、Billing Account を設定します。

Billing Account の一覧を取得して、設定したいACCOUNT_IDを確認し、プロジェクトに紐づけます。

```console
gcloud billing accounts list
```

```console
ACCOUNT_ID=xxxxxx-xxxxxx-xxxxxx
```

```console
gcloud billing projects link $PROJECT_ID --billing-account=$ACCOUNT_ID
```

これで `billingEnabled: true` になっているはずです。

```console
gcloud billing projects describe $PROJECT_ID
```

```
billingAccountName: billingAccounts/xxxxxx-xxxxxx-xxxxxx
billingEnabled: true
name: projects/m3-ai-team-k8s-tutorial-xxxxx/billingInfo
projectId: m3-ai-team-k8s-tutorial-xxxxx
```

## GKEクラスタの準備

### API の有効化

まずはAPIが有効になっているか確認します。

```console
gcloud services list --project=$PROJECT_ID | grep container.googleapis.
com
```

プロジェクト作成直後では `container.googleapis.com` が表示されず無効状態のはずなので有効化します。

```console
gcloud services enable container.googleapis.com --project=$PROJECT_ID
```

`container.googleapis.com` が有効になっているか確認します。

```console
gcloud services list --project=$PROJECT_ID | grep container.googleapis.com
```

```
container.googleapis.com            Kubernetes Engine API
```

### GKEクラスタの作成

いよいよGKEクラスタを作成します。今回はAutopilotモードで作成します。

```console
export CLUSTER_NAME=tutorial-cluster
export LOCATION=asia-northeast1
```

```console
gcloud container clusters create-auto CLUSTER_NAME \
    --location=$LOCATION \
    --project=$PROJECT_ID
```

しばらく時間がかかりますが、作成が完了すると以下のような出力が表示されます。

```
NAME              LOCATION         MASTER_VERSION      MASTER_IP     MACHINE_TYPE  NODE_VERSION        NUM_NODES  STATUS
tutorial-cluster  asia-northeast1  1.30.6-gke.1125000  34.146.50.13  e2-small      1.30.6-gke.1125000  3          RUNNING
```

※ 次のような警告が出ますが、後ほどインストールするので今は気にしなくて大丈夫です。
```
CRITICAL: ACTION REQUIRED: gke-gcloud-auth-plugin, which is needed for continued use of kubectl, was not found or is not executable. Install gke-gcloud-auth-plugin for use with kubectl by following https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#install_plugin
```

### クラスタへの接続

クラスタに対してデプロイなどの操作をするために認証を設定します。

```console
gcloud container clusters get-credentials $CLUSTER_NAME \
    --location=$LOCATION \
    --project=$PROJECT_ID
```

操作するためのCLI `kubectl` と、警告に出ていた認証用のプラグインをインストールします。

```console
gcloud components install kubectl gke-gcloud-auth-plugin
```

これでクラスタにアクセスできるようになりました。Namespace 一覧を表示してみましょう。

```console
kubectl get namespaces
```

```
NAME                       STATUS   AGE
default                    Active   9m47s
gke-gmp-system             Active   8m36s
gke-managed-cim            Active   9m19s
gke-managed-filestorecsi   Active   9m9s
gke-managed-system         Active   8m54s
gmp-public                 Active   8m36s
kube-node-lease            Active   9m48s
kube-public                Active   9m48s
kube-system                Active   9m48s
```

Namespaceが表示されていれば成功です！
