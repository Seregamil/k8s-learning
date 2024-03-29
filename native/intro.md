# k8s Intro
## Основные компоненты кластера 
`Компоненты` — это службы, модули и программы, обеспечивающие работу всего кластера. Они работают в нодах, то есть на виртуальных или физических серверах, где используется Kubernetes.  

![Простая архитектура компонентов кластера](https://blog.skillfactory.ru/wp-content/uploads/2023/02/kubernetes-6-8004460.png)  

Кластер Kubernetes включает следующие компоненты:

- `kube-apiserver` - С помощью сервера API обеспечивается работа API кластера, обрабатываются REST-операции и предоставляется интерфейс, через который остальные компоненты взаимодействуют друг с другом. Также через него проходят запросы на изменение состояния или чтение кластера. Работает на master-нодах;  
- `kube-scheduler` - Компонент, планирующий, на каких узлах разворачивать поды. Он учитывает такие факторы, как ограничения, требования к ресурсам, местонахождение данных и пр. Работает на master-нодах;  
- `etcd` - Распределенное хранилище в формате «ключ-значение». В нем хранится состояние всего кластера. Главная задача etcd — обеспечить отказоустойчивость кластера и консистентность данных. Etcd — самостоятельный проект. Он развивается отдельно от Kubernetes и применяется в разных продуктах. Работает на master-нодах;  
- `kube-proxy` - Служба, которая управляет правилами балансировки нагрузки. Она конфигурирует правила IPVS или iptables, через которые выполняются проксирование и роутинг. Работает на worker-нодах;  
- `kube-controller-manager` - Компонент запускает работу контроллеров. Работает на master-нодах;  
- `kubelet` - Cлужба, которая управляет состоянием ноды: запуском, остановкой и поддержанием работы контейнеров и подов. Работает на worker-нодах.  
___
## Glossary

1. `Kubernetes` — это платформа с открытым исходным кодом для управления контейнеризированными рабочими нагрузками и сервисами.

2. `Namespace`, пространство имен — это абстракция, которая позволяет разграничить объекты и рабочую нагрузку для разных команд или пользователей за счет разных политик доступа к объектам Kubernetes и ограничения вычислительных ресурсов.

3. `Node` — физическая или виртуальная машина, входящая в состав кластера Kubernetes.

4. `Pod` — это объект Kubernetes, который является описанием атомарной единицы рабочей нагрузки. Pod можно воспринимать, как описание запущенного инстанса сервиса или задачи.

5. `Service` — это объект Kubernetes, который описывает некоторый набор подов в качестве сетевого сервиса, а также способ доступа к этому сетевому сервису.

6. `Deployment` — это объект Kubernetes, который описывает в скольких экземплярах запущен сервис, а также стратегию обновления на новую версию.

7. `Ingress` — это объект Kubernetes, в котором описываются правила маршрутизации клиентского трафика.

8. `API Server` — это компонент управляющего слоя Kubernetes, который используется для управления кластером по API и взаимодействия внутренних компонентов.

9. `Annotation` — это пары ключ-значение, в которых хранится значимая информация, которую используют сторонние инструменты или контроллеры для своей работы.

10. `ControlPlane`, управляющий слой, также в официальной документации может называться “панель управления” и “плоскость управления” — это набор управляющих компонент Kubernetes, которые отвечают за координацию и распределение рабочей нагрузки.

11. `Kubelet` — это агент Kubernetes, который запущен на всех нодах кластера, и осуществляет работу с контейнерным окружением: следит за живостью контейнеров, запускает новые контейнеры, ограничивает контейнеры по ресурсам и т.д. 

12. `Object`, объект, ресурс – это хранящиеся внутри Kubernetes сущности. И Kubernetes их использует для представления состояния кластера.

13. `Label` — это пары типа ключ-значение, которые связаны с конкретным объектом Kubernetes, и используются, чтобы указывать наборы объектов без необходимости фиксировать конкретные идентификаторы в спецификациях

14. `Selector` — это выражения, позволяющие выбрать объекты по меткам.

15. `Controller` — это процесс, который пытается поддерживать в согласованном состоянии конфигурацию кластера и реальный мир.

16. `Manifest` — это файл с описанием объектов Kubernetes

17. `ConfigMap` — это объект Kubernetes, который хранит в себе конфигурацию

18. `Secret`, секрет — это объект, который предназначен для хранения чувствительной информации: например, логин пароль для подключения к базе данных. 
19. `Docker` — (в частности, Docker Engine) — это программное обеспечение для виртуализации на уровне операционной системы, которая также известна как контейнеризация. 
20. `CRI`, Container Runtime Interface, интерфейс среды выполнения контейнера — это API сред выполнения контейнера, которая интегрируется с kubelet на ноде. 
21. `CRI-O` — легковесная среда выполнения контейнеров в Kubernetes.
___

# Usefull links

> [Official k8s documentation](https://kubernetes.io/docs/home/)  

> [Kubernetes API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#-strong-api-overview-strong-)