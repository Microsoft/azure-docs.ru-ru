---
title: Использование GitHub Actions для развертывания статического сайта в службе хранилища Azure
description: Размещение статического веб-сайта службы хранилища Azure с помощью действий GitHub
author: juliakm
ms.service: storage
ms.topic: how-to
ms.author: jukullam
ms.reviewer: dineshm
ms.date: 01/11/2021
ms.subservice: blobs
ms.custom: devx-track-javascript, github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 8e5bb7bffddaedbf6349611c17e8d9cbec3c4139
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102172686"
---
# <a name="set-up-a-github-actions-workflow-to-deploy-your-static-website-in-azure-storage"></a>Настройка рабочего процесса GitHub Actions для развертывания статического веб-сайта в службе хранилища Azure

Начните работу с [действиями GitHub](https://docs.github.com/en/actions) , используя рабочий процесс для развертывания статического сайта в учетной записи хранения Azure. После настройки рабочего процесса для действий GitHub вы сможете автоматически развернуть сайт в Azure из GitHub при внесении изменений в код сайта.

> [!NOTE]
> Если вы используете [статические веб-приложения Azure](../../static-web-apps/index.yml), вам не нужно вручную настраивать рабочий процесс для действий GitHub.
> Статические веб-приложения Azure автоматически создают рабочий процесс для действий GitHub. 

## <a name="prerequisites"></a>Предварительные требования

Подписка Azure и учетная запись GitHub. 

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Репозиторий GitHub со статическим кодом веб-сайта. Если у вас нет учетной записи GitHub, [зарегистрируйтесь бесплатно](https://github.com/join).  
- Рабочий статический веб-сайт, размещенный в службе хранилища Azure. Сведения о [размещении статического веб-сайта в службе хранилища Azure](storage-blob-static-website-how-to.md). Для выполнения этого примера следует также развернуть [Azure CDN](static-website-content-delivery-network.md).

> [!NOTE]
> Обычно сеть доставки содержимого (CDN) используется для уменьшения задержки пользователей по всему миру и уменьшения количества транзакций в учетной записи хранения. Развертывание статического содержимого в облачной службе хранилища может снизить потребность в потенциально дорогостоящем вычислительном экземпляре. Дополнительные сведения см. в разделе [шаблон размещения статического содержимого](/azure/architecture/patterns/static-content-hosting).

## <a name="generate-deployment-credentials"></a>Создание учетных данных для развертывания.

Вы можете создать [субъект-службу](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) в [Azure CLI](/cli/azure/). Чтобы выполнить эту команду, откройте [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или нажмите кнопку **Попробовать**.

Замените заполнитель `myStaticSite` именем сайта, размещенным в службе хранилища Azure. 

```azurecli-interactive
   az ad sp create-for-rbac --name {myStaticSite} --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} --sdk-auth
```

В указанном выше примере замените заполнители своим идентификатором подписки и именем группы ресурсов. Выходные данные — это объект JSON с учетными данными назначения роли, которые предоставляют доступ к учетной записи хранения, как показано ниже. Скопируйте этот объект JSON для последующего использования.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Рекомендуется всегда предоставлять минимальные разрешения доступа. Область в предыдущем примере ограничена конкретным приложением службы приложений, а не всей группой ресурсов.

## <a name="configure-the-github-secret"></a>Настройка секрета GitHub

1. В [GitHub](https://github.com/) найдите нужный репозиторий.

1. Выберите **Settings > Secrets > New secret** (Параметры > Секреты > Новый секрет).

1. Вставьте все выходные данные JSON, полученные из команды Azure CLI, в поле значения секрета. Присвойте секрету имя, например `AZURE_CREDENTIALS` .

    Когда вы позже настроите файл рабочего процесса, этот секрет будет передан в действие входа в Azure как параметр `creds`. Пример:

    ```yaml
    - uses: azure/login@v1
    with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

## <a name="add-your-workflow"></a>Добавление рабочего процесса

1. Перейдите в раздел **Actions** (Действия) для репозитория GitHub. 

    :::image type="content" source="media/storage-blob-static-website/storage-blob-github-actions-header.png" alt-text="Пункт меню действий GitHub":::

1. Выберите **Set up your workflow yourself** (Настроить рабочий процесс самостоятельно). 

1. Удалите все содержимое в файле рабочего процесса после раздела `on:`. После этого рабочий процесс должен выглядеть примерно так. 

    ```yaml
    name: CI

    on:
        push:
            branches: [ master ]
        pull_request:
            branches: [ master ]
    ```

1. Присвойте рабочему процессу имя `Blob storage website CI` и добавьте действия для извлечения и входа. Эти действия извлекают код сайта из репозитория и выполняют проверку подлинности в Azure с помощью ранее созданного секрета GitHub `AZURE_CREDENTIALS`. 

    ```yaml
    name: Blob storage website CI

    on:
        push:
            branches: [ master ]
        pull_request:
            branches: [ master ]

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:            
        - uses: actions/checkout@v2
        - uses: azure/login@v1
          with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

1. Используйте действие Azure CLI, чтобы передать код в хранилище BLOB-объектов и очистить конечную точку CDN. Для `az storage blob upload-batch` Замените заполнитель именем своей учетной записи хранения. Скрипт будет отправлен в `$web` контейнер. Для `az cdn endpoint purge` Замените заполнители именем профиля CDN, именем конечной точки CDN и группой ресурсов.

    ```yaml
        - name: Upload to blob storage
          uses: azure/CLI@v1
          with:
            azcliversion: 2.0.72
            inlineScript: |
                az storage blob upload-batch --account-name <STORAGE_ACCOUNT_NAME> -d '$web' -s .
        - name: Purge CDN endpoint
          uses: azure/CLI@v1
          with:
            azcliversion: 2.0.72
            inlineScript: |
               az cdn endpoint purge --content-paths  "/*" --profile-name "CDN_PROFILE_NAME" --name "CDN_ENDPOINT" --resource-group "RESOURCE_GROUP"
    ``` 

1. Завершите создание рабочего процесса, добавив действие для выхода из Azure. Готовый рабочий процесс выглядит так. В папке `.github/workflows` репозитория появится новый файл.

    ```yaml
    name: Blob storage website CI

    on:
        push:
            branches: [ master ]
        pull_request:
            branches: [ master ]

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:            
        - uses: actions/checkout@v2
        - uses: azure/login@v1
          with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}

        - name: Upload to blob storage
          uses: azure/CLI@v1
          with:
            azcliversion: 2.0.72
            inlineScript: |
                az storage blob upload-batch --account-name <STORAGE_ACCOUNT_NAME> -d '$web' -s .
        - name: Purge CDN endpoint
          uses: azure/CLI@v1
          with:
            azcliversion: 2.0.72
            inlineScript: |
               az cdn endpoint purge --content-paths  "/*" --profile-name "CDN_PROFILE_NAME" --name "CDN_ENDPOINT" --resource-group "RESOURCE_GROUP"
      
      # Azure logout 
        - name: logout
          run: |
                az logout
    ```

## <a name="review-your-deployment"></a>Проверка развертывания

1. Перейдите в раздел **Actions** (Действия) для репозитория GitHub. 

1. Откройте первый результат, чтобы проверить подробные журналы выполнения рабочего процесса. 
 
    :::image type="content" source="../media/index/github-actions-run.png" alt-text="Журнал выполнения действий GitHub Actions":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если статический веб-сайт и репозиторий GitHub больше не требуются, очистите развернутые ресурсы, удалив группу ресурсов и репозиторий GitHub. 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Дополнительные сведения о статических веб-приложениях Azure](../../static-web-apps/index.yml)
