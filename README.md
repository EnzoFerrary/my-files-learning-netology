# Домашнее задание к занятию «Kubernetes. Часть 1»

---

### Задание 1

**Выполните действия:**

1. Запустите Kubernetes локально, используя k3s или minikube на свой выбор.
1. Добейтесь стабильной работы всех системных контейнеров.
2. В качестве ответа пришлите скриншот результата выполнения команды kubectl get po -n kube-system.

#### Скриншот:
![1 1](https://github.com/AleksandrNiylek/-Kubernetes.-1-/assets/136619576/f68aed98-ac05-48fb-9171-54b474a5d1dd)


------
### Задание 2


Есть файл с деплоем:

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis
        env:
         - name: REDIS_PASSWORD
           value: password123
        ports:
        - containerPort: 6379
```

------
**Выполните действия:**

1. Измените файл с учётом условий:

 * redis должен запускаться без пароля;
 * создайте Service, который будет направлять трафик на этот Deployment;
 * версия образа redis должна быть зафиксирована на 6.0.13.

2. Запустите Deployment в своём кластере и добейтесь его стабильной работы.
3. В качестве решения пришлите получившийся файл.

#### Ответ:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis:6.0.13
        env:
         - name: ALLOW_EMPTY_PASSWORD
           value: "yes"
        ports:
         - containerPort: 6379
```

```
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```

##### Скриншот:
![2 2](https://github.com/AleksandrNiylek/-Kubernetes.-1-/assets/136619576/436842f5-6d87-483c-b5d4-7fc8957d9925)


------
### Задание 3

**Выполните действия:**

1. Напишите команды kubectl для контейнера из предыдущего задания:

 - выполнения команды ps aux внутри контейнера;
 - просмотра логов контейнера за последние 5 минут;
 - удаления контейнера;
 - проброса порта локальной машины в контейнер для отладки.

2. В качестве решения пришлите получившиеся команды.

#### Команты:
sudo kubectl exec redis-7bfccd74cd-8bhrk -- ps aux

sudo kubectl logs --since=5m redis-7bfccd74cd-8bhrk

sudo kubectl delete -f redis.yaml && sudo kubectl delete -f redis-service.yaml

sudo kubectl port-forward redis-7bfccd74cd-jpg8j 12345:6739

#### Скриншот:
![3 2](https://github.com/AleksandrNiylek/-Kubernetes.-1-/assets/136619576/a14f270c-3a7c-4146-a3df-d6e460dbc5e5)



------
## Дополнительные задания* (со звёздочкой)

Их выполнение необязательное и не влияет на получение зачёта по домашнему заданию. Можете их решить, если хотите лучше разобраться в материале.

---

### Задание 4*

Есть конфигурация nginx:

```
location / {
    add_header Content-Type text/plain;
    return 200 'Hello from k8s';
}
```

**Выполните действия:**

1. Напишите yaml-файлы для развёртки nginx, в которых будут присутствовать:

 - ConfigMap с конфигом nginx;
 - Deployment, который бы подключал этот configmap;
 - Ingress, который будет направлять запросы по префиксу /test на наш сервис.

2. В качестве решения пришлите получившийся файл.

#### .yaml:

configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  nginx.conf: |
    location / {
      add_header Content-Type text/plain;
      return 200 'Hello from k9s';
    }
```
---
deploument.yaml
```
kind: deployment
metadata:
 name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-configmap
```
---
ingress.yaml
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
    - http:
        paths:
          - path: /test
            backend:
              serviceName: nginx-service
              servicePort: 80
```
