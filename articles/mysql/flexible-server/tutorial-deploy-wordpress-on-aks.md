---
title: Руководство по развертыванию WordPress в кластере AKS с Гибким сервером MySQL с помощью Azure CLI
description: Сведения о том, как быстро создать и развернуть WordPress в AKS с Гибким сервером Базы данных Azure для MySQL.
ms.service: mysql
author: mksuni
ms.author: sumuth
ms.topic: tutorial
ms.date: 11/25/2020
ms.custom: vc, devx-track-azurecli
ms.openlocfilehash: 0c6211f4cd647addd6f1d18a153695d16a9d9952
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770189"
---
# <a name="tutorial-deploy-wordpress-app-on-aks-with-azure-database-for-mysql---flexible-server"></a>Руководство по Развертывание приложения WordPress в AKS с Гибким сервером Базы данных Azure для MySQL

В рамках этого краткого руководства вы развернете приложение WordPress в кластере Azure Kubernetes Service (AKS) с Гибким сервером Базы данных Azure для MySQL (предварительная версия) с помощью Azure CLI. 
**[AKS](../../aks/intro-kubernetes.md)**  — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. **[Гибкий сервер Базы данных Azure для MySQL (предварительная версия)](overview.md)**  — это полностью управляемая служба базы данных, которая повышает точность управления, а также гибкость функций управления базами данных и параметров конфигурации. Сейчас предоставляется предварительная версия Гибкого сервера.

> [!NOTE]
> - Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.
> - В этом кратком руководстве предполагается, что у вас есть базовое представление о понятиях Kubernetes, WordPress и MySQL.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется последняя версия Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!NOTE]
> Если команды из этого краткого руководства выполняются в локальной среде (а не в Azure Cloud Shell), убедитесь, что они выполняются от имени администратора.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure — это логическая группа, в которой развертываются и управляются ресурсы Azure. Давайте создадим группу ресурсов *WordPress-Project* в расположении *eastus* с помощью команды [az group create] [az-group-create].

```azurecli-interactive
az group create --name wordpress-project --location eastus
```

> [!NOTE]
> Расположением для группы ресурсов называется место хранения метаданных для этой группы ресурсов. Также в нем выполняются ресурсы Azure, если вы не укажете другой регион во время создания конкретного ресурса.

В следующем примере выходных данных показано, что группа ресурсов успешно создана:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/wordpress-project",
  "location": "eastus",
  "managedBy": null,
  "name": "wordpress-project",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-aks-cluster"></a>Создание кластера AKS

