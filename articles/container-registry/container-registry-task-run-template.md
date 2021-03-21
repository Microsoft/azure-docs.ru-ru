---
title: Быстрое выполнение задачи с помощью шаблона
description: Поставить в очередь запуск задачи записи контроля доступа для создания образа с помощью шаблона Azure Resource Manager
ms.topic: article
ms.date: 04/22/2020
ms.openlocfilehash: 6e8023c088ac328c2b6e95fccd0230c4d40325c1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98916071"
---
# <a name="run-acr-tasks-using-resource-manager-templates"></a>Выполнение задач контроля доступа с помощью шаблонов диспетчер ресурсов

[Задачи Реестра контейнеров Azure](container-registry-tasks-overview.md) — это набор функций в Реестре контейнеров Azure, которые призваны упростить управление образами контейнеров и их изменение в течение всего жизненного цикла контейнеров. 

В этой статье приведены Azure Resource Manager примеры шаблонов для постановки в очередь выполнения быстрых задач, аналогичные тому, что можно создать вручную с помощью команды [AZ запись контроля][az-acr-build] доступа.

Шаблон диспетчер ресурсов для очереди запуска задачи полезен в сценариях автоматизации и расширяет функциональные возможности `az acr build` . Пример:

* Использование шаблона для создания реестра контейнеров и немедленного постановки в очередь запуска задачи для сборки и отправки образа контейнера
* Создание или включение дополнительных ресурсов, которые можно использовать при выполнении быстрого выполнения задачи, например управляемое удостоверение для ресурсов Azure.

## <a name="limitations"></a>Ограничения

