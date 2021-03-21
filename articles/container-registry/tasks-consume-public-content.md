---
title: Рабочий процесс задачи для управления общим содержимым реестра
description: Создайте автоматизированный рабочий процесс "задачи реестра контейнеров Azure" для мониторинга, управления и использования содержимого общедоступного образа в частном реестре контейнеров Azure.
author: SteveLasker
ms.topic: article
ms.author: stevelas
ms.date: 10/29/2020
ms.custom: ''
ms.openlocfilehash: 4fba6290b4973e797c13943fc9be4fadb19f3274
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96349288"
---
# <a name="how-to-consume-and-maintain-public-content-with-azure-container-registry-tasks"></a>Как использовать и поддерживать общедоступное содержимое с помощью задач реестра контейнеров Azure

В этой статье приведен пример рабочего процесса в реестре контейнеров Azure, который помогает управлять использованием и обслуживанием общедоступного содержимого:

1. Импорт локальных копий зависимых общедоступных образов.
1. Проверка общедоступных образов с помощью проверки безопасности и функционального тестирования.
1. Передвигайте образы в закрытые реестры для внутреннего использования.
1. Активация обновлений базовых образов для приложений, зависящих от общедоступного содержимого.
1. Используйте [задачи реестра контейнеров Azure](container-registry-tasks-overview.md) для автоматизации этого рабочего процесса.

На следующем рисунке показана сводка рабочего процесса.

![Использование рабочего процесса общего содержимого](./media/tasks-consume-public-content/consuming-public-content-workflow.png)

Рабочий процесс с условным импортом помогает управлять зависимостями Организации от внешних управляемых артефактов, например, с помощью образов, источником которых являются общедоступные реестры, включая [DOCKER Hub][docker-hub], [ГКР][gcr], [Quay][quay], [Реестр контейнеров GitHub][ghcr], [Реестр контейнеров Майкрософт][mcr]или даже другие реестры [контейнеров Azure][acr]. 

Дополнительные сведения о рисках, появившихся в зависимости от общедоступного содержимого, и об использовании реестра контейнеров Azure для их устранения см. в [блоге о потреблении общедоступного содержимого][oci-consuming-public-content] и [управлении общедоступным содержимым с помощью реестра контейнеров Azure](buffer-gate-public-content.md).

Для выполнения этого пошагового руководства можно использовать Azure Cloud Shell или локальную установку Azure CLI. Рекомендуется Azure CLI версии 2,10 или более поздней. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-cli].

## <a name="scenario-overview"></a>Обзор сценария

![Импорт компонентов рабочего процесса](./media/tasks-consume-public-content/consuming-public-content-objects.png)

В этом пошаговом руководстве настраивается следующее:

1. Три **реестра контейнеров**, представляющие:
   * Имитируемый [центр DOCKER][docker-hub] ( `publicregistry` ) для поддержки изменения базового образа
   * Реестр команды ( `contoso` ) для совместного использования частных образов
   * Общий реестр компании или команды `baseartifacts` для импортированного общедоступного содержимого
1. **Задача контроля** доступа в каждом реестре. Задачи:  
   1. Создание имитации общего `node` образа
   1. Импорт и проверка `node` образа в общем реестре компании или команды
   1. Сборка и развертывание `hello-world` образа
1. **Определения задач записи контроля** доступа, включая конфигурации для:
1. Коллекция **учетных данных реестра**, которые являются указателями на хранилище ключей.
1. Коллекция **секретов**, доступных в `acr-task.yaml` , которые являются указателями на хранилище ключей
1. Коллекция **настроенных значений** , используемых в элементе `acr-task.yaml`
1. **Хранилище ключей Azure** для защиты всех секретов
1. **Экземпляр контейнера Azure**, в котором размещается `hello-world` приложение сборки.

## <a name="prerequisites"></a>Предварительные условия

Следующие шаги используются для настройки значений ресурсов, создаваемых и используемых в этом пошаговом руководстве.

### <a name="set-environment-variables"></a>Настройка переменных среды

