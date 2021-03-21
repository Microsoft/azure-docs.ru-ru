---
title: Проверка подлинности между реестрами с помощью задачи ACR
description: Настройка задачи Реестра контейнеров Azure (задача ACR) для доступа к другому частному реестру контейнеров в Azure с использованием управляемого удостоверения для ресурсов Azure
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: 789d2c141f8b7c3f2eb8daa31d99090e3d028a43
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98915834"
---
# <a name="cross-registry-authentication-in-an-acr-task-using-an-azure-managed-identity"></a>Проверка подлинности между реестрами с помощью задачи ACR с использованием удостоверения, управляемого Azure 

В [задаче ACR](container-registry-tasks-overview.md) можно [включить управляемое удостоверение для ресурсов Azure](container-registry-tasks-authentication-managed-identity.md). В задаче это удостоверение будет использоваться для получения доступа к другим ресурсам Azure без необходимости предоставлять учетные данные или управлять ими. 

Из этой статьи вы узнаете, как включить управляемое удостоверение в задаче для извлечения образа из реестра, отличающегося от того, который использовался для выполнения задачи.

Для создания ресурсов Azure с помощью инструкций из этой статьи требуется Azure CLI версии 2.0.68 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli].

## <a name="scenario-overview"></a>Обзор сценария

В примере задачи базовый образ извлекается из другого реестра контейнеров в Azure для создания и отправки образа приложения. Чтобы извлечь базовый образ, необходимо настроить задачу с использованием управляемого удостоверения и назначить соответствующие разрешения. 

В этом примере описаны шаги, в которых используется управляемое удостоверение, назначаемое системой или пользователем. Выбор удостоверения зависит от требований организации.

В реальной ситуации у организации может быть набор базовых образов, который все команды разработчиков используют для создания приложений. Эти базовые образы хранятся в корпоративном реестре, и у каждой команды разработчиков есть только права на извлечение. 

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей вам понадобится два реестра контейнеров Azure:

* Первый реестр используется для создания и выполнения задач ACR. В этой статье ему присвоено имя *myregistry*. 
* Во втором реестре размещается базовый образ, используемый в задаче для создания образа. Имя второго реестра в этой статье — *mybaseregistry*. 

На следующих шагах замените эти значения именами ваших реестров.

Если у вас еще нет необходимых реестров контейнеров Azure, см. статью [Краткое руководство по созданию закрытого реестра контейнеров с помощью Azure CLI](container-registry-get-started-azure-cli.md). На этом этапе не нужно отправлять образы в реестр.

## <a name="prepare-base-registry"></a>Подготовка базового реестра

В демонстрационных целях в качестве одноразовой операции выполните команду [AZ запись контроля доступа] [AZ-контроля учетных записей], чтобы импортировать открытый образ Node.js из DOCKER Hub в основной реестр. На практике другая команда или процесс в Организации может поддерживать образы в основном реестре.

```azurecli
az acr import --name mybaseregistry \
  --source docker.io/library/node:15-alpine \
  --image baseimages/node:15-alpine 
```

## <a name="define-task-steps-in-yaml-file"></a>Определение шагов задачи в файле YAML

В этом примере шаги для [многоэтапной задачи](container-registry-tasks-multi-step.md) определяются в [файле YAML](container-registry-tasks-reference-yaml.md). Создайте файл с именем `helloworldtask.yaml` в локальной рабочей папке и вставьте в него следующее содержимое. Измените значение `REGISTRY_NAME` на этапе создания, указав имя сервера в базовом реестре.

```yml
version: v1.1.0
steps:
# Replace mybaseregistry with the name of your registry containing the base image
  - build: -t $Registry/hello-world:$ID  https://github.com/Azure-Samples/acr-build-helloworld-node.git#main -f Dockerfile-app --build-arg REGISTRY_NAME=mybaseregistry.azurecr.io
  - push: ["$Registry/hello-world:$ID"]
```

