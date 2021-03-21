---
title: Развертывание экземпляра контейнера по действию GitHub
description: Настройка действия GitHub, которое автоматизирует шаги по сборке, принудительной отправке и развертыванию образа контейнера в службе "экземпляры контейнеров Azure"
ms.topic: article
ms.date: 08/20/2020
ms.custom: github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 1409d8fc1430cd9bf67bd735d9826a74979d495b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98762962"
---
# <a name="configure-a-github-action-to-create-a-container-instance"></a>Настройка действия GitHub для создания экземпляра контейнера

[Действия GitHub](https://docs.github.com/en/actions) — это набор функций в GitHub для автоматизации рабочих процессов разработки программного обеспечения в том же месте, где вы храните код и работаете над запросами на вытягивание и проблемами.

Чтобы автоматизировать развертывание одного контейнера в службе "экземпляры контейнеров Azure", используйте действие GitHub " [развернуть в Azure Container](https://github.com/azure/aci-deploy) ". Это действие позволяет задать свойства для экземпляра контейнера, аналогичные тем, которые находятся в команде [AZ Container Create][az-container-create] .

В этой статье показано, как настроить рабочий процесс в репозитории GitHub, который выполняет следующие действия:

* Создание образа с помощью Dockerfile
* Отправка образа в реестр контейнеров Azure
* Развертывание образа контейнера в экземпляре контейнера Azure

В этой статье показано два способа настройки рабочего процесса.

* [Настройка рабочего процесса GitHub](#configure-github-workflow) . Создайте рабочий процесс в репозитории GitHub с помощью действия "развернуть в службе" экземпляры контейнеров Azure "и других действий.  
* [Использование расширения CLI](#use-deploy-to-azure-extension) — используйте `az container app up` команду в расширении [Deploy to Azure](https://github.com/Azure/deploy-to-azure-cli-extension) в Azure CLI. Эта команда упрощает создание рабочего процесса GitHub и этапов развертывания.

> [!IMPORTANT]
> Действие GitHub для службы "экземпляры контейнеров Azure" сейчас находится на этапе предварительной версии. Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="prerequisites"></a>Предварительные условия

* **Учетная запись GitHub** . Создайте учетную запись, https://github.com если она еще не создана.
* **Azure CLI** — для выполнения Azure CLI действий можно использовать Azure Cloud Shell или локальную установку Azure CLI. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].
* **Реестр контейнеров Azure** . Если у вас ее нет, создайте реестр контейнеров Azure на уровне "базовый" с помощью [Azure CLI](../container-registry/container-registry-get-started-azure-cli.md), [портал Azure](../container-registry/container-registry-get-started-portal.md)или других методов. Запишите группу ресурсов, используемую для развертывания, которая используется для рабочего процесса GitHub.

## <a name="set-up-repo"></a>Настройка репозитория

* В примерах, приведенных в этой статье, используйте GitHub для разветвления следующего репозитория: https://github.com/Azure-Samples/acr-build-helloworld-node

  Этот репозиторий содержит Dockerfile и исходные файлы для создания образа контейнера для небольшого веб-приложения.

  ![Снимок экрана с выделенной кнопкой "Fork" (Создать вилку) в GitHub](../container-registry/media/container-registry-tutorial-quick-build/quick-build-01-fork.png)

* Убедитесь, что для вашего репозитория включены действия. Перейдите к разветвленному репозиторию и выберите **Параметры**  >  **действия**. В списке **разрешения действий** убедитесь, что выбран **параметр включить локальные и сторонние действия для этого репозитория** .

## <a name="configure-github-workflow"></a>Настройка рабочего процесса GitHub

### <a name="create-service-principal-for-azure-authentication"></a>Создание субъекта-службы для проверки подлинности Azure

В рабочем процессе GitHub необходимо предоставить учетные данные Azure для проверки подлинности в Azure CLI. В следующем примере создается субъект-служба с ролью участника, областью действия которой является группа ресурсов для реестра контейнеров.

Сначала получите идентификатор ресурса для группы ресурсов. Замените имя группы в следующей команде [AZ Group][az-group-show] By:

```azurecli
groupId=$(az group show \
  --name <resource-group-name> \
  --query id --output tsv)
```

Для создания субъекта-службы используйте команду [AZ AD SP Create-для – RBAC][az-ad-sp-create-for-rbac] :

```azurecli
az ad sp create-for-rbac \
  --scope $groupId \
  --role Contributor \
  --sdk-auth
```

Она выводит выходные данные следующего вида:

```json
{
  "clientId": "xxxx6ddc-xxxx-xxxx-xxx-ef78a99dxxxx",
  "clientSecret": "xxxx79dc-xxxx-xxxx-xxxx-aaaaaec5xxxx",
  "subscriptionId": "xxxx251c-xxxx-xxxx-xxxx-bf99a306xxxx",
  "tenantId": "xxxx88bf-xxxx-xxxx-xxxx-2d7cd011xxxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

Сохраните выходные данные JSON, так как они используются на более позднем шаге. Кроме того, обратите внимание на `clientId` , что необходимо обновить субъект-службу в следующем разделе.

### <a name="update-service-principal-for-registry-authentication"></a>Обновление субъекта-службы для проверки подлинности реестра

Обновите учетные данные субъекта-службы Azure, чтобы разрешить принудительный и опрашивающий доступ к реестру контейнеров. Этот шаг позволяет рабочему процессу GitHub использовать субъект-службу для [проверки подлинности в реестре контейнеров](../container-registry/container-registry-auth-service-principal.md) и отправки и извлечения образа DOCKER. 

Получите идентификатор ресурса реестра контейнеров. Замените имя реестра в следующей команде [AZ запись контроля][az-acr-show] доступа:

```azurecli
registryId=$(az acr show \
  --name <registry-name> \
  --query id --output tsv)
```

Чтобы назначить роль Акрпуш, которая обеспечивает принудительный и опрашивающий доступ к реестру, используйте команду [AZ Role назначение Create][az-role-assignment-create] . Замените идентификатор клиента субъекта-службы:

```azurecli
az role assignment create \
  --assignee <ClientId> \
  --scope $registryId \
  --role AcrPush
```

### <a name="save-credentials-to-github-repo"></a>Сохранение учетных данных в репозитории GitHub

1. В пользовательском интерфейсе GitHub перейдите к разветвленному репозиторию и выберите **Параметры**  >  **секреты**. 

1. Выберите **Добавить новый секрет** , чтобы добавить следующие секреты:

|Секрет  |Значение  |
|---------|---------|
|`AZURE_CREDENTIALS`     | Все выходные данные JSON шага создания субъекта-службы |
|`REGISTRY_LOGIN_SERVER`   | Имя сервера входа в реестр (все строчные буквы). Пример: *myregistry.azurecr.IO*        |
|`REGISTRY_USERNAME`     |  `clientId`Из выходных данных JSON при создании субъекта-службы       |
|`REGISTRY_PASSWORD`     |  `clientSecret`Из выходных данных JSON при создании субъекта-службы |
| `RESOURCE_GROUP` | Имя группы ресурсов, использованной для определения субъекта-службы. |

### <a name="create-workflow-file"></a>Создать файл рабочего процесса

1. В пользовательском интерфейсе GitHub выберите **действия**  >  **новый рабочий процесс**.
1. Выберите **настроить рабочий процесс самостоятельно**.
1. В поле **изменить новый файл** вставьте следующее содержимое YAML, чтобы перезаписать образец кода. Примите имя файла по умолчанию `main.yml` или укажите имя файла, которое вы выбрали.
1. Выберите **начать фиксацию**, при необходимости укажите короткие и расширенные описания фиксации и нажмите кнопку **зафиксировать новый файл**.

```yml
on: [push]
name: Linux_Container_Workflow

jobs:
    build-and-deploy:
        runs-on: ubuntu-latest
        steps:
        # checkout the repo
        - name: 'Checkout GitHub Action'
          uses: actions/checkout@main
          
        - name: 'Login via Azure CLI'
          uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
        
        - name: 'Build and push image'
          uses: azure/docker-login@v1
          with:
            login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
            username: ${{ secrets.REGISTRY_USERNAME }}
            password: ${{ secrets.REGISTRY_PASSWORD }}
        - run: |
            docker build . -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}
            docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}

        - name: 'Deploy to Azure Container Instances'
          uses: 'azure/aci-deploy@v1'
          with:
            resource-group: ${{ secrets.RESOURCE_GROUP }}
            dns-name-label: ${{ secrets.RESOURCE_GROUP }}${{ github.run_number }}
            image: ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}
            registry-login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
            registry-username: ${{ secrets.REGISTRY_USERNAME }}
            registry-password: ${{ secrets.REGISTRY_PASSWORD }}
            name: aci-sampleapp
            location: 'west us'
