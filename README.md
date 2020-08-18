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
Конфигурация хранится на виртуальной машине minikube в директории /etc/kubernetes/manifests/
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

# kubernetes-controllers

## Настройка окружения:

* Создание кластера:

```
kind create cluster --config kind-config.yaml
```

* Список кластеров:

```
kind get clusters
```

* Информация о кластере:

```
kubectl cluster-info --context kind-kind
```

## ReplicaSet

* Проверка работы ReplicaSet, matchLabels выполняется по app=frontend:

```
kubectl apply -f frontend-replicaset.yaml -n default
```

* Увеличение количества replica:

```
kubectl scale replicaset frontend --replicas=3
```

* Проверка образа указанного в ReplicaSet:

```
kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}' -n default
```

* Проверка образа из которого запущен контейнер:

```
kubectl get pods -l app=frontend -o=jsonpath='{.items[0].spec.containers[0].image}' -n default
```

### Описание:

```
ReplicaSet сделит за тем, сколько POD'ов должно быть запущено, значение указано в .spec.replicas
```

## Deployment

* Проверка работы Deployment:

```
kubectl apply -f paymentservice-deployment.yam -n default
```

* История Deployment:

```
kubectl rollout history deployment paymentservice -n default
```

* Просмотр всех элементов namespace:

```
kubectl get all -o name -n default

pod/paymentservice-7c8468cf9-hcwts
pod/paymentservice-7c8468cf9-kkf6k
pod/paymentservice-7c8468cf9-xvqbx
service/kubernetes
deployment.apps/paymentservice
replicaset.apps/paymentservice-68c65b7974
replicaset.apps/paymentservice-7c8468cf9
```

* Возврат к предыдущей версии:

```
kubectl rollout undo deployment paymentservice --to-revision=1 -n default
```

* Проверка работы сценария развёртывания blue-green:

```
kubectl apply -f paymentservice-deployment-bg.yaml -n default
```

* Проверка работы сценария развёртывания Reverse Rolling Update:

```
kubectl apply -f paymentservice-deployment-reverse.yaml -n default
```

## Probes

* Проверка работы readinessProbe:

```
kubectl apply -f frontend-deployment.yaml -n default
```

* Просмотр состояние POD'ов:

```
kubectl describe pod -n default
```

* Проверка статуса обновления:

```
kubectl rollout status deployment/frontend --timeout=60s -n default
```

* Отмена развёртывания с некорректным readinessProbe:

```
kubectl rollout undo deployment/frontend -n default
```

## DaemonSet

* Создание namespace:

```
kubectl create namespace monitor
```

* Развёртывание node-exporter:

```
kubectl apply -f node-exporter-daemonset.yaml -n monitor
```

* Проверка node-exporter:

```
kubectl port-forward node-exporter-7s2bg 9100:9100 -n monitor

curl -v localhost:9100/metrics
```

### Развёртывание node-exporter на всех узлах:

* Создание SA и проверка:

```
kubectl apply -f node-exporter-serviceAccount.yaml

kubectl get serviceaccounts -n monitor
```

* Создание cluster role и проверка:

```
kubectl apply -f node-exporter-clusterRole.yaml

kubectl get clusterroles.rbac.authorization.k8s.io
```

* Roles Bindings и проверка:

```
kubectl apply -f node-exporter-clusterRoleBinding.yaml

kubectl get clusterrolebindings.rbac.authorization.k8s.io
```

* Развёртывание node-exporter:

```
kubectl apply -f node-exporter-daemonset.yaml
```

* Создание сервиса и проверка:

```
kubectl apply -f node-exporter-service.yaml

kubectl get services -n monitor
```

