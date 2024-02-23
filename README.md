# Домашнее задание к занятию «Запуск приложений в K8S»
- Цель задания
- В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.
## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod
- Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
- После запуска увеличить количество реплик работающего приложения до 2.
- Продемонстрировать количество подов до и после масштабирования.
- Создать Service, который обеспечит доступ до реплик приложений из п.1.
- Создать отдельный Pod с приложением multitool и убедиться с помощью curl, что из пода есть доступ до приложений из п.1.

```
kubectl apply -f apps.yaml
vagrant@vagrant:~/kube/zad3$ kubectl apply -f apps.yaml
deployment.apps/my-deployment created
```
[apps.yml](https://github.com/EVolgina/kuber3/blob/main/apps)
- Чтобы увеличить количество реплик приложения до 2, вносим изменения в YAML-файл apps.yml, установив значение replicas в 2, и снова применить его к кластеру с помощью kubectl apply
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f apps.yaml
deployment.apps/my-deployment configured
```
- Создаем файл для сервиса - vagrant@vagrant:~/kube/zad3$ sudo nano service.yml
[service.yml](https://github.com/EVolgina/kuber3/blob/main/service)
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f service.yaml  - запускаем сервис
service/my-service created
```
- созданием Pod с приложением multitool
[mult.yml](https://github.com/EVolgina/kuber3/blob/main/mult)
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f mult.yaml
pod/multitool-pod configured
vagrant@vagrant:~/kube/zad3$ kubectl get pods - проверяем все
NAME                            READY   STATUS    RESTARTS   AGE
multitool                       1/1     Running   0          126m
multitool-7f8c7df657-dqjp5      1/1     Running   0          125m
multitool-pod                   1/1     Running   0          65m
my-deployment-9b5db5dc8-7kx2k   2/2     Running   0          34m
my-deployment-9b5db5dc8-9jx9g   2/2     Running   0          34m
vagrant@vagrant:~/kube/zad3$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP   14d
my-service   ClusterIP   10.152.183.249   <none>        80/TCP    32m
vagrant@vagrant:~/kube/zad3$ kubectl describe service my-service
Name:              my-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=my-app
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.152.183.249
IPs:               10.152.183.249
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:
Session Affinity:  None
Events:            <none>
```
```
vagrant@vagrant:~/kube/zad3$ kubectl get deployment my-deployment
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   2/2     2            2           3h58m
vagrant@vagrant:~/kube/zad3$ kubectl get svc my-service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
my-service   ClusterIP   10.152.183.249   <none>        80/TCP    43m
vagrant@vagrant:~/kube/zad3$ kubectl exec -it multitool-pod -- curl http://my-service
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
![1](https://github.com/EVolgina/kuber3/blob/main/3.PNG)
## Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий
- Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
- Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
- Создать и запустить Service. Убедиться, что Init запустился.
- Продемонстрировать состояние пода до и после запуска сервиса.
## Ответ:
- Нам нужно создать YAML-файл, который определит наш Deployment с Init-контейнером. Cоздаем файл nginx-deployment.yaml [deployment](https://github.com/EVolgina/kuber3/blob/main/nginx-deployment)
- Создаем файл nginx-service.yaml [service](https://github.com/EVolgina/kuber3/blob/main/nginx-service)
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
vagrant@vagrant:~/kube/zad3$ kubectl apply -f nginx-service.yaml
service/my-service1 created
devops@WORKBOOK:/mnt/kube$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           2d18h
Проверим состояние подов:
kubectl get pods
devops@WORKBOOK:/mnt/kube$ kubectl get pods
NAME                               READY   STATUS    RESTARTS      AGE
hello-world                        1/1     Running   7 (42h ago)   7d19h
netology-web                       1/1     Running   7 (42h ago)   7d18h
nginx-deployment-bfc6fffb9-bjnvd   1/1     Running   0             18s
devops@WORKBOOK:/mnt/kube$ kubectl get svc
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.152.183.1     <none>        443/TCP    10d
netology-svc   ClusterIP   10.152.183.204   <none>        80/TCP     7d18h
service        ClusterIP   10.152.183.65    <none>        8080/TCP   2d18h
my-service     ClusterIP   10.152.183.52    <none>        8080/TCP   43h
```
- Теперь убедимся, что контейнер с приложением nginx не стартует до тех пор, пока инициализационный контейнер не завершит свою работу.
```
devops@WORKBOOK:/mnt/kube$ kubectl describe pod nginx-deployment-bfc6fffb9-bjnvd
Name:             nginx-deployment-bfc6fffb9-bjnvd
Namespace:        default
Priority:         0
Service Account:  default
Node:             workbook/172.23.49.32
Start Time:       Fri, 23 Feb 2024 14:51:19 +0500
Labels:           app=nginx
                  pod-template-hash=bfc6fffb9
Annotations:      cni.projectcalico.org/containerID: 091d728b7d62b63d756d77d4afd3a4ed43b2277b9a6365565fab098cfaee2fd0
                  cni.projectcalico.org/podIP: 10.1.211.220/32
                  cni.projectcalico.org/podIPs: 10.1.211.220/32
Status:           Running
IP:               10.1.211.220
IPs:
  IP:           10.1.211.220
Controlled By:  ReplicaSet/nginx-deployment-bfc6fffb9
Init Containers:
  init-busybox:
    Container ID:  containerd://687f0f6d9c95f6e7718cf9d68fc2dfed146f4402418faf3bacc0d99c733e121e
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:6d9ac9237a84afe1516540f40a0fafdc86859b2141954b4d643af7066d598b74
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup my-service.default.svc.cluster.local; do echo waiting for my-service; sleep 2; done;
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 23 Feb 2024 14:51:25 +0500
      Finished:     Fri, 23 Feb 2024 14:51:25 +0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gzskn (ro)
Containers:
  nginx-container:
    Container ID:   containerd://4783a3e8de19a4293d17501215cf609c913fda0bbb72d70ab6d5f000d49cbbd4
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:c26ae7472d624ba1fafd296e73cecc4f93f853088e6a9c13c0d52f6ca5865107
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 23 Feb 2024 14:51:28 +0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gzskn (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-gzskn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m20s  default-scheduler  Successfully assigned default/nginx-deployment-bfc6fffb9-bjnvd to workbook
  Normal  Pulling    4m18s  kubelet            Pulling image "busybox"
  Normal  Pulled     4m15s  kubelet            Successfully pulled image "busybox" in 3.317s (3.317s including waiting)
  Normal  Created    4m15s  kubelet            Created container init-busybox
  Normal  Started    4m15s  kubelet            Started container init-busybox
  Normal  Pulling    4m14s  kubelet            Pulling image "nginx"
  Normal  Pulled     4m12s  kubelet            Successfully pulled image "nginx" in 1.795s (1.795s including waiting)
  Normal  Created    4m12s  kubelet            Created container nginx-container
  Normal  Started    4m12s  kubelet            Started container nginx-container
```
- Переделываем файлы, перезапускаем все [depoloy](https://github.com/EVolgina/kuber3/blob/main/deploy)
```
devops@WORKBOOK:/mnt/kube$ kubectl get pods - До
NAME                               READY   STATUS    RESTARTS      AGE
hello-world                        1/1     Running   7 (42h ago)   7d19h
netology-web                       1/1     Running   7 (42h ago)   7d18h
nginx-deployment-bfc6fffb9-bjnvd   1/1     Running   0             5m55s
devops@WORKBOOK:/mnt/kube$ kubectl apply -f depoy.yml
deployment.apps/nginx-deployment configured
service/my-service configured
devops@WORKBOOK:/mnt/kube$ kubectl get pods -После
NAME                                READY   STATUS    RESTARTS      AGE
hello-world                         1/1     Running   7 (42h ago)   7d19h
netology-web                        1/1     Running   7 (42h ago)   7d18h
nginx-deployment-7cc746858d-xxzrp   1/1     Running   0             15s
```





