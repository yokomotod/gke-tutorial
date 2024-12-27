# プレーンな YAML ファイルでのデプロイ

Namespace の作成

```console
kubectl apply -f namespace.yaml
```

Deployment の作成

```console
kubectl apply -f deployment.yaml
```

```
namespace/tutorial created
deployment.apps/helloweb created
```

```console
kubectl -n tutorial get po
```

```
NAME                        READY   STATUS    RESTARTS   AGE
helloweb-5b6b6cc799-t6n8h   1/1     Running   0          14s
```

```console
kubectl -n tutorial logs helloweb-5b6b6cc799-t6n8h
```

```
2024/12/26 19:02:32 Server listening on port 8080
```

Service の作成

```console
kubectl apply -f service.yaml
```

Ingrees の作成

```console
kubectl apply -f ingress.yaml
```

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
