# Домашнее задание к занятию "Сетевое взаимодействие в K8S. Часть 2" dev-17_kuber-homeworks-1.5-yakovlev_vs
kuber-homeworks-1.5

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным git-репозиторием.

------

### Инструменты/ дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с кол-вом реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool. 
3. Добавить Service'ы, которые обеспечат доступ к обоим приложениям внутри кластера. 
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment'а и Service в решении, а также скриншоты или вывод команды п.4.

#### Решение

- [Frontend](file/frontend.yaml) - создает Deployment и svc к нему

- [Backend](file/backend.yaml) - создает Deployment и svc к нему


```bash
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS       AGE
myapp-pod-76f4f9959f-fr8kg   1/1     Running   3 (102m ago)   7d18h
frontend-79cc958448-4c7r2    1/1     Running   0              12m
frontend-79cc958448-9gxtk    1/1     Running   0              12m
frontend-79cc958448-g4pbk    1/1     Running   0              12m
backend-5596b5d66d-mqm4m     1/1     Running   0              12m
$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP                         12d
np-mysvc     NodePort    10.152.183.210   <none>        9001:30080/TCP,9002:32080/TCP   41h
fe-svc       ClusterIP   10.152.183.119   <none>        80/TCP                          34s
be-svc       ClusterIP   10.152.183.234   <none>        80/TCP                          25s
```

Проверяем доступность друг друга
- доступность backend
```bash
$ kubectl exec frontend-79cc958448-4c7r2 -- curl be-svc
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   141  100   141    0     0  15666      0 --:--:-- --:--:-- --:--:-- 15666
WBITT Network MultiTool (with NGINX) - backend-5596b5d66d-mqm4m - 10.1.128.243 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

- доступность frontend
```bash
 $ kubectl exec backend-5596b5d66d-mqm4m -- curl fe-svc
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  15444      0 --:--:-- --:--:-- --:--:-- 15692
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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



------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

#### Решение

1. Включить Ingress-controller в microk8s

```bash
root@microk8s:~# microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
```

2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера microk8s, так чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_

- [Ingress](file/ingress.yaml)

```bash
$ kubectl describe ingress
Name:             my-ingress
Namespace:        default
Address:          127.0.0.1
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /      fe-svc:80 (10.1.128.234:80,10.1.128.245:80,10.1.128.246:80)
              /api   be-svc:80 (10.1.128.243:80)
Annotations:  <none>
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    8m51s (x2 over 9m39s)  nginx-ingress-controller  Scheduled for sync
```

3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера

```bash
$curl 192.168.1.88
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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

```bash
$ curl 192.168.1.88/api
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   141  100   141    0     0  15666      0 --:--:-- --:--:-- --:--:-- 15666
WBITT Network MultiTool (with NGINX) - backend-5596b5d66d-mqm4m - 10.1.128.243 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

4. Предоставить манифесты, а также скриншоты или вывод команды п.2

- [Ingress](file/ingress.yaml)
------

### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md

------