Для создания образа используется файл `Dockerfile-app` в репозитории [Azure-Samples/acr-build-helloworld-node](https://github.com/Azure-Samples/acr-build-helloworld-node.git). `--build-arg` ссылается на базовый реестр для извлечения базового образа. Созданный образ отправляется в реестр, используемый для выполнения задачи.

## <a name="option-1-create-task-with-user-assigned-identity"></a>Вариант 1. Создание задачи с удостоверением, назначаемым пользователем

В этом разделе описано, как создать задачу и включить удостоверение, назначаемое пользователем. Если вы хотите включить удостоверение, назначаемое системой, см. раздел [Вариант 2. Создание задачи с удостоверением, назначаемым системой](#option-2-create-task-with-system-assigned-identity). 

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="create-task"></a>Создание задачи

Теперь создайте задачу *helloworldtask*, выполнив следующую команду [az acr task create][az-acr-task-create]. Задача выполняется без контекста исходного кода, а команда ссылается на файл `helloworldtask.yaml` в рабочей папке. Параметр `--assign-identity` передает идентификатор ресурса для удостоверения, назначаемого пользователем. 

```azurecli
az acr task create \
  --registry myregistry \
  --name helloworldtask \
  --context /dev/null \
  --file helloworldtask.yaml \
  --assign-identity $resourceID
```

[!INCLUDE [container-registry-tasks-user-id-properties](../../includes/container-registry-tasks-user-id-properties.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Предоставление удостоверению разрешений на извлечение данных из базового реестра

В этом разделе описано, как предоставить управляемому удостоверению разрешения на извлечение данных из базового реестра *mybaseregistry*.

Используйте команду [az acr show][az-acr-show], чтобы получить идентификатор ресурса для базового реестра и сохранить его в переменной.

```azurecli
baseregID=$(az acr show --name mybaseregistry --query id --output tsv)
```

Используйте команду [az role assignment create][az-role-assignment-create], чтобы назначить удостоверению роль `acrpull` для доступа к базовому реестру. Эта роль имеет разрешения только на извлечение образов из реестра.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

Перейдите к [разделу Добавление учетных данных целевого реестра в задачу](#add-target-registry-credentials-to-task).

## <a name="option-2-create-task-with-system-assigned-identity"></a>Вариант 2. Создание задачи с удостоверением, назначаемым системой

В этом разделе описано, как создать задачу и включить удостоверение, назначаемое системой. Если вы хотите включить удостоверение, назначаемое пользователем, см. раздел [Вариант 1. Создание задачи с удостоверением, назначаемым пользователем](#option-1-create-task-with-user-assigned-identity). 

### <a name="create-task"></a>Создание задачи

Теперь создайте задачу *helloworldtask*, выполнив следующую команду [az acr task create][az-acr-task-create]. Задача выполняется без контекста исходного кода, а команда ссылается на файл `helloworldtask.yaml` в рабочей папке. Параметр `--assign-identity` без значений включает удостоверение, назначаемое системой, в задаче. 

```azurecli
az acr task create \
  --registry myregistry \
  --name helloworldtask \
  --context /dev/null \
  --file helloworldtask.yaml \
  --assign-identity 
```
[!INCLUDE [container-registry-tasks-system-id-properties](../../includes/container-registry-tasks-system-id-properties.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Предоставление удостоверению разрешений на извлечение данных из базового реестра

В этом разделе описано, как предоставить управляемому удостоверению разрешения на извлечение данных из базового реестра *mybaseregistry*.

Используйте команду [az acr show][az-acr-show], чтобы получить идентификатор ресурса для базового реестра и сохранить его в переменной.

```azurecli
baseregID=$(az acr show --name mybaseregistry --query id --output tsv)
```

Используйте команду [az role assignment create][az-role-assignment-create], чтобы назначить удостоверению роль `acrpull` для доступа к базовому реестру. Эта роль имеет разрешения только на извлечение образов из реестра.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

## <a name="add-target-registry-credentials-to-task"></a>Добавление учетных данных целевого реестра в задачу

Теперь с помощью команды [az acr task credential add][az-acr-task-credential-add] разрешите задачам выполнять проверку подлинности в базовом реестре с использованием учетных данных удостоверения. Выполните команду в соответствии с типом управляемого удостоверения, включенного в задаче. Если вы включили удостоверение, назначаемое пользователем, передайте `--use-identity` с идентификатором клиента удостоверения. Если вы включили удостоверение, назначаемое системой, передайте `--use-identity [system]`.

```azurecli
# Add credentials for user-assigned identity to the task
az acr task credential add \
  --name helloworldtask \
  --registry myregistry \
  --login-server mybaseregistry.azurecr.io \
  --use-identity $clientID

# Add credentials for system-assigned identity to the task
az acr task credential add \
  --name helloworldtask \
  --registry myregistry \
  --login-server mybaseregistry.azurecr.io \
  --use-identity [system]
```

## <a name="manually-run-the-task"></a>Выполнение задачи вручную

Чтобы проверить, выполняется ли задача, в которой включено управляемое удостоверение, запустите ее вручную с помощью команды [az acr task run][az-acr-task-run]. 

```azurecli
az acr task run \
  --name helloworldtask \
  --registry myregistry
```

Если задача выполняется успешно, выходные данные будут выглядеть примерно так:

```
Queued a run with ID: cf10
Waiting for an agent...
2019/06/14 22:47:32 Using acb_vol_dbfbe232-fd76-4ca3-bd4a-687e84cb4ce2 as the home volume
2019/06/14 22:47:39 Creating Docker network: acb_default_network, driver: 'bridge'
2019/06/14 22:47:40 Successfully set up Docker network: acb_default_network
2019/06/14 22:47:40 Setting up Docker configuration...
2019/06/14 22:47:41 Successfully set up Docker configuration
2019/06/14 22:47:41 Logging in to registry: myregistry.azurecr.io
2019/06/14 22:47:42 Successfully logged into myregistry.azurecr.io
2019/06/14 22:47:42 Logging in to registry: mybaseregistry.azurecr.io
2019/06/14 22:47:43 Successfully logged into mybaseregistry.azurecr.io
2019/06/14 22:47:43 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/14 22:47:43 Scanning for dependencies...
2019/06/14 22:47:45 Successfully scanned dependencies
2019/06/14 22:47:45 Launching container with name: acb_step_0
Sending build context to Docker daemon   25.6kB
Step 1/6 : ARG REGISTRY_NAME
Step 2/6 : FROM ${REGISTRY_NAME}/baseimages/node:15-alpine
15-alpine: Pulling from baseimages/node
[...]
Successfully built 41b49a112663
Successfully tagged myregistry.azurecr.io/hello-world:cf10
2019/06/14 22:47:56 Successfully executed container: acb_step_0
2019/06/14 22:47:56 Executing step ID: acb_step_1. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/14 22:47:56 Pushing image: myregistry.azurecr.io/hello-world:cf10, attempt 1
The push refers to repository [myregistry.azurecr.io/hello-world]
[...]
2019/06/14 22:48:00 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 2.517011)
2019/06/14 22:48:00 The following dependencies were found:
2019/06/14 22:48:00
- image:
    registry: myregistry.azurecr.io
    repository: hello-world
    tag: cf10
    digest: sha256:611cf6e3ae3cb99b23fadcd89fa144e18aa1b1c9171ad4a0da4b62b31b4e38d1
  runtime-dependency:
    registry: mybaseregistry.azurecr.io
    repository: baseimages/node
    tag: 15-alpine
    digest: sha256:e8e92cffd464fce3be9a3eefd1b65dc9cbe2484da31c11e813a4effc6105c00f
  git:
    git-head-revision: 0f988779c97fe0bfc7f2f74b88531617f4421643

Run ID: cf10 was successful after 32s
```

Чтобы убедиться, что образ создан и успешно отправлен в *myregistry*, выполните команду [az acr repository show-tags][az-acr-repository-show-tags].

```azurecli
az acr repository show-tags --name myregistry --repository hello-world --output tsv
```

Выходные данные примера:

```console
cf10
```

## <a name="next-steps"></a>Дальнейшие действия

* См. сведения о [включении управляемого удостоверения в задаче ACR](container-registry-tasks-authentication-managed-identity.md).
* См. статью [Справка по задачам Реестра контейнеров Azure: YAML](container-registry-tasks-reference-yaml.md).

<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az-login
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az-acr-task-credential-add
[az-group-create]: /cli/azure/group?#az-group-create
