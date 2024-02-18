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
[mult.yml]()
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
##Ответ:
- Нам нужно создать YAML-файл, который определит наш Deployment с Init-контейнером. Cоздаем файл nginx-deployment.yaml [deployment]()
- Создаем файл nginx-service.yaml [service]()
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
vagrant@vagrant:~/kube/zad3$ kubectl apply -f nginx-service.yaml
service/my-service1 created
vagrant@vagrant:~/kube/zad3$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
multitool          1/1     1            1           140m
my-deployment      2/2     2            2           4h39m
nginx-deployment   0/1     1            0           40s
Проверьте состояние подов:
kubectl get pods
vagrant@vagrant:~/kube/zad3$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
multitool                           1/1     Running    0          3h23m
multitool-7f8c7df657-dqjp5          1/1     Running    0          3h23m
multitool-pod                       1/1     Running    0          143m
my-deployment-9b5db5dc8-7kx2k       2/2     Running    0          112m
my-deployment-9b5db5dc8-9jx9g       2/2     Running    0          111m
nginx-deployment-568fd5d77b-ms56n   0/1     Init:0/1   0          63m
nginx-deployment-789d7b97dd-vw2tq   0/1     Init:0/1   0          3m9s
vagrant@vagrant:~/kube/zad3$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    14d
my-service   ClusterIP   10.152.183.249   <none>        80/TCP     96m
service      ClusterIP   10.152.183.161   <none>        8080/TCP   2m19s
vagrant@vagrant:~/kube/zad3$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
multitool          1/1     1            1           140m
my-deployment      2/2     2            2           4h39m
nginx-deployment   0/1     1            0           40s
Проверьте состояние подов:
kubectl get pods
vagrant@vagrant:~/kube/zad3$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
multitool                           1/1     Running    0          3h23m
multitool-7f8c7df657-dqjp5          1/1     Running    0          3h23m
multitool-pod                       1/1     Running    0          143m
my-deployment-9b5db5dc8-7kx2k       2/2     Running    0          112m
my-deployment-9b5db5dc8-9jx9g       2/2     Running    0          111m
nginx-deployment-568fd5d77b-ms56n   0/1     Init:0/1   0          63m
nginx-deployment-789d7b97dd-vw2tq   0/1     Init:0/1   0          3m9s
vagrant@vagrant:~/kube/zad3$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    14d
my-service   ClusterIP   10.152.183.249   <none>        80/TCP     96m
service      ClusterIP   10.152.183.161   <none>        8080/TCP   2m19s
```
- Теперь убедимся, что контейнер с приложением nginx не стартует до тех пор, пока инициализационный контейнер не завершит свою работу.
```
vagrant@vagrant:~/kube/zad3$ kubectl describe pod nginx-deployment-789d7b97dd-vw2tq
Name:             nginx-deployment-789d7b97dd-vw2tq
Namespace:        default
Priority:         0
Service Account:  default
Node:             vagrant/10.0.2.15
Start Time:       Sun, 18 Feb 2024 11:46:15 +0000
Labels:           app=nginx
                  pod-template-hash=789d7b97dd
Annotations:      cni.projectcalico.org/containerID: 5ad32a0433773014da0b641cb27efedeaead12a91436688098c1a1ae2866a5e5
                  cni.projectcalico.org/podIP: 10.1.52.157/32
                  cni.projectcalico.org/podIPs: 10.1.52.157/32
Status:           Pending
IP:               10.1.52.157
IPs:
  IP:           10.1.52.157
Controlled By:  ReplicaSet/nginx-deployment-789d7b97dd
Init Containers:
  init-busybox:
    Container ID:  containerd://1c1e06f9839c9952cdcba24d611244075bb1085f69e4154b8109b65dcaef1645
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:6d9ac9237a84afe1516540f40a0fafdc86859b2141954b4d643af7066d598b74
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup service; do echo waiting for my-service; sleep 2; done;
    State:          Running
      Started:      Sun, 18 Feb 2024 11:46:23 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zx46s (ro)
Containers:
  nginx-container:
    Container ID:
    Image:          nginx
    Image ID:
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zx46s (ro)
Conditions:
  Type              Status
  Initialized       False
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-zx46s:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```
- И тут я застряла. Помогите разобраться, потому что контейнеры так и замерли в стадии инициализации
```
vagrant@vagrant:~/kube/zad3$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
multitool                           1/1     Running    0          4h
multitool-7f8c7df657-dqjp5          1/1     Running    0          3h59m
multitool-pod                       1/1     Running    0          179m
my-deployment-9b5db5dc8-7kx2k       2/2     Running    0          148m
my-deployment-9b5db5dc8-9jx9g       2/2     Running    0          148m
nginx-deployment-789d7b97dd-7wkrs   0/1     Init:0/1   0          22m
nginx-deployment-56cfbbb6c-fz6x2    0/1     Init:0/1   0          3m57s
vagrant@vagrant:~/kube/zad3$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
multitool                           1/1     Running    0          4h1m
multitool-7f8c7df657-dqjp5          1/1     Running    0          4h1m
multitool-pod                       1/1     Running    0          3h1m
my-deployment-9b5db5dc8-7kx2k       2/2     Running    0          149m
my-deployment-9b5db5dc8-9jx9g       2/2     Running    0          149m
nginx-deployment-789d7b97dd-7wkrs   0/1     Init:0/1   0          23m
nginx-deployment-56cfbbb6c-fz6x2    0/1     Init:0/1   0          5m22s
vagrant@vagrant:~/kube/zad3$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
multitool                           1/1     Running    0          4h17m
multitool-7f8c7df657-dqjp5          1/1     Running    0          4h17m
multitool-pod                       1/1     Running    0          3h17m
my-deployment-9b5db5dc8-7kx2k       2/2     Running    0          165m
my-deployment-9b5db5dc8-9jx9g       2/2     Running    0          165m
nginx-deployment-789d7b97dd-7wkrs   0/1     Init:0/1   0          40m
nginx-deployment-56cfbbb6c-fz6x2    0/1     Init:0/1   0          21m
vagrant@vagrant:~/kube/zad3$ kubectl get -f myapp.yaml
error: the path "myapp.yaml" does not exist
vagrant@vagrant:~/kube/zad3$ kubectl get -f apps.yaml
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   2/2     2            2           6h38m
vagrant@vagrant:~/kube/zad3$ ls
apps.yaml  mult.yaml  nginx-deployment.yaml  nginx-service.yaml  service.yaml
vagrant@vagrant:~/kube/zad3$ kubectl get -f nginx-deployment.yaml
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/1     1            0           119m
vagrant@vagrant:~/kube/zad3$ kubectl get -f  nginx-service.yaml
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service   ClusterIP   10.152.183.161   <none>        8080/TCP   106m
```




