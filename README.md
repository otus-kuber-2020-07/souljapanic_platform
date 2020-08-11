# souljapanic_platform
souljapanic Platform repository

# kubernetes-intro

## Настройка рабочего окружения:

* Установка minikube с driver virtualbox:

```
minikube start --driver=virtualbox
```

* Проверка статуса minikube:

```
minikube status
```

* Подключение к виртуальной машине minikube:

```
minikube ssh
```

* Проверка статуса k8s:

```
kubectl cluster-info
kubectl get cs
```

* Проверка статуса POD'ов проекта kube-system:

```
kubectl get pods -n kube-system
```

## Проверка отказоустойчивости:

Для проверки отказоустойчивости удалим все контейнере на виртуальной машине и проверим, будет выполнен их повторный запуск или нет.

Выполняем следующие команды:

* minikube ssh

* docker rm -f $(docker ps -a -q)

Выполняем проверку:

* kubectl get pods -n kube-system

Наблюдаем, что POD'ы запустились повторно.

## Методы запуска POD'ов:

```
kubectl get deployment.apps/coredns -o yaml -n kube-system
Используется ReplicaSet
```

```
kubectl get daemonset.apps/kube-proxy -o yaml -n kube-system
Используется Daemonset
```

```
Все остальные POD'ы используют Static Pods.
Конфигурация хранится на виртуальной машине minikube в диреткории /etc/kubernetes/manifests/
```

## Создание POD'а web:

* Сборка и публикация образа:

```
podman build --rm --no-cache -t docker.io/souljapanic/kubernetes-intro-web -f Dockerfile .
podman push docker.io/souljapanic/kubernetes-intro-web
```

* kubectl apply -f web-pod.yaml -n default

* kubectl get pods -n default

* Проверка UID:

```
kubectl exec -it pod/web -- id
uid=1001(nginx) gid=101(nginx) groups=101(nginx)
```

* Проверка работоспособности контейнера:

```
kubectl port-forward --address 0.0.0.0 pod/web 8000:8000 -n default
curl localhost:8000
```

## Создание POD'а frontend:

* Сборка и публикация образа:

```
podman build --rm --no-cache -t docker.io/souljapanic/kubernetes-intro-frontend -f microservices-demo/src/frontend/Dockerfile microservices-demo/src/frontend/.
podman push docker.io/souljapanic/kubernetes-intro-frontend
```

* kubectl run frontend --image docker.io/souljapanic/kubernetes-intro-frontend --restart=Never

* Создание manifest файла:

```
kubectl run frontend --image docker.io/souljapanic/kubernetes-intro-frontend --restart=Never --dry-run=client -o yaml > frontend-pod.yaml
```

* Отладка POD'а:

```
kubectl logs frontend -n default
```

Ошибка связана с отсутствующими переменными.

* kubectl apply -f frontend-pod-healthy.yaml -n default

* Просмотр переменных:

```
kubectl set env pod/frontend --list
```
