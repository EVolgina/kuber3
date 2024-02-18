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
[apps.yml]()
- Чтобы увеличить количество реплик приложения до 2, вносим изменения в YAML-файл apps.yml, установив значение replicas в 2, и снова применить его к кластеру с помощью kubectl apply
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f apps.yaml
deployment.apps/my-deployment configured
```
- Создаем файл для сервиса - vagrant@vagrant:~/kube/zad3$ sudo nano service.yml
[service.yml]()
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f service.yaml  - запускаем сервис
service/my-service created
```
- созданием Pod с приложением multitool
[mult.yml]
```
vagrant@vagrant:~/kube/zad3$ kubectl apply -f mult.yaml
pod/multitool-pod configured
vagrant@vagrant:~/kube/zad3$ kubectl get pods - проверяем все
NAME                             READY   STATUS              RESTARTS       AGE
multitool                        1/1     Running             0              92m
multitool-7f8c7df657-dqjp5       1/1     Running             0              91m
multitool-pod                    1/1     Running             0              31m
my-deployment-57d7f95b5f-45qlw   1/2     CrashLoopBackOff    16 (54s ago)   62m
my-deployment-9b5db5dc8-7kx2k    2/2     Running             0              16s
my-deployment-9b5db5dc8-9jx9g    0/2     ContainerCreating   0              4s
my-deployment-57d7f95b5f-mskhx   1/2     Terminating         16             62m
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
![1]()





