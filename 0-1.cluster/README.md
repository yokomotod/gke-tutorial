本家: https://cloud.google.com/kubernetes-engine/docs/how-to/creating-an-autopilot-cluster

# Project の作成

```console
export PROJECT_ID=m3-ai-team-k8s-tutorial-xxxxx
```

```console
gcloud projects create $PROJECT_ID
```

## Billing Account の設定

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

`billingEnabled: false` なので Billing Account を設定する

Billing Account の一覧を取得、ACCOUNT_ID を確認

```console
gcloud billing accounts list
```

```console
export ACCOUNT_ID=xxxxxx-xxxxxx-xxxxxx
```

Billing Account を設定

```console
gcloud billing projects link $PROJECT_ID --billing-account=$ACCOUNT_ID
```

有効になったか確認

```console
gcloud billing projects describe $PROJECT_ID
```

```
billingAccountName: billingAccounts/xxxxxx-xxxxxx-xxxxxx
billingEnabled: true
name: projects/m3-ai-team-k8s-tutorial-xxxxx/billingInfo
projectId: m3-ai-team-k8s-tutorial-xxxxx
```

## クラスタの作成

### API の有効化

API が有効になっているか確認

```console
gcloud services list --project=$PROJECT_ID | grep container.googleapis.
com
```

`container.googleapis.com` が表示されない場合は有効化する

```console
gcloud services enable container.googleapis.com --project=$PROJECT_ID
```

```
Operation "operations/acf.p2-1028706878295-621c3558-5fdf-460d-b40a-85488884ee53" finished successfully.
```

有効になったか確認

```console
gcloud services list --project=$PROJECT_ID | grep container.googleapis.com
```

```
container.googleapis.com            Kubernetes Engine API
```

### クラスタの作成

```console
export CLUSTER_NAME=tutorial-cluster
export LOCATION=asia-northeast1
```

```console
gcloud container clusters create-auto CLUSTER_NAME \
    --location=$LOCATION \
    --project=$PROJECT_ID
```

```
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.

Creating cluster tutorial-cluster in asia-northeast1... Cluster is being health-checked (Kubernetes Control Plane is healthy)...done.

Created [https://container.googleapis.com/v1/projects/m3-ai-team-k8s-tutorial-xxxxx/zones/asia-northeast1/clusters/tutorial-cluster].

To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/asia-northeast1/tutorial-cluster?project=m3-ai-team-k8s-tutorial-xxxxx

CRITICAL: ACTION REQUIRED: gke-gcloud-auth-plugin, which is needed for continued use of kubectl, was not found or is not executable. Install gke-gcloud-auth-plugin for use with kubectl by following https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#install_plugin
kubeconfig entry generated for tutorial-cluster.

NAME              LOCATION         MASTER_VERSION      MASTER_IP     MACHINE_TYPE  NODE_VERSION        NUM_NODES  STATUS
tutorial-cluster  asia-northeast1  1.30.6-gke.1125000  34.146.50.13  e2-small      1.30.6-gke.1125000  3          RUNNING
```

### クラスタへの接続

認証

```console
gcloud container clusters get-credentials $CLUSTER_NAME \
    --location=$LOCATION \
    --project=$PROJECT_ID
```

CLI のインストール

```console
gcloud components install kubectl gke-gcloud-auth-plugin
```

Namespace 一覧を表示してみる

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
