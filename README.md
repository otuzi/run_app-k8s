# Домашнее задание к занятию «Запуск приложений в K8S»

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

### Ответ
1. Deployment: https://github.com/otuzi/run_app-k8s/blob/main/deployment.yaml
   возникшую ошибку решил изменением работы сервиса multitool с 80 порта на порт 8080.

2 и 3. Количество подов до увеличение replicaset до 2
   ```
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl apply -f deployment.yaml 
   deployment.apps/multi-container-deployment created
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
   NAME                                          READY   STATUS              RESTARTS   AGE
   multi-container-deployment-755c6d9fb4-lg967   0/2     ContainerCreating   0          2s
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
   NAME                                          READY   STATUS    RESTARTS   AGE
   multi-container-deployment-755c6d9fb4-lg967   2/2     Running   0          10s

   ```
   количество подов после изменение replicaset до 2
   ```
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ vim deployment.yaml 
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl apply -f deployment.yaml 
   deployment.apps/multi-container-deployment configured
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods

   NAME                                          READY   STATUS    RESTARTS   AGE
   multi-container-deployment-755c6d9fb4-kr4wg   2/2     Running   0          5s
   multi-container-deployment-755c6d9fb4-lg967   2/2     Running   0          5m49s

   ```

4. Service: https://github.com/otuzi/run_app-k8s/blob/main/service.yaml
5. Команда по запуску временного пода с multitool и вывод curl из нег
   ```
   kubectl run mymultitool --image=praqma/network-multitool:latest -i --tty --rm -- sh
   ```
   Вывод:
   ```
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl run mymultitool --image=praqma/network-multitool:latest -i --tty --rm -- sh
   If you don't see a command prompt, try pressing enter.
   / #
   / # curl multi-container-service -I
   HTTP/1.1 200 OK
   Server: nginx/1.27.1
   Date: Wed, 04 Sep 2024 15:06:45 GMT
   Content-Type: text/html
   Content-Length: 615
   Last-Modified: Mon, 12 Aug 2024 14:21:01 GMT
   Connection: keep-alive
   ETag: "66ba1a4d-267"
   Accept-Ranges: bytes

   / # 
   / # curl multi-container-service:8080 -I
   HTTP/1.1 200 OK
   Server: nginx/1.18.0
   Date: Wed, 04 Sep 2024 15:06:54 GMT
   Content-Type: text/html
   Content-Length: 1597
   Last-Modified: Wed, 04 Sep 2024 14:54:54 GMT
   Connection: keep-alive
   ETag: "66d874be-63d"
   Accept-Ranges: bytes

   / # 
   / # exit
   Session ended, resume using 'kubectl attach mymultitool -c mymultitool -i -t' command when the pod is running
   pod "mymultitool" deleted
   ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
   ```

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.


### Ответ 
Файл используемый в задании:

Deployment: https://github.com/otuzi/run_app-k8s/blob/main/nginx-deployment.yaml

Service: https://github.com/otuzi/run_app-k8s/blob/main/nginx-service.yaml

вывод после запуска сервиса, начал работать основной под
```
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl apply -f nginx-deployment.yaml 
deployment.apps/nginx-deployment created
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
nginx-deployment-869548cf9b-mnqc4   0/1     Init:0/1   0          4s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
nginx-deployment-869548cf9b-mnqc4   0/1     Init:0/1   0          10s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
nginx-deployment-869548cf9b-mnqc4   0/1     Init:0/1   0          29s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl apply -f nginx-service.yaml 
service/nginx-service created
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
NAME                                READY   STATUS            RESTARTS   AGE
nginx-deployment-869548cf9b-mnqc4   0/1     PodInitializing   0          40s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-869548cf9b-mnqc4   1/1     Running   0          43s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-869548cf9b-mnqc4   1/1     Running   0          51s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ kubectl get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.152.183.1     <none>        443/TCP   78m
nginx-service   ClusterIP   10.152.183.123   <none>        80/TCP    18s
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$ 
ubuntu@fhmqhsd17bhg88oa8oac:~/k8s$
```

------