Используйте команду [az aks create](/cli/azure/aks#az_aks_create), чтобы создать кластер AKS. В следующем примере создается кластер *myAKSCluster* с одним узлом. Это может занять несколько минут.

```azurecli-interactive
az aks create --resource-group wordpress-project --name myAKSCluster --node-count 1 --generate-ssh-keys
```

Через несколько минут выполнение команды завершается и отображаются сведения о кластере в формате JSON.

> [!NOTE]
> При создании кластера AKS для хранения ресурсов AKS автоматически создается вторая группа ресурсов. См. раздел [Почему с AKS создаются две группы ресурсов?](../../aks/faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Управлять кластером Kubernetes можно c помощью [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Чтобы установить `kubectl` локально, используйте команду [az aks install-cli](/cli/azure/aks#az_aks_install_cli):

```azurecli-interactive
az aks install-cli
```

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials](/cli/azure/aks#az_aks_get_credentials). Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group wordpress-project --name myAKSCluster
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

## <a name="create-an-azure-database-for-mysql---flexible-server"></a>Создание Гибкого сервера Базы данных Azure для MySQL
Создайте Гибкий сервер с помощью команды [az mysql flexible-server create](/cli/azure/mysql/flexible-server). Следующая команда создает сервер, используя параметры и значения по умолчанию для локального контекста Azure CLI:

```azurecli-interactive
az mysql flexible-server create --public-access <YOUR-IP-ADDRESS>
```

Сервер создается со следующими атрибутами:
- При первой подготовке сервера создается новая пустая база данных с именем ```flexibleserverdb```. В рамках этого краткого руководства мы будем использовать именно эту базу данных.
- Автоматически созданное имя сервера, имя пользователя и пароль администратора, имя группы ресурсов (если оно не указано в локальном контексте) и расположение, которое выбрано для группы ресурсов.
- Службы по умолчанию для остальных конфигураций сервера: уровень вычислений (с накапливаемыми ресурсами), размер (уровень) вычислительных ресурсов или номер SKU (B1MS), срок хранения резервных копий (7 дней) и версия MySQL (5.7).
- Использование аргумента public-access позволяет создать сервер с открытым доступом, защищенный правилами брандмауэра. Указанный здесь IP-адрес добавляется в правило брандмауэра, чтобы разрешить доступ с клиентского компьютера.
- Так как эта команда использует локальный контекст, сервер создается в группе ресурсов ```wordpress-project``` и в регионе ```eastus```.


### <a name="build-your-wordpress-docker-image"></a>Создание образа Docker для WordPress

Скачайте [последнюю версию WordPress](https://wordpress.org/download/). Создайте для своего проекта новый каталог ```my-wordpress-app``` с представленной здесь несложной структурой папок.

```
└───my-wordpress-app
    └───public
        ├───wp-admin
        │   ├───css
        . . . . . . .
        ├───wp-content
        │   ├───plugins
        . . . . . . .
        └───wp-includes
        . . . . . . .
        ├───wp-config-sample.php
        ├───index.php
        . . . . . . .
    └─── Dockerfile

```


Переименуйте ```wp-config-sample.php``` в ```wp-config.php``` и замените строки 21–32 предложенным здесь фрагментом кода. Приведенный ниже фрагмент кода позволяет получить сведения об узле базы данных, имя пользователя и пароль из файла манифеста Kubernetes.

```php
//Using environment variables for DB connection information

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */

$connectstr_dbhost = getenv('DATABASE_HOST');
$connectstr_dbusername = getenv('DATABASE_USERNAME');
$connectstr_dbpassword = getenv('DATABASE_PASSWORD');

/** MySQL database name */
define('DB_NAME', 'flexibleserverdb');

/** MySQL database username */
define('DB_USER', $connectstr_dbusername);

/** MySQL database password */
define('DB_PASSWORD',$connectstr_dbpassword);

/** MySQL hostname */
define('DB_HOST', $connectstr_dbhost);

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');


/** SSL*/
define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
```

### <a name="create-a-dockerfile"></a>Создание файла Dockerfile
Создайте новый файл Dockerfile и скопируйте в него этот фрагмент кода. Этот Dockerfile настроит веб-сервер Apache с PHP и включит расширение mysqli.

```docker
FROM php:7.2-apache
COPY public/ /var/www/html/
RUN docker-php-ext-install mysqli
RUN docker-php-ext-enable mysqli
```

### <a name="build-your-docker-image"></a>Создание образа Docker
В терминале убедитесь, что вы находитесь в каталоге ```my-wordpress-app```, выполнив команду ```cd```. Запустите сборку образа с помощью следующей команды.

``` bash

docker build --tag myblog:latest .

```

Разверните этот образ в [Центре Docker](https://docs.docker.com/get-started/part3/#create-a-docker-hub-repository-and-push-your-image) или в [Реестре контейнеров Azure](../../container-registry/container-registry-get-started-azure-cli.md).

> [!IMPORTANT]
>Если вы используете Реестр контейнеров Azure (ACR), выполните команду ```az aks update```, чтобы присоединить учетную запись ACR к данным в кластере AKS.
>
>```azurecli-interactive
>az aks update -n myAKSCluster -g wordpress-project --attach-acr <your-acr-name>
> ```
>

## <a name="create-kubernetes-manifest-file"></a>Создание файла манифеста Kubernetes

Файл манифеста Kubernetes определяет требуемое состояние для кластера, включая образы контейнеров, которые нужно запустить. Создайте файл манифеста `mywordpress.yaml` и скопируйте в него приведенное ниже определение YAML.

>[!IMPORTANT]
> - Замените значение ```[DOCKER-HUB-USER/ACR ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]``` фактическим именем и тегом образа Docker для WordPress, например ```docker-hub-user/myblog:latest```.
> - Измените раздел ```env``` ниже, указав реальные значения ```SERVERNAME```, ```YOUR-DATABASE-USERNAME``` и ```YOUR-DATABASE-PASSWORD``` для Гибкого сервера MySQL.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress-blog
  template:
    metadata:
      labels:
        app: wordpress-blog
    spec:
      containers:
      - name: wordpress-blog
        image: [DOCKER-HUB-USER-OR-ACR-ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "SERVERNAME.mysql.database.azure.com"
        - name: DATABASE_USERNAME
          value: "YOUR-DATABASE-USERNAME"
        - name: DATABASE_PASSWORD
          value: "YOUR-DATABASE-PASSWORD"
        - name: DATABASE_NAME
          value: "flexibleserverdb"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - wordpress-blog
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: v1
kind: Service
metadata:
  name: php-svc
spec:
  type: LoadBalancer
  ports:
    - port: 80
  selector:
    app: wordpress-blog
```

## <a name="deploy-wordpress-to-aks-cluster"></a>Развертывание WordPress в кластере AKS
Разверните приложение с помощью команды [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) и укажите имя манифеста YAML:

```console
kubectl apply -f mywordpress.yaml
```

В следующем примере выходных данных показано, что развертывания и службы успешно созданы.

```output
deployment "wordpress-blog" created
service "php-svc" created
```

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете. Процесс создания может занять несколько минут.

Чтобы отслеживать ход выполнения, используйте команду [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) с аргументом `--watch`.

```azurecli-interactive
kubectl get service wordpress-blog --watch
```

Изначально для параметра *EXTERNAL-IP* службы *wordpress-blog* отображается состояние *pending*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress-blog   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда значение *EXTERNAL-IP* изменится с состояния *pending* на фактический общедоступный IP-адрес, используйте команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```output
wordpress-blog  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

### <a name="browse-wordpress"></a>Просмотр WordPress

Введите в веб-браузере внешний IP-адрес службы, чтобы открыть страницу установки WordPress.

   :::image type="content" source="./media/tutorial-deploy-wordpress-on-aks/wordpress-aks-installed-success.png" alt-text="Система WordPress успешно установлена в AKS с Гибким сервером MySQL":::

>[!NOTE]
> - Сейчас сайт WordPress не поддерживает HTTPS. Мы рекомендуем использовать [параметр ENABLE TLS и собственные сертификаты](../../aks/ingress-own-tls.md).
> - Кроме того, вы можете включить для кластера [маршрутизацию HTTP](../../aks/http-application-routing.md).

## <a name="clean-up-the-resources"></a>очищать ресурсы.

Чтобы избежать расходов за использование Azure, необходимо удалить ненужные ресурсы.  Чтобы удалить ненужные кластер, группу ресурсов, службу контейнеров и все связанные с ней ресурсы, выполните команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
az group delete --name wordpress-project --yes --no-wait
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями](../../aks/kubernetes-service-principal.md#additional-considerations). Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [открыть веб-панель мониторинга Kubernetes](../../aks/kubernetes-dashboard.md) для кластера AKS.
- Узнайте, как [масштабировать кластер](../../aks/tutorial-kubernetes-scale.md).
- Узнайте, как управлять [Гибким сервером MySQL](./quickstart-create-server-cli.md).
- Узнайте, как [настроить параметры сервера](./how-to-configure-server-parameters-cli.md) для сервера базы данных.
