クラスタの準備が出来たので、次はデプロイするサンプルアプリケーションのDockerイメージを準備します。

公式ドキュメント: https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster

## デプロイするDockerイメージを準備する

## サンプルアプリケーションをビルドする

Google Cloudが提供するサンプルアプリケーションをビルドします。

ソースコード: https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/blob/main/quickstarts/hello-app/main.go

ビルド結果のイメージが公開されていて、公式ドキュメントではそちらを使っていますが、せっかくなのでビルドしてプッシュする部分もやってみましょう。

ソースコードをcloneしてビルドします。イメージタグは後ほど作成するArtifact Registryの設定に合わせて指定します。

```console
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples.git
cd kubernetes-engine-samples/quickstarts/hello-app/
```

```console
docker build -t $LOCATION-docker.pkg.dev/$PROJECT_ID/tutorial/hello-app:latest .
```

ローカルで動作確認してみましょう

```console
docker run --rm -p 8080:8080 $LOCATION-docker.pkg.dev/$PROJECT_ID/tutorial/hello-app
```

起動ログ

```
2024/12/26 18:52:16 Server listening on port 8080
```

curlやブラウザでアクセスできればビルド成功です。

```console
curl localhost:8080
```

```
Hello, world!
Version: 1.0.0
Hostname: 748b9d88aa6e
```

### Artifact Registryにプッシュする

無事ビルドできたので、Artifact Registryにプッシュします。

プッシュ先のリポジトリが必要なので作成しましょう。

```console
gcloud artifacts repositories create tutorial --repository-format=docker \
    --location=$LOCATION \
    --project=$PROJECT_ID
```

プッシュできるように認証を設定します。

```console
gcloud auth configure-docker $LOCATION-docker.pkg.dev
```

push！

```console
docker push $LOCATION-docker.pkg.dev/$PROJECT_ID/tutorial/hello-app:latest
```