```

### <a name="validate-workflow"></a>Проверка рабочего процесса

После фиксации файла рабочего процесса запускается рабочий процесс. Чтобы просмотреть ход выполнения рабочего процесса, перейдите в раздел **действия**  >  **рабочие процессы**. 

![Просмотр хода выполнения рабочего процесса](./media/container-instances-github-action/github-action-progress.png)

Сведения о просмотре состояния и результатов каждого шага в рабочем процессе см. в статье [Просмотр журнала выполнения рабочего процесса](https://docs.github.com/en/actions/managing-workflow-runs/viewing-workflow-run-history) . Если рабочий процесс не завершен, см. раздел [Просмотр журналов для диагностики сбоев](https://docs.github.com/en/actions/managing-workflow-runs/using-workflow-run-logs#viewing-logs-to-diagnose-failures).

Когда рабочий процесс завершится успешно, получите сведения об экземпляре контейнера с именем *ACI-SampleApp* , выполнив команду [AZ Container показывать][az-container-show] . Замените имя группы ресурсов: 

```azurecli
az container show \
  --resource-group <resource-group-name> \
  --name aci-sampleapp \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --output table
```

Она выводит выходные данные следующего вида:

```console
FQDN                                   ProvisioningState
---------------------------------      -------------------
aci-action01.westus.azurecontainer.io  Succeeded
```

После подготовки экземпляра перейдите к полному доменному имени контейнера в браузере, чтобы просмотреть работающее веб-приложение.

![Запуск веб-приложения в браузере](./media/container-instances-github-action/github-action-container.png)

## <a name="use-deploy-to-azure-extension"></a>Использование развертывания в расширении Azure

Кроме того, для настройки рабочего процесса можно использовать [расширение развертывание в Azure](https://github.com/Azure/deploy-to-azure-cli-extension) в Azure CLI. `az container app up`Команда в расширении принимает входные параметры, чтобы настроить рабочий процесс для развертывания в службе "экземпляры контейнеров Azure". 

Рабочий процесс, созданный Azure CLI, аналогичен рабочему процессу, который можно [создать вручную с помощью GitHub](#configure-github-workflow).

### <a name="additional-prerequisite"></a>Дополнительное необходимое условие

В дополнение к настройке [предварительных требований](#prerequisites) и [репозитория](#set-up-repo) для этого сценария необходимо установить  **расширение Deploy в Azure** для Azure CLI.

Выполните команду [AZ Extension Add][az-extension-add] , чтобы установить расширение:

```azurecli
az extension add \
  --name deploy-to-azure
