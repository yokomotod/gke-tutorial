本家: https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster

# デプロイする Docker イメージを作成する

## Google Artifact Registry のリポジトリを作成する

```console
gcloud artifacts repositories create tutorial --repository-format=docker \
    --location=$LOCATION \
    --project=$PROJECT_ID
```

```
Create request issued for: [tutorial]
Waiting for operation [projects/m3-ai-team-k8s-tutorial-xxxxx/locations/asia-northeast1/operations/32d8a90b-a277-4ff0-a852-07c2de09e277] to
complete...done.
Created repository [tutorial].
```

認証

```console
gcloud auth configure-docker $LOCATION-docker.pkg.dev
```

```
Adding credentials for: asia-northeast1-docker.pkg.dev
After update, the following will be written to your Docker config file located at [/home/ubuntu/.docker/config.json]:
 {
  "credHelpers": {
    "asia-northeast1-docker.pkg.dev": "gcloud"
  }
}

Do you want to continue (Y/n)?

Docker configuration file updated.
```

## サンプルアプリケーションをビルドする

https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/blob/main/quickstarts/hello-app/main.go

```console
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples.git
```

```console
cd kubernetes-engine-samples/quickstarts/hello-app/
```

```console
docker build -t $LOCATION-docker.pkg.dev/$PROJECT_ID/tutorial/hello-app:latest .
```

ローカルで動作確認

```console
docker run --rm -p 8080:8080 $LOCATION-docker.pkg.dev/$PROJECT_ID/tutorial/hello-app
```

```
2024/12/26 18:52:16 Server listening on port 8080
```

```console
curl localhost:8080
```

GAR にプッシュ

```console
docker push $LOCATION-docker.pkg.dev/$PROJECT_ID/tutorial/hello-app:latest
```
