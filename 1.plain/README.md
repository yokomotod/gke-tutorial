お疲れ様でした。クラスタとアプリケーションのイメージが出来たので、いよいよデプロイしていきましょう。



## プレーンなYAMLファイルでのデプロイ

Kubernetesにデプロイするものは基本的にYAMLファイルで記述します。

まずは普通にYAMLファイルを作成してデプロイしてみましょう。


### Namespaceの作成

Namespaceはデプロイするリソースを分けるためのものです。

今回は `tutorial-dev` という名前で作成します。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tutorial-dev
```

```console
kubectl apply -f namespace.yaml
```

### Deployment の作成

DeploymentはPodのデプロイを管理するリソースです。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloweb
  namespace: tutorial
  labels:
    app: hello
spec:
  selector:
    matchLabels:
      app: hello
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello-app
          image: asia-northeast1-docker.pkg.dev/m3-ai-team-k8s-tutorial-xxxxx/tutorial/hello-app:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 200m
```

```console
kubectl apply -f deployment.yaml
```

デプロイすると `replicas: 3` に従ってPodが3つ作成されているはずです。

```console
kubectl -n tutorial get po
```

```
NAME                        READY   STATUS    RESTARTS   AGE
helloweb-5b6b6cc799-wq6n6   1/1     Running   0          14s
helloweb-5b6b6cc799-sl38s   1/1     Running   0          14s
helloweb-5b6b6cc799-t6n8h   1/1     Running   0          14s
```

ログも見てみましょう。

```console
kubectl -n tutorial logs helloweb-5b6b6cc799-t6n8h
```

```
2024/12/26 19:02:32 Server listening on port 8080
```

無事起動しましたが、まだ外部からアクセスできるようにはなっていません。

### Service、Ingress の作成

Global Load Balancerを使って外部からアクセスできるようにします。

Global Load BalancerはGKEクラスタの外にあるGoogle Cloudのリソースですが、GKEではIngressリソースを使うことでk8sから構築できます。

Serviceの作成

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloweb
  namespace: tutorial
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 8080
```

```console
kubectl apply -f service.yaml
```

Ingreesの作成

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloweb
  namespace: tutorial
spec:
  defaultBackend:
    service:
      name: helloweb
      port:
        number: 80
```

```console
kubectl apply -f ingress.yaml
```

applyしてしばらくすると `ADDRESS` が割り当てられ、さらにまたしばらくするとアクセスできるようになります。

（見た目構築が完了したように見えてもアクセスが出来ない時間がしばらくあるので、なにか構築を間違えたのかと心配になるのですが、焦らずしばらく待ってみましょう）

```console
kubectl -n tutorial get ing
```

```
NAME       CLASS    HOSTS   ADDRESS         PORTS   AGE
helloweb   <none>   *       34.54.192.203   80      5m28s
```

```console
curl 34.54.192.203
```

```
Hello, world!
Version: 1.0.0
Hostname: helloweb-5b6b6cc799-t6n8h
```

おめでとうございます！デプロイに成功しました！
