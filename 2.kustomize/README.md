プレーンなYAMLを使ってデプロイに成功しましたが、実際の運用ではベタ書きのYAMLでは管理が難しくなりがちです。

例えば、デプロイ先がdev環境とprod環境と複数あって設定が少し異なる、ということはありがちだと思います。

ほとんど同じYAMLを複数メンテナンスすることは非DRYで変更漏れやミスが起こりやすく管理が大変なので、いい感じにYAMLを合成する方法が欲しくなります。

## Kustomize と Skaffold を導入

### Kustomize の導入

Kustomizeは使うことでYAMLを合成することができます。

例えばここでは、以下のようにbaseとなるYAMLを作成し、そこに対してdevとprodの環境ごとに差分を適用します。

`base/kustomization.yaml`:

```yaml
namespace: TO_BE_SPECIFIED

resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
```

今回はNamespace名だけを変更するような差分を適用してみます。

`dev/kustomization.yaml`:

```yaml
namespace: tutorial

resources:
  - ../base
```

`prod/kustomization.yaml`:

```yaml
namespace: tutorial-prod

resources:
  - ../base
```

Kustomizeしただけで合成結果のYAMLは変化していないはずなので、差分がないことを確認します。

```console
kubectl diff -k dev/
```

デプロイする場合は

```console
kubectl apply -k dev/
```

のようにデプロイできます。

### Skaffold の導入

Kustomizeを導入したことでYAMLの管理が楽になりましたが、まだDockerイメージのビルドとデプロイを個別に行う必要があります。

Skaffoldを使うことで、Dockerイメージのビルドとデプロイといったパイプラインを一括で行えるようにしてみましょう。


```console
gcloud components install skaffold
```

`skaffold.yaml`:

```yaml
apiVersion: skaffold/v4beta11
kind: Config
build:
  artifacts:
    - image: asia-northeast1-docker.pkg.dev/m3-ai-team-k8s-tutorial-xxxxx/tutorial/hello-app
      context: ../kubernetes-engine-samples/quickstarts/hello-app/
      docker:
        dockerfile: Dockerfile
  local:
    useBuildkit: true

manifests:
  kustomize:
    paths:
      - dev

profiles:
  - name: dev

  - name: prod
    manifests:
      kustomize:
        paths:
          - prod
```

これで、以下のコマンドでDockerイメージのビルドとデプロイが一括で行えるようになります。

```console
skaffold run -p dev
```

### prod環境のデプロイ

prod 環境の Namespace を作成

```console
kubectl create namespace tutorial-prod
```

```console
skaffold run -p prod
```