Настройте переменные, уникальные для вашей среды. Мы соблюдаем рекомендации по размещению ресурсов с устойчивым содержимым в собственной группе ресурсов, чтобы избежать случайного удаления. Однако при необходимости их можно поместить в одну группу ресурсов.

Примеры в этой статье форматируются для оболочки bash.

```bash
# Set the three registry names, must be globally unique:
REGISTRY_PUBLIC=publicregistry
REGISTRY_BASE_ARTIFACTS=contosobaseartifacts
REGISTRY=contoso

# set the location all resources will be created in:
RESOURCE_GROUP_LOCATION=eastus

# default resource groups
REGISTRY_PUBLIC_RG=${REGISTRY_PUBLIC}-rg
REGISTRY_BASE_ARTIFACTS_RG=${REGISTRY_BASE_ARTIFACTS}-rg
REGISTRY_RG=${REGISTRY}-rg

# fully qualified registry urls
REGISTRY_DOCKERHUB_URL=docker.io
REGISTRY_PUBLIC_URL=${REGISTRY_PUBLIC}.azurecr.io
REGISTRY_BASE_ARTIFACTS_URL=${REGISTRY_BASE_ARTIFACTS}.azurecr.io
REGISTRY_URL=${REGISTRY}.azurecr.io

# Azure key vault for storing secrets, name must be globally unique
AKV=acr-task-credentials
AKV_RG=${AKV}-rg

# ACI for hosting the deployed application
ACI=hello-world-aci
ACI_RG=${ACI}-rg
```

### <a name="git-repositories-and-tokens"></a>Репозитории и токены Git

Для имитации среды разветвление каждой из следующих репозиториев Git в репозитории, которыми вы можете управлять. 

* https://github.com/importing-public-content/base-image-node.git
* https://github.com/importing-public-content/import-baseimage-node.git
* https://github.com/importing-public-content/hello-world.git

Затем обновите следующие переменные для разветвленных репозиториев.

Элемент, `:main` добавленный к концу URL-адресов Git, представляет ветвь репозитория по умолчанию.

```bash
GIT_BASE_IMAGE_NODE=https://github.com/<your-fork>/base-image-node.git#main
GIT_NODE_IMPORT=https://github.com/<your-fork>/import-baseimage-node.git#main
GIT_HELLO_WORLD=https://github.com/<your-fork>/hello-world.git#main
```

