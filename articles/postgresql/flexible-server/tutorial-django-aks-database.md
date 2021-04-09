---
title: Руководство по развертыванию Django в кластере AKS с гибким сервером PostgreSQL с помощью Azure CLI
description: Узнайте, как быстро создать и развернуть Django в AKS с гибким сервером Базы данных Azure для PostgreSQL.
ms.service: postgresql
author: mksuni
ms.author: sumuth
ms.topic: tutorial
ms.date: 12/10/2020
ms.custom: mvc
ms.openlocfilehash: 71066fc2e2f87405455a059fe23c20277c4b09fb
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105726385"
---
# <a name="tutorial-deploy-django-app-on-aks-with-azure-database-for-postgresql---flexible-server"></a>Руководство по развертыванию приложения Django в AKS с гибким сервером Базы данных Azure для PostgreSQL

В рамках этого краткого руководства вы развернете приложение Django в кластере Azure Kubernetes Service (AKS) с гибким сервером Базы данных Azure для PostgreSQL (предварительная версия) с помощью Azure CLI.

**[AKS](../../aks/intro-kubernetes.md)**  — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. **[Гибкий сервер Базы данных Azure для PostgreSQL (предварительная версия)](overview.md)**  — это полностью управляемая служба базы данных, которая обеспечивает более детализированный контроль и гибкость функций управления базами данных, а также параметрами конфигурации.

> [!NOTE]
> - Сейчас гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме общедоступной предварительной версии.
> - В этом кратком руководстве предполагается, что у вас есть базовое представление о функциях Kubernetes, Django и PostgreSQL.