```

Дополнительные сведения о поиске, установке и управлении расширениями см. [в статье Использование расширений с Azure CLI](/cli/azure/azure-cli-extensions-overview).

### <a name="run-az-container-app-up"></a>Выполнить `az container app up`

Чтобы выполнить команду [AZ Container App up][az-container-app-up] , укажите минимум:

* Имя реестра контейнеров Azure, например *myregistry*
* URL-адрес репозитория GitHub, например `https://github.com/<your-GitHub-Id>/acr-build-helloworld-node`

Пример команды:

```azurecli
az container app up \
  --acr myregistry \
  --repository https://github.com/myID/acr-build-helloworld-node
```

### <a name="command-progress"></a>Ход выполнения команд

* При появлении запроса укажите учетные данные GitHub или предоставьте [личный маркер доступа GitHub](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) (PAT), который содержит *репозиторий* и области *пользователей* для проверки подлинности в учетной записи GitHub. При предоставлении учетных данных GitHub команда создает PAT. Следуйте дополнительным запросам, чтобы настроить рабочий процесс.

* Команда создает секреты репозитория для рабочего процесса:

  * Учетные данные субъекта-службы для Azure CLI
  * Учетные данные для доступа к реестру контейнеров Azure

* После того как команда зафиксирует файл рабочего процесса в репозитории, Рабочий процесс активируется. 