Для клонирования и установки веб-перехватчиков Git необходим [маркер доступа GitHub (PAT)][git-token] для задач контроля доступа. Инструкции по созданию маркера с необходимыми разрешениями для частного репозитория см. в разделе [Создание маркера доступа GitHub](container-registry-tutorial-build-task.md#create-a-github-personal-access-token). 

```bash
GIT_TOKEN=<set-git-token-here>
```

### <a name="docker-hub-credentials"></a>Учетные данные DOCKER HUB  
Чтобы избежать запросов на регулирование и идентификацию при получении образов из DOCKER Hub, создайте [токен DOCKER Hub][docker-hub-tokens]. Затем задайте следующие переменные среды:

```bash
REGISTRY_DOCKERHUB_USER=<yourusername>
REGISTRY_DOCKERHUB_PASSWORD=<yourtoken>
```

### <a name="create-registries"></a>Создание реестров

С помощью Azure CLIных команд создайте три реестра контейнеров уровня Premium, каждый в отдельной группе ресурсов:

```azurecli-interactive
az group create --name $REGISTRY_PUBLIC_RG --location $RESOURCE_GROUP_LOCATION
az acr create --resource-group $REGISTRY_PUBLIC_RG --name $REGISTRY_PUBLIC --sku Premium

az group create --name $REGISTRY_BASE_ARTIFACTS_RG --location $RESOURCE_GROUP_LOCATION
az acr create --resource-group $REGISTRY_BASE_ARTIFACTS_RG --name $REGISTRY_BASE_ARTIFACTS --sku Premium

az group create --name $REGISTRY_RG --location $RESOURCE_GROUP_LOCATION
az acr create --resource-group $REGISTRY_RG --name $REGISTRY --sku Premium
```

### <a name="create-key-vault-and-set-secrets"></a>Создание хранилища ключей и задание секретов

Создайте хранилище ключей:

```azurecli-interactive
az group create --name $AKV_RG --location $RESOURCE_GROUP_LOCATION
az keyvault create --resource-group $AKV_RG --name $AKV
```

Задайте имя пользователя и токен DOCKER Hub в хранилище ключей:

```azurecli-interactive
az keyvault secret set \
--vault-name $AKV \
--name registry-dockerhub-user \
--value $REGISTRY_DOCKERHUB_USER

az keyvault secret set \
--vault-name $AKV \
--name registry-dockerhub-password \
--value $REGISTRY_DOCKERHUB_PASSWORD
```

Установите и проверьте Git PAT в хранилище ключей:

```azurecli-interactive
az keyvault secret set --vault-name $AKV --name github-token --value $GIT_TOKEN

az keyvault secret show --vault-name $AKV --name github-token --query value -o tsv
```

### <a name="create-resource-group-for-an-azure-container-instance"></a>Создание группы ресурсов для экземпляра контейнера Azure

Эта группа ресурсов используется в более поздней задаче при развертывании `hello-world` образа.

```azurecli-interactive
az group create --name $ACI_RG --location $RESOURCE_GROUP_LOCATION
```

## <a name="create-public-node-base-image"></a>Создать общедоступный `node` базовый образ

Чтобы имитировать `node` образ в DOCKER Hub, создайте [задачу контроля][acr-task] доступа для создания и обслуживания общедоступного образа. Эта настройка позволяет имитировать изменения в процессе `node` обслуживания изображений.

```azurecli-interactive
az acr task create \
  --name node-public \
  -r $REGISTRY_PUBLIC \
  -f acr-task.yaml \
  --context $GIT_BASE_IMAGE_NODE \
  --git-access-token $(az keyvault secret show \
                        --vault-name $AKV \
                        --name github-token \
                        --query value -o tsv) \
  --set REGISTRY_FROM_URL=${REGISTRY_DOCKERHUB_URL}/ \
  --assign-identity
```

Чтобы избежать регулирования DOCKER, добавьте в задачу [учетные данные центра DOCKER][docker-hub-tokens] . Команду [учетных данных задачи записи контроля][acr-task-credentials] доступа можно использовать для передачи учетных данных DOCKER в любой реестр, включая DOCKER Hub.

```azurecli-interactive
az acr task credential add \
  -n node-public \
  -r $REGISTRY_PUBLIC \
  --login-server $REGISTRY_DOCKERHUB_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-dockerhub-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-dockerhub-password \
  --use-identity [system]
```

Предоставьте задаче доступ для чтения значений из хранилища ключей:

```azurecli-interactive
az keyvault set-policy \
  --name $AKV \
  --resource-group $AKV_RG \
  --object-id $(az acr task show \
                  --name node-public \
                  --registry $REGISTRY_PUBLIC \
                  --query identity.principalId --output tsv) \
  --secret-permissions get
```

[Задачи можно активировать][acr-task-triggers] с помощью фиксаций Git, обновлений базовых образов, таймеров или ручных запусков. 

Запустите задачу вручную, чтобы создать `node` образ:

```azurecli-interactive
az acr task run -r $REGISTRY_PUBLIC -n node-public
```

Перечислите изображение в имитируемом общедоступном реестре:

```azurecli-interactive
az acr repository show-tags -n $REGISTRY_PUBLIC --repository node
```

## <a name="create-the-hello-world-image"></a>Создание `hello-world` образа

Создайте образ на основе имитации общего `node` образа `hello-world` .

### <a name="create-token-for-pull-access-to-simulated-public-registry"></a>Создание маркера для доступа по запросу к имитируемому общедоступному реестру

Создайте [маркер доступа][acr-tokens] к имитируемому общедоступному реестру в области `pull` . Затем задайте его в хранилище ключей:

```azurecli-interactive
az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_PUBLIC}-user" \
  --value "registry-${REGISTRY_PUBLIC}-user"

az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_PUBLIC}-password" \
  --value $(az acr token create \
              --name "registry-${REGISTRY_PUBLIC}-user" \
              --registry $REGISTRY_PUBLIC \
              --scope-map _repositories_pull \
              -o tsv \
              --query credentials.passwords[0].value)
```

### <a name="create-token-for-pull-access-by-azure-container-instances"></a>Создание маркера для доступа по запросу для службы "экземпляры контейнеров Azure"

Создайте [маркер доступа][acr-tokens] к реестру, в котором размещается `hello-world` образ, с областью действия Pull. Затем задайте его в хранилище ключей:

```azurecli-interactive
az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY}-user" \
  --value "registry-${REGISTRY}-user"

az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY}-password" \
  --value $(az acr token create \
              --name "registry-${REGISTRY}-user" \
              --registry $REGISTRY \
              --repository hello-world content/read \
              -o tsv \
              --query credentials.passwords[0].value)
```

### <a name="create-task-to-build-and-maintain-hello-world-image"></a>Создание задачи для сборки и обслуживания `hello-world` образа

Следующая команда создает задачу из определения в `acr-tasks.yaml` в `hello-world` репозитории. Шаги задачи создают `hello-world` образ, а затем развертывают его в службе "экземпляры контейнеров Azure". Группа ресурсов для службы "экземпляры контейнеров Azure" была создана в предыдущем разделе. Вызывая `az container create` в задаче только разницу в `image:tag` , задача развертывается в том же экземпляре в рамках этого пошагового руководства.

```azurecli-interactive
az acr task create \
  -n hello-world \
  -r $REGISTRY \
  -f acr-task.yaml \
  --context $GIT_HELLO_WORLD \
  --git-access-token $(az keyvault secret show \
                        --vault-name $AKV \
                        --name github-token \
                        --query value -o tsv) \
  --set REGISTRY_FROM_URL=${REGISTRY_PUBLIC_URL}/ \
  --set KEYVAULT=$AKV \
  --set ACI=$ACI \
  --set ACI_RG=$ACI_RG \
  --assign-identity
```

Добавьте учетные данные в задачу для имитации общедоступного реестра:

```azurecli-interactive
az acr task credential add \
  -n hello-world \
  -r $REGISTRY \
  --login-server $REGISTRY_PUBLIC_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-password \
  --use-identity [system]
```

Предоставьте задаче доступ для чтения значений из хранилища ключей:

```azurecli-interactive
az keyvault set-policy \
  --name $AKV \
  --resource-group $AKV_RG \
  --object-id $(az acr task show \
                  --name hello-world \
                  --registry $REGISTRY \
                  --query identity.principalId --output tsv) \
  --secret-permissions get
```

Предоставьте задачам доступ для создания и управления экземплярами контейнеров Azure, предоставив доступ к группе ресурсов:

```azurecli-interactive
az role assignment create \
  --assignee $(az acr task show \
  --name hello-world \
  --registry $REGISTRY \
  --query identity.principalId --output tsv) \
  --scope $(az group show -n $ACI_RG --query id -o tsv) \
  --role owner
```

После создания и настройки задачи запустите задачу для сборки и развертывания `hello-world` образа.

```azurecli-interactive
az acr task run -r $REGISTRY -n hello-world
```

После создания получите IP-адрес контейнера, в котором размещается `hello-world` образ.

```azurecli-interactive
az container show \
  --resource-group $ACI_RG \
  --name ${ACI} \
  --query ipAddress.ip \
  --out tsv
```

В браузере перейдите по IP-адресу, чтобы увидеть работающее приложение.

## <a name="update-the-base-image-with-a-questionable-change"></a>Обновление базового образа с "сомнительным" изменением

В этом разделе имитируется изменение базового образа, которое может вызвать проблемы в среде.

1. Откройте `Dockerfile` в разветвленном `base-image-node` репозитории.
1. Измените на, чтобы `BACKGROUND_COLOR` `Orange` имитировать изменение.

```Dockerfile
ARG REGISTRY_NAME=
FROM ${REGISTRY_NAME}node:15-alpine
ENV NODE_VERSION 15-alpine
ENV BACKGROUND_COLOR Orange
```

Зафиксируйте изменения и следите за задачами контроля доступа, чтобы автоматически начать создание.

Следите за тем, чтобы задача начала выполняться:

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_PUBLIC -o table
```

Со временем вы должны увидеть состояние `Succeeded` на основе триггера `Commit` :

```azurecli-interactive
RUN ID    TASK      PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------  ----------  ---------  ---------  --------------------  ----------
ca4       hub-node  linux       Succeeded  Commit     2020-10-24T05:02:29Z  00:00:22
```

Введите **CTRL + C** , чтобы выйти из команды Watch, а затем просмотрите журналы для последнего запуска:

```azurecli-interactive
az acr task logs -r $REGISTRY_PUBLIC
```

После `node` завершения образа `watch` для задач записи контроля доступа автоматически приступит к созданию `hello-world` образа:

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY -o table
```

Со временем вы должны увидеть состояние `Succeeded` на основе триггера `Image Update` :

```azurecli-interactive
RUN ID    TASK         PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  -----------  ----------  ---------  ------------  --------------------  ----------
dau       hello-world  linux       Succeeded  Image Update  2020-10-24T05:08:45Z  00:00:31
```

Введите **CTRL + C** , чтобы выйти из команды Watch, а затем просмотрите журналы для последнего запуска:

```azurecli-interactive
az acr task logs -r $REGISTRY
```

После завершения получите IP-адрес сайта, на котором размещен обновленный `hello-world` образ:

```azurecli-interactive
az container show \
  --resource-group $ACI_RG \
  --name ${ACI} \
  --query ipAddress.ip \
  --out tsv
```

В браузере перейдите на сайт, который должен иметь оранжевый (сомнительный) фон.

### <a name="checking-in"></a>Возврат

На этом этапе вы создали `hello-world` образ, который автоматически создается на основе фиксаций Git и изменений в базовом `node` образе. В этом примере задача выполняет сборку на базовом образе в реестре контейнеров Azure, но может использоваться любой поддерживаемый реестр.

Обновление базового образа автоматически вызывает запуск задачи при `node` обновлении образа. Как видно здесь, не все обновления нужны.

## <a name="gated-imports-of-public-content"></a>Условные импорты общедоступного содержимого

Чтобы предотвратить появление вышестоящего изменения критически важных рабочих нагрузок, можно добавить проверку безопасности и функциональные тесты.

В этом разделе вы создадите задачу контроля учетных записей для:

* Создание тестового образа
* Запуск скрипта функционального теста `./test.sh` для тестового образа
* Если изображение успешно проверяется, импортируйте общедоступный образ в реестр **басеимажес**

### <a name="add-automation-testing"></a>Добавление тестирования автоматизации

Для вентиля любого вышестоящего содержимого реализуется автоматическое тестирование. В этом примере указывается, `test.sh` который проверяет `$BACKGROUND_COLOR` . Если тест завершается ошибкой, `EXIT_CODE` возвращается исключение из, `1` что приводит к сбою шага задачи записи контроля доступа, завершением выполнения задачи. Тесты можно разворачивать в любой форме инструментов, включая результаты ведения журнала. Шлюз управляется ответом "успех/сбой" в скрипте, который создается следующим образом:

```bash
if [ ""$(echo $BACKGROUND_COLOR | tr '[:lower:]' '[:upper:]') = 'RED' ]; then
    echo -e "\e[31mERROR: Invalid Color:\e[0m" ${BACKGROUND_COLOR}
    EXIT_CODE=1
else
  echo -e "\e[32mValidation Complete - No Known Errors\e[0m"
fi
exit ${EXIT_CODE}
```
### <a name="task-yaml"></a>YAML задачи 

Проверьте `acr-task.yaml` в `import-baseimage-node` репозитории, который выполняет следующие действия:

1. Создайте базовый образ теста, используя следующие Dockerfile:
    ```dockerfile
    ARG REGISTRY_FROM_URL=
    FROM ${REGISTRY_FROM_URL}node:15-alpine
    WORKDIR /test
    COPY ./test.sh .
    CMD ./test.sh
    ```
1. По завершении проверьте образ, запустив контейнер, в котором выполняется `./test.sh`
1. Только если успешно завершено, выполните шаги импорта, которые включены в `when: ['validate-base-image']`

```yaml
version: v1.1.0
steps:
  - id: build-test-base-image
    # Build off the base image we'll track
    # Add a test script to do unit test validations
    # Note: the test validation image isn't saved to the registry
    # but the task logs captures log validation results
    build: >
      --build-arg REGISTRY_FROM_URL={{.Values.REGISTRY_FROM_URL}}
      -f ./Dockerfile
      -t {{.Run.Registry}}/node-import:test
      .
  - id: validate-base-image
    # only continues if node-import:test returns a non-zero code
    when: ['build-test-base-image']
    cmd: "{{.Run.Registry}}/node-import:test"
  - id: pull-base-image
    # import the public image to base-artifacts
    # Override the stable tag,
    # and create a unique tag to enable rollback
    # to a previously working image
    when: ['validate-base-image']
    cmd: >
        docker pull {{.Values.REGISTRY_FROM_URL}}node:15-alpine
  - id: retag-base-image
    when: ['pull-base-image']
    cmd: docker tag {{.Values.REGISTRY_FROM_URL}}node:15-alpine {{.Run.Registry}}/node:15-alpine
  - id: retag-base-image-unique-tag
    when: ['pull-base-image']
    cmd: docker tag {{.Values.REGISTRY_FROM_URL}}node:15-alpine {{.Run.Registry}}/node:15-alpine-{{.Run.ID}}
  - id: push-base-image
    when: ['retag-base-image', 'retag-base-image-unique-tag']
    push:
    - "{{.Run.Registry}}/node:15-alpine"
    - "{{.Run.Registry}}/node:15-alpine-{{.Run.ID}}"
```

### <a name="create-task-to-import-and-test-base-image"></a>Создание задачи для импорта и тестирования базового образа

```azurecli-interactive
  az acr task create \
  --name base-import-node \
  -f acr-task.yaml \
  -r $REGISTRY_BASE_ARTIFACTS \
  --context $GIT_NODE_IMPORT \
  --git-access-token $(az keyvault secret show \
                        --vault-name $AKV \
                        --name github-token \
                        --query value -o tsv) \
  --set REGISTRY_FROM_URL=${REGISTRY_PUBLIC_URL}/ \
  --assign-identity
```

Добавьте учетные данные в задачу для имитации общедоступного реестра:

```azurecli-interactive
az acr task credential add \
  -n base-import-node \
  -r $REGISTRY_BASE_ARTIFACTS \
  --login-server $REGISTRY_PUBLIC_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_PUBLIC}-password \
  --use-identity [system]
```

Предоставьте задаче доступ для чтения значений из хранилища ключей:

```azurecli-interactive
az keyvault set-policy \
  --name $AKV \
  --resource-group $AKV_RG \
  --object-id $(az acr task show \
                  --name base-import-node \
                  --registry $REGISTRY_BASE_ARTIFACTS \
                  --query identity.principalId --output tsv) \
  --secret-permissions get
```

Выполните задачу импорта:

```azurecli-interactive
az acr task run -n base-import-node -r $REGISTRY_BASE_ARTIFACTS
```

> [!NOTE]
> Если задача завершается ошибкой из-за `./test.sh: Permission denied` , убедитесь, что скрипт имеет разрешения на выполнение, и выполните фиксацию в репозитории Git:
>```bash
>chmod +x ./test.sh
>```

## <a name="update-hello-world-image-to-build-from-gated-node-image"></a>Обновить `hello-world` образ для сборки из условного `node` образа

Создайте [маркер доступа][acr-tokens] для доступа к реестру базовых артефактов в пределах `read` `node` репозитория. Затем задайте в хранилище ключей:

```azurecli-interactive
az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_BASE_ARTIFACTS}-user" \
  --value "registry-${REGISTRY_BASE_ARTIFACTS}-user"

az keyvault secret set \
  --vault-name $AKV \
  --name "registry-${REGISTRY_BASE_ARTIFACTS}-password" \
  --value $(az acr token create \
              --name "registry-${REGISTRY_BASE_ARTIFACTS}-user" \
              --registry $REGISTRY_BASE_ARTIFACTS \
              --repository node content/read \
              -o tsv \
              --query credentials.passwords[0].value)
```

Добавьте учетные данные в задачу **Hello-World** для базового реестра артефактов:

```azurecli-interactive
az acr task credential add \
  -n hello-world \
  -r $REGISTRY \
  --login-server $REGISTRY_BASE_ARTIFACTS_URL \
  -u https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_BASE_ARTIFACTS}-user \
  -p https://${AKV}.vault.azure.net/secrets/registry-${REGISTRY_BASE_ARTIFACTS}-password \
  --use-identity [system]
```

Обновите задачу, чтобы изменить `REGISTRY_FROM_URL` для использования `BASE_ARTIFACTS` реестра.

```azurecli-interactive
az acr task update \
  -n hello-world \
  -r $REGISTRY \
  --set KEYVAULT=$AKV \
  --set REGISTRY_FROM_URL=${REGISTRY_BASE_ARTIFACTS_URL}/ \
  --set ACI=$ACI \
  --set ACI_RG=$ACI_RG
```

Выполните задачу **Hello-World** , чтобы изменить ее зависимость базового образа:

```azurecli-interactive
az acr task run -r $REGISTRY -n hello-world
```

## <a name="update-the-base-image-with-a-valid-change"></a>Обновление базового образа с помощью "допустимого" изменения

1. Откройте `Dockerfile` в `base-image-node` репозитории.
1. Измените `BACKGROUND_COLOR` на, `Green` чтобы имитировать допустимое изменение.

```Dockerfile
ARG REGISTRY_NAME=
FROM ${REGISTRY_NAME}node:15-alpine
ENV NODE_VERSION 15-alpine
ENV BACKGROUND_COLOR Green
```

Зафиксируйте изменения и отслеживайте последовательность обновлений:

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_PUBLIC -o table
```

После запуска введите **CTRL + C** и отслеживайте журналы:

```azurecli-interactive
az acr task logs -r $REGISTRY_PUBLIC
```

По завершении выполните мониторинг задачи **базового образа** :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_BASE_ARTIFACTS -o table
```

После запуска введите **CTRL + C** и отслеживайте журналы:

```azurecli-interactive
az acr task logs -r $REGISTRY_BASE_ARTIFACTS
```

По завершении выполните мониторинг задачи **Hello-World** :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY -o table
```

После запуска введите **CTRL + C** и отслеживайте журналы:

```azurecli-interactive
az acr task logs -r $REGISTRY
```

После завершения получите IP-адрес сайта, на котором размещен обновленный `hello-world` образ:

```azurecli-interactive
az container show \
  --resource-group $ACI_RG \
  --name ${ACI} \
  --query ipAddress.ip \
  --out tsv
```

В браузере перейдите на сайт, который должен иметь зеленый (допустимый) фон.

### <a name="view-the-gated-workflow"></a>Просмотр рабочего процесса с проверкой изменений

Снова выполните действия, описанные в предыдущем разделе, с красным цветом фона.

1. Открытие `Dockerfile` в `base-image-node` репозитории
1. Измените `BACKGROUND_COLOR` на, `Red` чтобы имитировать недопустимое изменение.

```Dockerfile
ARG REGISTRY_NAME=
FROM ${REGISTRY_NAME}node:15-alpine
ENV NODE_VERSION 15-alpine
ENV BACKGROUND_COLOR Red
```

Зафиксируйте изменения и отслеживайте последовательность обновлений:

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_PUBLIC -o table
```

После запуска введите **CTRL + C** и отслеживайте журналы:

```azurecli-interactive
az acr task logs -r $REGISTRY_PUBLIC
```

По завершении выполните мониторинг задачи **базового образа** :

```azurecli-interactive
watch -n1 az acr task list-runs -r $REGISTRY_BASE_ARTIFACTS -o table
```

После запуска введите **CTRL + C** и отслеживайте журналы:

```azurecli-interactive
az acr task logs -r $REGISTRY_BASE_ARTIFACTS
```

На этом этапе вы должны увидеть, что задача " **Базовый-импорт-узел** " не проходит проверку, и останавливает последовательность для публикации `hello-world` обновления. Она выводит выходные данные следующего вида:

```console
[...]
2020/10/30 03:57:39 Launching container with name: validate-base-image
Validating Image
NODE_VERSION: 15-alpine
BACKGROUND_COLOR: Red
ERROR: Invalid Color: Red
2020/10/30 03:57:40 Container failed during run: validate-base-image. No retries remaining.
failed to run step ID: validate-base-image: exit status 1
```

### <a name="publish-an-update-to-hello-world"></a>Публикация обновления в `hello-world`

Изменения `hello-world` образа будут продолжать использовать Последнее проверенное `node` изображение.

Любые дополнительные изменения в базовом `node` образе, которые проходят проверки с условным обозначением, будут активировать обновления базового образа для `hello-world` образа.

## <a name="cleaning-up"></a>Очистка

Удалите ресурсы, используемые в этой статье, если они больше не нужны.

```azurecli-interactive
az group delete -n $REGISTRY_RG --no-wait -y
az group delete -n $REGISTRY_PUBLIC_RG --no-wait -y
az group delete -n $REGISTRY_BASE_ARTIFACTS_RG --no-wait -y
az group delete -n $AKV_RG --no-wait -y
az group delete -n $ACI_RG --no-wait -y
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье. Вы использовали задачи контроля доступа, чтобы создать рабочий процесс автоматической ограничения, чтобы добавить в среду обновленные базовые образы. См. Дополнительные сведения об управлении образами в реестре контейнеров Azure.


* [Рекомендации по отстановке тегов и управления версиями для образов контейнеров](container-registry-image-tag-version.md)
* [Блокировка образа контейнера в реестре контейнеров Azure](container-registry-image-lock.md)

[install-cli]:                  /cli/azure/install-azure-cli
[acr]:                          https://aka.ms/acr
[acr-repo-permissions]:         ./container-registry-repository-scoped-permissions.md
[acr-task]:                     ./container-registry-tasks-overview.md
[acr-task-triggers]:            container-registry-tasks-overview.md#task-scenarios
[acr-task-credentials]:       container-registry-tasks-authentication-managed-identity.md#4-optional-add-credentials-to-the-task
[acr-tokens]:                   ./container-registry-repository-scoped-permissions.md
[aci]:                          https://aka.ms/aci
[alpine-public-image]:          https://hub.docker.com/_/alpine
[docker-hub]:                   https://hub.docker.com
[docker-hub-tokens]:            https://hub.docker.com/settings/security
[git-token]:                    https://github.com/settings/tokens
[gcr]:                          https://cloud.google.com/container-registry
[ghcr]:                         https://docs.github.com/en/free-pro-team@latest/packages/getting-started-with-github-container-registry/about-github-container-registry
[helm-charts]:                  https://helm.sh
[mcr]:                          https://aka.ms/mcr
[nginx-public-image]:           https://hub.docker.com/_/nginx
[oci-artifacts]:                ./container-registry-oci-artifacts.md
[oci-consuming-public-content]: https://opencontainers.org/posts/blog/2020-10-30-consuming-public-content/
[opa]:                          https://www.openpolicyagent.org/
[quay]:                         https://quay.io