# [RBAC - Role-based access control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
## Description
`RBAC (Role-based access control)` - система распределения прав доступа к различными объектам кластера k8s.    
`Пользователи кластера k8s` - все, кто шлет запросы в API-сервер, не только администраторы и разработчики, но и компоненты CI/CD, control plane, kubelet, kubeproxy на узлах, а так же приложения, запущенные в кластере.  
___
## Сущности 
В рамках модели контроля доступа на основании ролей есть пять сущностей:  
- `Role`  
- `RoleBinding`  
- `ClusterRole`  
- `ClusterRoleBinding`  
- `ServiceAccount` - самый простой тип пользователей. Автоматически генерируется для каждого создаваемого объекта кластера с ролью *default*;  
___

### [ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
* Является самым простым типом пользователя кластера.  
* Создан для ограничения прав ПО, работающего в кластере.  
* Все общение между компонентами кластера идет через запросы к API-серверу, и каждый такой запрос авторизуется посредством специального *JWT-токена*.  
Токен автоматически генерируется при создании объекта типа ServiceAccount и кладется в *secret*.

sample-service-account.yaml
```yaml
apiVersion: v1
kind: ServiceAccount
metadata: 
    name: test-account
spec:
    serviceAccountName: TestFuckingServiceAccount
```

> В отличии от обычного пользователя, которому можно задать произвольный пароль, JWT-токен содержит внутри себя служебную информацию и подписан корневым сертификатом кластера.  

Далее данный secret монтируется внутрь контейнера, где работает приложение, и, если приложение умеет в k8s, оно берет токен из файла `/var/run/secrets/kubernetes.io/serviceaccount/token` и с ним делает запросы в API кластера.

Свой ServiceAccount **default** есть в каждом namespace и создается автоматически. По умолчанию, прав у данного аккаунта на доступ к API нет никаких.

default.ServiceAccount
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2019-05-18T20:08:00Z"
  name: default # Service account name setted by YAML or default
  namespace: default # ServiceAccount namespace setted by YAML or default
  resourceVersion: "2120"
  uid: a2cf45e6-79a8-11e9-b6a7-fa163e91ccaf
secrets:
- name: default-token-2r2tz # SecretName 
```

Можно создать свой ServiceAccount и указать его имя в манифесте пода, тогда пи запуске контейнеров пода в них будет примонтирован secret с токеном указанного ServiceAccount.  

Если не указывать ServiceAccount, будет примонтирован default.ServiceAccount
___
### [Role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-example)
`Role` - YAML-манифест, который описывае некий набор прав на объекты кластера k8s; Role никому ничего не разрешают, это лишь список.  

Ниже представлен список правил, описывающих прав доступа:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    namespace: test-namespace
    name: pod-reader
rules: 
apiGroups: [""] # To Core-API group
resources: ["pods"] # Only for Pod's
verbs: ["get", "list", "watch"] # Can get pod, list all pods and watch
```
> `apiGroups` - описывает API-группу манифеста. Это то, то описано в поле `apiVersion:`. Если в *apiVersion* указана только версия, например *v1*, то считается, что у этого манифеста корневая группа *(core-group)*  

> `resources` - список ресурсов, к которым описывается доступ, во множественном числе.   
Пример перечня ресурсов: 
>> /api/v1/namespaces  
/api/v1/pods  
/api/v1/namespaces/my-namespace/pods  
/apis/apps/v1/deployments  
/apis/apps/v1/namespaces/my-namespace/deployments  
/apis/apps/v1/namespaces/my-namespace/deployments/my-deployment  

> `verbs` - список действий, которые можно сделать с ресурсами, описанными ранее:  
>> `get` - получить  
`list` - посмотреть список  
`watch` - следить за изменениями  
`create` - создать  
`update` - обновить  
`patch` - отредактировать  
`delete` - удалить   

`ResourceNames` - с помощью данного поля можно указать название объекта и, например, дать права на изменения не всех *configMap*, а только одного конкретного configMap с именем `*-controller-leader`  

Это всего лишь перечисление списка правил, объединенных в единую абстракцию. 
___
### [RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-example)
Манифест данного типа дает доступ **только** к тем сущностям, которые находятся в том же namespace, что и манифест Rolebinding.

В манифесте RoleBinding есть два типа полей
> `roleRef` - указывается Role, которая будет применена на перечень *subjects*  
`subjects` - перечисление, кому будут назначены права или данная роль  

role-bind.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: read-pods
    namespace: default
roleRef: 
  apiGroup: rbac.authorization.k8s.io
  kind: Role # this must be Role or ClusterRole  
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
subjects:
- kind: ServiceAccount
  name: pod-reader
  namespace: pod-reader
- kind: User
  name: jane              # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: developer      # ex: organization in user certificate
  apiGroup: rbac.authorization.k8s.io
```
В случае, если `kind: ServiceAccount`, то права выдаются сервис-аккаунту `ingress-nginx` в рамках namespace: `ingress-nginx`;  
> То есть, API-сервер кластера, при получении HTTP-запроса на действие с объектом кластера, увидит в заголовке JWT-токен, проверит, что токен подписан корневым сертификатом кластера, возьмет из токена название ServiceAccount, namespace и провеит соответствующие RoleBinding в кластере, чтобы узнать, разрешено ли проводить запрошенное действие над объектом кластера.  

Так же в списке `subjects` предусмотрены `kind: User` и `kind: Group`.  
Данные kind нужны для того, чтобы описать разрешения для запросов, которые были аутентифицированны не через токен от ServiceAccount, а иным способом.
___
### [ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#clusterrole-example)
Манифест типа `Role` описывает права в рамках namespace, позволяет создавать роли с одинаковым именем в разных namespace.  

`ClusterRole` является кластерным объектом, описывающим права на объекты **во всем** кластере.  

cluster-role.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata: 
    name: secret-reader
rules:
    apiGroups: [""] 
    resources: ["secrets"]
    verbs: ["get", "watch", "list"]
```

В k8s есть много преднастроенных кластерных ролей, в т.ч. роли *admin*, *edit*, *view* - описывающие права, разрешающие администрирование, редактирование и толькот просмотр сущностей.

Посмотреть роль можно в своём кластере, если у вас есть права администратора, командой

```shell
kubectl get clusterrole edit -o yaml
```
И есть ёще роль кластерного администратора, которая даёт все права на все сущности кластера.
```shell
kubectl get clusterrole cluster-admin -o yaml
```
___
### [ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#clusterrolebinding-example)
В отличии от манифеста типа *RoleBinding*, который дает доступ лишь к сущностям, которые находятся в том же namespace, что и сам манифест, манифесты типа *ClusterRole* позволяют выдать доступ к сущностям во всех неймспейсах кластера сразу.  

К сожалению, механизма, позволяющего выдать доступ к неймспейсам по label или регекспу имени не существует. Или `RoleBinding` в одном конкретном неймспейсе или `ClusterRoleBinding` сразу на все неймспейсы кластера.  
___
## Итоги

`Role` в неймспейсах привязываются через `RoleBinding` в неймспейсе для выдачи прав объекту, который находится в этом неймспейсе.  
`ClusterRole` привязываются через `ClusterRoleBinding` для выдачи прав объекту в рамках всего кластера.  

`ClusterRole` можно привязать через `RoleBinding` в неймспейсе для выдачи прав доступа в неймспейсе. Данная методика повзоляет не создавать в каждом namespace одинаковые роли *edit*, можно использлвать стандартную кластерную роль *edit* и раздавать таким способом пользователям доступ в нужный namespace.  