Она выводит выходные данные следующего вида:

```console
[...]
Checking in file github/workflows/main.yml in the Github repository myid/acr-build-helloworld-node
Creating workflow...
GitHub Action Workflow has been created - https://github.com/myid/acr-build-helloworld-node/runs/515192398
GitHub workflow completed.
Workflow succeeded
Your app is deployed at:  http://acr-build-helloworld-node.eastus.azurecontainer.io:8080/
```

Чтобы просмотреть состояние рабочего процесса и результаты каждого шага в пользовательском интерфейсе GitHub, см. раздел [Просмотр журнала выполнения рабочего процесса](https://docs.github.com/en/actions/managing-workflow-runs/viewing-workflow-run-history).

### <a name="validate-workflow"></a>Проверка рабочего процесса

Рабочий процесс развертывает экземпляр контейнера Azure с базовым именем репозитория GitHub, в данном случае записью записи *контроля доступа — Build-HelloWorld-node*. Когда рабочий процесс завершится успешно, получите сведения об экземпляре контейнера с именем "запись *контроля доступа — сборка-HelloWorld-узел* ", выполнив команду [AZ Container показывать][az-container-show] . Замените имя группы ресурсов: 

```azurecli
az container show \
  --resource-group <resource-group-name> \
  --name acr-build-helloworld-node \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --output table
```

Она выводит выходные данные следующего вида:

```console
FQDN                                                   ProvisioningState
---------------------------------                      -------------------
acr-build-helloworld-node.westus.azurecontainer.io     Succeeded
```

После подготовки экземпляра перейдите к полному доменному имени контейнера в браузере, чтобы просмотреть работающее веб-приложение.

## <a name="clean-up-resources"></a>Очистка ресурсов

Остановите экземпляр контейнера с помощью команды [az container delete][az-container-delete]:

```azurecli
az container delete \
  --name <instance-name>
  --resource-group <resource-group-name>
```

Чтобы удалить группу ресурсов и все ресурсы в ней, выполните команду [AZ Group Delete][az-group-delete] :

```azurecli
az group delete \
  --name <resource-group-name>
```

## <a name="next-steps"></a>Дальнейшие действия

Обзор [GitHub Marketplace](https://github.com/marketplace?type=actions) для получения дополнительных действий по автоматизации рабочего процесса разработки


<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->

[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-show]: /cli/azure/group#az-group-show
[az-group-delete]: /cli/azure/group#az-group-delete
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-container-create]: /cli/azure/container#az-container-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-container-show]: /cli/azure/container#az-container-show
[az-container-delete]: /cli/azure/container#az-container-delete
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-container-app-up]: /cli/azure/ext/deploy-to-azure/container/app#ext-deploy-to-azure-az-container-app-up