* В качестве [исходного расположения](container-registry-tasks-overview.md#context-locations) для выполнения задачи необходимо указать удаленный контекст, такой как репозиторий GitHub. Нельзя использовать локальный исходный контекст.
* Для запуска задач с использованием управляемого удостоверения разрешено только управляемое *пользователем* удостоверение.

## <a name="prerequisites"></a>Предварительные условия

* **Учетная запись GitHub** . Создайте учетную запись, https://github.com если она еще не создана. 
* **Пример репозитория разветвления** . для приведенных здесь примеров задач используйте пользовательский интерфейс GitHub, чтобы разбить следующий пример репозитория на учетную запись GitHub: https://github.com/Azure-Samples/acr-build-helloworld-node . Этот репозиторий содержит пример файлы dockerfile и исходный код для создания мелких образов контейнеров.

## <a name="example-create-registry-and-queue-task-run"></a>Пример. выполнение задачи создания реестра и очереди

В этом примере используется [Пример шаблона](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment/quickdockerbuild) для создания реестра контейнеров и постановка в очередь выполнения задачи, которая создает и отправляет изображение. 

### <a name="template-parameters"></a>Параметры шаблона

Для этого примера укажите значения для следующих параметров шаблона:

|Параметр  |Значение  |
|---------|---------|
|регистринаме     |Уникальное имя создаваемого реестра         |
|repository     |Целевой репозиторий для задачи сборки        |
|таскруннаме     |Имя запуска задачи, которое указывает тег Image |
|sourceLocation     |Удаленный контекст для задачи сборки, например https://github.com/Azure-Samples/acr-build-helloworld-node . Dockerfile в корне репозитория создает образ контейнера для небольшого Node.js веб-приложения. При необходимости используйте вилку репозитория в качестве контекста сборки.         |

### <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон с помощью команды [AZ Deployment Group Create][az-deployment-group-create] . В этом примере выполняется сборка и отправка образа *HelloWorld-Node: TestRun* в реестр с именем *миконтаинеррегистри*.

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/acr/master/docs/tasks/run-as-deployment/quickdockerbuild/azuredeploy.json \
  --parameters \
    registryName=mycontainerregistry \
    repository=helloworld-node \
    taskRunName=testrun \
    sourceLocation=https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
 ```

Предыдущая команда передает параметры в командной строке. При необходимости передайте их в [файл параметров](../azure-resource-manager/templates/parameter-files.md).

### <a name="verify-deployment"></a>Проверка развертывания

После успешного завершения развертывания убедитесь, что образ создан, выполнив команду [AZ запись репозитория-Теги][az-acr-repository-show-tags]:

```azurecli
az acr repository show-tags \
  --name mycontainerregistry \
  --repository helloworld-node --output table
```

Выходные данные:

```console
Result
--------
testrun
```

### <a name="view-run-log"></a>Просмотр журнала выполнения

Чтобы просмотреть сведения о выполнении задачи, просмотрите журнал выполнения.

Сначала получите идентификатор запуска с помощью команды [AZ контроля доступа список задач-runs][az-acr-task-list-runs] .
```azurecli
az acr task list-runs \
  --registry mycontainerregistry --output table
```

Она выводит выходные данные следующего вида:

```console
RUN ID    TASK    PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  ------  ----------  ---------  ---------  --------------------  ----------
ca1               linux       Succeeded  Manual     2020-03-23T17:54:28Z  00:00:48
```

Выполните команду [AZ контроля доступа к журналам задач][az-acr-task-logs] , чтобы просмотреть журналы выполнения задач для идентификатора запуска, в данном случае *CA1*:

```azurecli
az acr task logs \
  --registry mycontainerregistry \
  --run-id ca1
```

В выходных данных отображается журнал выполнения задач.

Журнал выполнения задач также можно просмотреть в портал Azure. 

1. Перейдите к реестру контейнеров.
2. В разделе **службы** выберите **задачи**  >  **запуски**.
3. Выберите идентификатор запуска, в данном случае *CA1*. 

На портале отображается журнал выполнения задач.

## <a name="example-task-run-with-managed-identity"></a>Пример. выполнение задачи с управляемым удостоверением

Использование [примера шаблона](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment/quickdockerbuildwithidentity) для постановки в очередь выполнения задачи, которая позволяет назначить управляемое пользователем удостоверение. Во время выполнения задачи удостоверение проверяет подлинность для извлечения образа из другого реестра контейнеров Azure. 

Этот сценарий похож на [проверку подлинности между реестром в задаче записи контроля доступа с помощью удостоверения, управляемого Azure](container-registry-tasks-cross-registry-authentication.md). Например, Организация может поддерживать централизованный реестр с базовыми изображениями, доступ к которым осуществляется несколькими группами разработчиков.

### <a name="prepare-base-registry"></a>Подготовка базового реестра

В демонстрационных целях создайте отдельный реестр контейнеров в качестве базового реестра и отправьте Node.js базовый образ, извлеченный из DOCKER Hub.

1. Создайте второй реестр контейнеров, например *мибасерегистри*, для хранения базовых образов.
1. Извлекать `node:9-alpine` образ из DOCKER Hub, разметить его для основного реестра и отправить в основной реестр:

  ```azurecli
  docker pull node:9-alpine
  docker tag node:9-alpine mybaseregistry.azurecr.io/baseimages/node:9-alpine
  az acr login -n mybaseregistry
  docker push mybaseregistry.azurecr.io/baseimages/node:9-alpine
  ```

### <a name="create-new-dockerfile"></a>Создание нового Dockerfile

Создайте Dockerfile, который извлекает базовый образ из базового реестра. Выполните следующие действия в локальной вилке репозитория GitHub, например `https://github.com/myGitHubID/acr-build-helloworld-node.git` .

1. В пользовательском интерфейсе GitHub выберите **создать новый файл**.
1. Присвойте файлу имя *Dockerfile-Test* и вставьте следующее содержимое. Подставьте имя реестра для *мибасерегистри*.
    ```
    FROM mybaseregistry.azurecr.io/baseimages/node:9-alpine
    COPY . /src
    RUN cd /src && npm install
    EXPOSE 80
    CMD ["node", "/src/server.js"]
    ```
 1. Выберите **зафиксировать новый файл**.

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Предоставление удостоверению разрешений на извлечение данных из базового реестра

Предоставьте управляемому удостоверению разрешения на извлечение из базового реестра *мибасерегистри*.

Используйте команду [az acr show][az-acr-show], чтобы получить идентификатор ресурса для базового реестра и сохранить его в переменной.

```azurecli
baseregID=$(az acr show \
  --name mybaseregistry \
  --query id --output tsv)
```

Чтобы назначить удостоверение роли Акрпулл в основном реестре, используйте команду [AZ Role назначение Create][az-role-assignment-create] . Эта роль имеет разрешения только на извлечение образов из реестра.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

### <a name="template-parameters"></a>Параметры шаблона

Для этого примера укажите значения для следующих параметров шаблона:

|Параметр  |Значение  |
|---------|---------|
|регистринаме     |Имя реестра, в котором строится образ  |
|repository     |Целевой репозиторий для задачи сборки        |
|таскруннаме     |Имя запуска задачи, которое указывает тег Image |
|усерассигнедидентити |Идентификатор ресурса назначенного пользователем удостоверения, включенного в задаче|
|кустомрегистридентити | Идентификатор клиента, которому назначено пользовательское удостоверение, которое включено в задаче и используется для проверки подлинности с помощью пользовательского реестра |
|кустомрегистри |Имя сервера входа пользовательского реестра, доступ к которому осуществляется в задаче, например *mybaseregistry.azurecr.IO*|
|sourceLocation     |Удаленный контекст для задачи сборки, например *https://github.com/ \<your-GitHub-ID\> /Акр-буилд-хелловорлд-Ноде.* |
|Параметре dockerfilepath | Путь к Dockerfile в удаленном контексте, используемый для сборки образа. |

### <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон с помощью команды [AZ Deployment Group Create][az-deployment-group-create] . В этом примере выполняется сборка и отправка образа *HelloWorld-Node: TestRun* в реестр с именем *миконтаинеррегистри*. Базовый образ извлекается из *mybaseregistry.azurecr.IO*.

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/acr/master/docs/tasks/run-as-deployment/quickdockerbuildwithidentity/azuredeploy.json \
  --parameters \
    registryName=mycontainerregistry \
    repository=helloworld-node \
    taskRunName=basetask \
    userAssignedIdentity=$resourceID \
    customRegistryIdentity=$clientID \
    sourceLocation=https://github.com/<your-GitHub-ID>/acr-build-helloworld-node.git#main \
    dockerFilePath=Dockerfile-test \
    customRegistry=mybaseregistry.azurecr.io
```

Предыдущая команда передает параметры в командной строке. При необходимости передайте их в [файл параметров](../azure-resource-manager/templates/parameter-files.md).

### <a name="verify-deployment"></a>Проверка развертывания

После успешного завершения развертывания убедитесь, что образ создан, выполнив команду [AZ запись репозитория-Теги][az-acr-repository-show-tags]:

```azurecli
az acr repository show-tags \
  --name mycontainerregistry \
  --repository helloworld-node --output table
```

Выходные данные:

```console
Result
--------
basetask
```

### <a name="view-run-log"></a>Просмотр журнала выполнения

Чтобы просмотреть журнал выполнения, см. действия, описанные в [предыдущем разделе](#view-run-log).

## <a name="next-steps"></a>Дальнейшие действия

 * Дополнительные примеры шаблонов см. в [репозитории GitHub](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment).
 * Дополнительные сведения о свойствах шаблона см. в справочнике по шаблонам для [выполнения задач](/azure/templates/microsoft.containerregistry/2019-06-01-preview/registries/taskruns) и [задач](/azure/templates/microsoft.containerregistry/2019-06-01-preview/registries/tasks).


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-logs]: /cli/azure/acr/task#az-acr-task-logs
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-identity-show]: /cli/azure/identity#az-identity-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