## <a name="pre-requisites"></a>Предварительные требования
[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

- [Azure Cloud Shell](../../cloud-shell/quickstart.md) можно использовать в среде Bash.

   [![Внедрение запуска](https://shell.azure.com/images/launchcloudshell.png "Запуск Azure Cloud Shell")](https://shell.azure.com)  
- При необходимости [установите](/cli/azure/install-azure-cli) Azure CLI, чтобы выполнять справочные команды CLI.
  - Если вы используете локальную установку, выполните вход с помощью команды Azure CLI [az login](/cli/azure/reference-index#az-login).  Чтобы выполнить аутентификацию, следуйте инструкциям в окне терминала.  Сведения о дополнительных возможностях, доступных при входе, см. в статье [Вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli).
  - Если появится запрос, установите расширения Azure CLI при первом использовании.  Дополнительные сведения о расширениях см. в статье [Использование расширений с Azure CLI](/cli/azure/azure-cli-extensions-overview).
  - Выполните команду [az version](/cli/azure/reference-index?#az_version), чтобы узнать установленную версию и зависимые библиотеки. Чтобы обновиться до последней версии, выполните команду [az upgrade](/cli/azure/reference-index?#az_upgrade). Для работы с этой статьей требуется последняя версия Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!NOTE]
> Если команды из этого краткого руководства выполняются в локальной среде (а не в Azure Cloud Shell), убедитесь, что они выполняются от имени администратора.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure — это логическая группа, в которой развертываются и управляются ресурсы Azure. Создайте группу ресурсов *django-project* в расположении *eastus* с помощью команды [az group create] [az-group-create].

```azurecli-interactive
az group create --name django-project --location eastus
```

> [!NOTE]
> Расположением для группы ресурсов называется место хранения метаданных для этой группы ресурсов. Также в нем выполняются ресурсы Azure, если вы не укажете другой регион во время создания конкретного ресурса.

В следующем примере выходных данных показано, что группа ресурсов успешно создана:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/django-project",
  "location": "eastus",
  "managedBy": null,
  
  "name": "django-project",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-aks-cluster"></a>Создание кластера AKS

Используйте команду [az aks create](/cli/azure/aks#az-aks-create), чтобы создать кластер AKS. В следующем примере создается кластер *myAKSCluster* с одним узлом. Это может занять несколько минут.

```azurecli-interactive
az aks create --resource-group django-project --name djangoappcluster --node-count 1 --generate-ssh-keys
```

Через несколько минут выполнение команды завершается и отображаются сведения о кластере в формате JSON.

> [!NOTE]
> При создании кластера AKS для хранения ресурсов AKS автоматически создается вторая группа ресурсов. См. раздел [Почему с AKS создаются две группы ресурсов?](../../aks/faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Управлять кластером Kubernetes можно c помощью [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Чтобы установить `kubectl` локально, используйте команду [az aks install-cli](/cli/azure/aks#az-aks-install-cli):

```azurecli-interactive
az aks install-cli
```

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials](/cli/azure/aks#az-aks-get-credentials). Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group django-project --name djangoappcluster
```

> [!NOTE]
> Приведенная выше команда использует расположение по умолчанию [файла конфигурации Kubernetes](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) — `~/.kube/config`. Вы можете указать другое расположение файла конфигурации Kubernetes с помощью параметра *--file*.

Чтобы проверить подключение к кластеру, используйте команду [kubectl get]( https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) для получения списка узлов кластера.

```azurecli-interactive
kubectl get nodes
```

В следующем примере показан единый узел, созданный на предыдущих шагах. Убедитесь, что узел находится в состоянии *Ready* (Готово):

```output
NAME                       STATUS   ROLES   AGE     VERSION
aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.12.8
```

## <a name="create-an-azure-database-for-postgresql---flexible-server"></a>Создание гибкого сервера Базы данных Azure для PostgreSQL
Создайте гибкий сервер с помощью команды [az postgreSQL flexible-server create](/cli/azure/postgres/flexible-server#az_postgres_flexible_server_create). Следующая команда создает сервер, используя параметры и значения по умолчанию для локального контекста Azure CLI:

```azurecli-interactive
az postgres flexible-server create --public-access <YOUR-IP-ADDRESS>
```

Сервер создается со следующими атрибутами:
- При первой подготовке сервера создается новая пустая база данных с именем ```postgres```. В рамках этого краткого руководства мы будем использовать именно эту базу данных.
- Автоматически созданное имя сервера, имя пользователя и пароль администратора, имя группы ресурсов (если оно не указано в локальном контексте) и расположение, которое выбрано для группы ресурсов.
- Службы по умолчанию для остальных конфигураций сервера: уровень вычислений (общего назначения), размер (уровень) вычислительных ресурсов (Standard_D2s_v3 с 2vCores), срок хранения резервных копий (7 дней) и версия PostgreSQL (12).
- Использование аргумента public-access позволяет создать сервер с открытым доступом, защищенный правилами брандмауэра. Указанный здесь IP-адрес добавляется в правило брандмауэра, чтобы разрешить доступ с клиентского компьютера.
- Так как эта команда использует локальный контекст, сервер создается в группе ресурсов ```django-project``` и в регионе ```eastus```.


## <a name="build-your-django-docker-image"></a>Создание образа Docker для Django

Создайте [приложение Django](https://docs.djangoproject.com/en/3.1/intro/) или используйте имеющийся проект Django. Убедитесь, что код находится в этой структуре папок.

```
└───my-djangoapp
    └───views.py
    └───models.py
    └───forms.py
    ├───templates
          . . . . . . .
    ├───static
         . . . . . . .
└───my-django-project
    └───settings.py
    └───urls.py
    └───wsgi.py
        . . . . . . .
    └─── Dockerfile
    └─── requirements.txt
    └─── manage.py
    
```
Обновите ```ALLOWED_HOSTS``` в файле ```settings.py```, чтобы приложение Django использовало внешний IP-адрес, назначенный приложению Kubernetes.

```python
ALLOWED_HOSTS = ['*']
```

Обновите раздел ```DATABASES={ }``` в файле ```settings.py```. Приведенный ниже фрагмент кода позволяет получить сведения об узле базы данных, имя пользователя и пароль из файла манифеста Kubernetes.

```python
DATABASES={
   'default':{
      'ENGINE':'django.db.backends.postgresql_psycopg2',
      'NAME':os.getenv('DATABASE_NAME'),
      'USER':os.getenv('DATABASE_USER'),
      'PASSWORD':os.getenv('DATABASE_PASSWORD'),
      'HOST':os.getenv('DATABASE_HOST'),
      'PORT':'5432',
      'OPTIONS': {'sslmode': 'require'}
   }
}
```

### <a name="generate-a-requirementstxt-file"></a>Создание файла requirements.txt
Создайте файл ```requirements.txt```, чтобы вывести список зависимостей для приложения Django. Ниже приведен пример файла ```requirements.txt```. Файл requirements.txt для имеющегося приложения можно создать с использованием [``` pip freeze > requirements.txt```](https://pip.pypa.io/en/stable/reference/pip_freeze/).

``` text
Django==2.2.17
postgres==3.0.0
psycopg2-binary==2.8.6
psycopg2-pool==1.1
pytz==2020.4
```

### <a name="create-a-dockerfile"></a>Создание файла Dockerfile
Создайте файл с именем ```Dockerfile``` и скопируйте в него приведенный ниже фрагмент кода. Этот файл Dockerfile предназначен для настройки Python 3.8 и установки всех требований, перечисленных в файле requirements.txt.

```docker
# Use the official Python image from the Docker Hub
FROM python:3.8.2

# Make a new directory to put our code in.
RUN mkdir /code

# Change the working directory.
WORKDIR /code

# Copy to code folder
COPY . /code/

# Install the requirements.
RUN pip install -r requirements.txt

# Run the application:
CMD python manage.py runserver 0.0.0.0:8000
```

### <a name="build-your-image"></a>Сборка образа
В терминале убедитесь, что вы находитесь в каталоге ```my-django-app```, выполнив команду ```cd```. Выполните следующую команду, чтобы создать образ для доски объявлений:

``` bash

docker build --tag myblog:latest .

```

Разверните этот образ в [Центре Docker](https://docs.docker.com/get-started/part3/#create-a-docker-hub-repository-and-push-your-image) или в [Реестре контейнеров Azure](../../container-registry/container-registry-get-started-azure-cli.md).

> [!IMPORTANT]
>Если вы используете Реестр контейнеров Azure (ACR), выполните команду ```az aks update```, чтобы присоединить учетную запись ACR к данным в кластере AKS.
>
>```azurecli-interactive
>az aks update -n myAKSCluster -g django-project --attach-acr <your-acr-name>
> ```
>

## <a name="create-kubernetes-manifest-file"></a>Создание файла манифеста Kubernetes

Файл манифеста Kubernetes определяет требуемое состояние для кластера, включая образы контейнеров, которые нужно запустить. Создайте файл манифеста ```djangoapp.yaml``` и скопируйте в него приведенное ниже определение YAML.

>[!IMPORTANT]
> - Замените значение ```[DOCKER-HUB-USER/ACR ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]``` фактическим именем и тегом образа Docker для Django, например ```docker-hub-user/myblog:latest```.
> - Измените раздел ```env``` ниже, указав реальные значения ```SERVERNAME```, ```YOUR-DATABASE-USERNAME``` и ```YOUR-DATABASE-PASSWORD``` для гибкого сервера postgres.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: django-app
  template:
    metadata:
      labels:
        app: django-app
    spec:
      containers:
      - name: django-app
        image: [DOCKER-HUB-USER-OR-ACR-ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "SERVERNAME.postgres.database.azure.com"
        - name: DATABASE_USERNAME
          value: "YOUR-DATABASE-USERNAME"
        - name: DATABASE_PASSWORD
          value: "YOUR-DATABASE-PASSWORD"
        - name: DATABASE_NAME
          value: "postgres"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - django-app
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: v1
kind: Service
metadata:
  name: python-svc
spec:
  type: LoadBalancer
  ports:
    - port: 8000
  selector:
    app: django-app
```

## <a name="deploy-django-to-aks-cluster"></a>Развертывание Django в кластере AKS
Разверните приложение с помощью команды [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) и укажите имя манифеста YAML:

```console
kubectl apply -f djangoapp.yaml
```

В следующем примере выходных данных показано, что развертывания и службы успешно созданы.

```output
deployment "django-app" created
service "python-svc" created
```

Развертывание ```django-app``` дает возможность получить подробные сведения о развертывании, например о том, какие образы следует использовать для приложения, количество модулей pod и конфигурацию модуля pod. Будет создана служба ```python-svc```, которая открывает доступ к приложению через внешний IP-адрес.

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете. Процесс создания может занять несколько минут.

Чтобы отслеживать ход выполнения, используйте команду [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) с аргументом `--watch`.

```azurecli-interactive
kubectl get service django-app --watch
```

Изначально для параметра *EXTERNAL-IP* службы *django-app* отображается состояние *pending*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
django-app   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда значение *EXTERNAL-IP* изменится с состояния *pending* на фактический общедоступный IP-адрес, используйте команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```output
django-app  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Теперь откройте в веб-браузере внешний IP-адрес вашей службы для просмотра приложения Django.  

>[!NOTE]
> - Сейчас сайт Django не поддерживает HTTPS. Мы рекомендуем использовать [параметр ENABLE TLS и собственные сертификаты](../../aks/ingress-own-tls.md).
> - Кроме того, вы можете включить для кластера [маршрутизацию HTTP](../../aks/http-application-routing.md). Если маршрутизация HTTP включена, то она настраивает контроллер объекта ingress в кластере AKS. Когда приложения развернуты, решение также создает общедоступные DNS-имена для конечных точек приложений.

## <a name="run-database-migrations"></a>Выполнение миграций баз данных

Для любого приложения Django потребуется выполнить перенос базы данных или собрать статические файлы. Эти команды оболочки Django можно выполнить с помощью ```$ kubectl exec <pod-name> -- [COMMAND]```.  Чтобы выполнить эту команду, необходимо найти имя модуля pod с помощью ```kubectl get pods```. 

```bash
$ kubectl get pods
```

Должны отобразиться выходные данные, аналогичные указанным ниже.
```output
NAME                             READY   STATUS          RESTARTS   AGE
django-app-5d9cd6cd8-l6x4b     1/1     Running              0       2m
```

После обнаружения имени модуля pod можно выполнить перенос базы данных Django с помощью команды ```$ kubectl exec <pod-name> -- [COMMAND]```. Обратите внимание, что ```/code/``` является рабочей папкой для проекта, определенной в файле ```Dockerfile``` выше.

```bash
$ kubectl exec django-app-5d9cd6cd8-l6x4b -- python /code/manage.py migrate
```

Выходные данные будут выглядеть следующим образом. 
```output 
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  . . . . . . 
```

Если возникли проблемы, запустите ```kubectl logs <pod-name>```, чтобы узнать, какое исключение вызывает приложение. Если все успешно работает, при запуске ```kubectl logs``` отобразится результат, подобный следующему.

```output
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
December 08, 2020 - 23:24:14
Django version 2.2.17, using settings 'django_postgres_app.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

## <a name="clean-up-the-resources"></a>очищать ресурсы.

Чтобы избежать расходов за использование Azure, необходимо удалить ненужные ресурсы.  Чтобы удалить ненужные кластер, группу ресурсов, службу контейнеров и все связанные с ней ресурсы, выполните команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
az group delete --name django-project --yes --no-wait
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями](../../aks/kubernetes-service-principal.md#additional-considerations). Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [открыть веб-панель мониторинга Kubernetes](../../aks/kubernetes-dashboard.md) для кластера AKS.
- Узнайте, как [включить непрерывное развертывание](../../aks/deployment-center-launcher.md).
- Узнайте, как [масштабировать кластер](../../aks/tutorial-kubernetes-scale.md).
- Узнайте, как управлять [гибким сервером postgres](./quickstart-create-server-cli.md).
- Узнайте, как [настроить параметры сервера](./howto-configure-server-parameters-using-cli.md) для сервера базы данных.
