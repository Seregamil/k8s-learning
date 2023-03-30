# K8s objects
## Description

### Спецификация и статус объекта

Почти каждый объект k8s содержит в себе два вложенных поля-объекта, которые управляют конфигурацией объекта: `spec` и `status`.  
> `spec` указывают *требуемое* состояние объекта (описание характеристик, которые должны быть у объекта)  

> `status` описывает *текущее состояние* объекта, которое создается и обновляется самим k8s и его компонентами.  

`deployment.yaml`  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### Обязательные поля
В `.yaml`-файле требуется всегда указывать значения следующих полей:  
- `apiVersion` - используемая для создания версия API k8s  
- `kind` - тип объекта  
- `metadata` - данные, позволяющте идентифицировать объект (`name`, `UID`, `namespace`)  
- `spec` - требуемое состояние объекта кластера  

> Конкретный формат поля-объекта `spec` зависит от типа объекта Kubernetes и содержит вложенные поля, предназначенные только для используемого объекта. В справочнике API Kubernetes можно найти формат спецификации любого объекта Kubernetes. Например, формат `spec` для объекта `Pod` находится в ядре [PodSpec v1](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#podspec-v1-core), а формат spec для Deployment — в [DeploymentSpec v1 apps](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#deploymentspec-v1-apps).

## Объекты Kubernetes

Полный перечень ресурсов и объектов представлен на [официальном сайте](https://kubernetes.io/docs/reference/kubernetes-api/)  

___
### Pod ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/pods/))
> `Pod` самой малой единицей объекта кластера k8s, способный содержать в себе один и более контейнер приложения. Контейнеры, размещенные в рамках одного пода используют одни и те же ресурсы.  

> `Pod` является *одним и единственным* объектом кластера, который запускает контейнер приложения. *Нет Pod, нет контейнера.*  

Расширенный список [примеров манифестов](https://k8s-examples.container-solutions.com/examples/Pod/Pod.html)  
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#pod-v1-core)  

debug-network.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-network-pod
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: praqma/network-multitool
      name: debug-network-container
```

simple.yaml
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pods-simple-pod
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: pods-simple-container
```
___
### Controllers ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/))

* `Deployment` (развертывание) - Набор алгоритмов и директив, описывающих поды, число их экземпляров, порядок их замены при изменении характеристик. С его помощью разработчик декларирует изменения нодов и наборов реплик. Это самый популярный способ разместить приложение в Kubernetes.  
* `ReplicaSet` (набор реплик) - Описывает и управляет работой нескольких подов на кластере. Чем больше таких реплик, тем выше устойчивость системы к отказам и лучше масштабируются приложения.  
* `StatefulSet` (набор состояния) - Контроллер применяется для управления приложениями с отслеживанием состояния (stateful-приложениями) с использованием постоянного хранилища.  
* `DaemonSet` (набор «демонов») - Представляет собой совокупность «демонов» (фоновых программ, работающих без взаимодействия с пользователем) и отвечает за запуск одной реплики выбранного пода на каждом отдельном узле. То есть по мере добавления узлов в кластер добавляются и поды.
* `Job` (задание) - Контроллер, отвечающий за однократный запуск выбранного пода и завершающий его работу.   
* `CronJob` — его разновидность, запускающая группу заданий по заданному расписанию.  
___
#### Deployment ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/))
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#deployment-v1-apps)  

nginx-deployment.yaml  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

В данном примере:  
* Наименование `Deployment`, согласно `.metadata.name` - **nginx-deployent**  
* `Deployment` создает `ReplicaSet`, которая в свою очередь декларирует три реплики `Pod`, согласно `.spec.replicas`  
* Спецификация `.spec.selector.matchLabels` отмечает поды, по `.labels.app` которых будет производиться поиск и репликация `Pod` 
* Спецификация `template` содержит следующие вложенные спецификации:  
    * `Pod` с ссылкой **app: nginx** (*.spec.template.metadata.labels*);  
    * Спецификация контейнера с наименованием *nginx*, использующим образ *nginx:1.14.2* и открытым портом *80*

___
#### ReplicaSet ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/))
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#replicaset-v1-apps)

Основная роль `ReplicaSet` - поддержание актуальное число реплик подов в текущий момент времени. Используется в качестве гаранта доступности определенного набора подов.  

По сути, использование `ReplicaSet` в рамках написания манифестов не является необходимостью, т.к. `Deployment` решает данную задачу и является более высокоуровневой реализацией `ReplicaSet`.  

___
#### StatefulSet ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/))
[Habr](https://habr.com/ru/company/timeweb/blog/703550/)  
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#statefulset-v1-apps)  
[simple-stateful-set.yaml](https://k8s-examples.container-solutions.com/examples/StatefulSet/StatefulSet.html)

`StatefulSet` – это контроллер Kubernetes, применяемый для эксплуатации сохраняющих состояние приложений в виде контейнеров (подов) в кластере Kubernetes. `StatefulSet` присваивают каждому поду идентификатор-липучку *(sticky identity)* – порядковый номер начиная с нуля — а не случайные ID каждой реплике пода. Новый под создается клонированием данных уже существовавшего пода. Если ранее существовавший под находился в ожидающем состоянии, то новый под создан не будет. Удаление подов происходит в обратном порядке, а не в случайном.  
Например, если у вас было четыре реплики, и в результате масштабирования их количество было сокращено до трех, то под номер 3 будет удален.

Развертывание приложений с сохранением состояния в кластере Kubernetes порой бывает утомительной задачей. Дело в том, что приложение, сохраняющее состояние ожидает, что придется работать с архитектурой «ведущий-ведомый» (primary-replica), а имя пода будет фиксированным.  
Контроллер `StatefulSets` решает эту проблему, развертывая в кластере Kubernetes приложение с сохранением состояния.  

___ 
### DaemonSet ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/))
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#daemonset-v1-apps)  
[simple-daemon-set.yaml](https://k8s-examples.container-solutions.com/examples/DaemonSet/DaemonSet.html)  

`DaemonSet` — объект API Kubernetes, который гарантирует, что определенный под будет запущен на всех (или некоторых) узлах.

> `selector` — определяет какой под будет запущен на узле.  
`labels` — cелектор меток шаблона модуля. Значение данного параметра должно совпадать с `Selector`.  
`nodeSelector` — определяет, на каких узлах должны быть развернуты реплики пода.  


Чтобы убедиться, что поды были созданы и каждый узел содержит реплики этих подов:

1. Проверить создание подов можно с помощью команды oc get pods:

```bash
$ oc get pods

>> hello-daemonset-cx6md 1/1 Running 0 2m
>> hello-daemonset-e3md9 1/1 Running 0 2m
```

2. Проверить расположение подов на узле можно с помощью команды `oc describe pod`:

```bash
$ oc describe pod/hello-daemonset-cx6md|grep Node
>> Node: openshift-node01.hostname.com/10.14.20.134
$ oc describe pod/hello-daemonset-e3md9|grep Node
>> Node: openshift-node02.hostname.com/10.14.20.137
```
___
### Job ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/job/))
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#job-v1-batch)  
[Habr](https://habr.com/ru/company/southbridge/blog/526130/)  

`Job` (работа, задание) — это yaml-манифест, который создаёт под для выполнения разовой задачи.  
Если запуск задачи завершается с ошибкой, `Job` перезапускает поды до успешного выполнения или до истечения таймаутов.  
Когда задача выполнена, `Job` считается завершённым и больше никогда в кластере не запускается. `Job` — это сущность для разовых задач.

Применение Job:  
> При установке и настройке окружения.  
Например, мы построили CI/CD, который при создании новой ветки автоматически создаёт для неё окружение для тестирования. Появилась ветка — в неё пошли коммиты — CI/CD создал в кластере отдельный namespace и запустил Job — тот, в свою очередь, создал базу данных, налил туда данные, все конфиги сохранил в `Secret` и `ConfigMap`. То есть `Job` подготовил цельное окружение, на котором можно тестировать и отлаживать новую функциональность.  

> При выкатке helm chart. После развёртывания helm chart с помощью хуков (hook) запускается `Job`, чтобы проверить, как раскатилось приложение и работает ли оно.
___
### CronJob ([kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/))
[Спецификация](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#cronjob-v1-batch)  
Создает `Job`, с той лишь разницей, что он может выполняться по расписанию Cron.  

Cron syntax  
```bash
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *
```
cron-sample.yaml
```yaml
---
# https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-simple
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - args:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster cronjob
              image: busybox
              name: cronjob-simple-container
          restartPolicy: OnFailure
```
___