# Kustomize と Skaffold を導入

## Kustomize の導入

dev 環境に差分がないことを確認

```console
kubectl diff -k dev/
```

## Skaffold の導入

```console
gcloud components install skaffold
```

```console
skaffold run -p dev
```

```console
skafoold run -p prod
```

## prod 環境のデプロイ

prod 環境の Namespace を作成

```console
kubectl create namespace tutorial-prod
```

```console
skaffold run -p prod
```
