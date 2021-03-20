---
title: Подключение к API-интерфейсу служб мультимедиа Azure v3 — Python
description: В этой статье показано, как подключиться к API служб мультимедиа v3 с помощью Python.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/18/2020
ms.author: inhenkel
ms.custom: devx-track-python
ms.openlocfilehash: 76df8baaf170b05762b93478a496eb1e9ed802d5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94916752"
---
# <a name="connect-to-media-services-v3-api---python"></a>Подключение к API служб мультимедиа v3 — Python

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье показано, как подключиться к пакету SDK для служб мультимедиа Azure v3 для Python с помощью метода входа субъекта-службы.

## <a name="prerequisites"></a>Предварительные условия

- Скачать Python из [Python.org](https://www.python.org/downloads/)
- Обязательно задайте `PATH` переменную среды.
- [Создание учетной записи Служб мультимедиа](./create-account-howto.md). Обязательно запомните имя группы ресурсов и имя учетной записи Служб мультимедиа.
- Выполните действия, описанные в разделе [Доступ к API](./access-api-howto.md). Запишите идентификатор подписки, идентификатор приложения (идентификатор клиента), ключ проверки подлинности (секрет) и идентификатор клиента, необходимые на следующем шаге.

> [!IMPORTANT]
> Проверьте [соглашения об именовании](media-services-apis-overview.md#naming-conventions).

## <a name="install-the-modules"></a>Установка модулей

Для работы со службами мультимедиа Azure с помощью Python необходимо установить эти модули.

* `azure-mgmt-resource`Модуль, который включает модули Azure для Active Directory.
* `azure-mgmt-media`Модуль, который включает сущности служб мультимедиа.

    Обязательно получите [последнюю версию пакета SDK служб мультимедиа для Python](https://pypi.org/project/azure-mgmt-media/).

Откройте программу командной строки и используйте следующие команды для установки модулей.

```
pip3 install azure-mgmt-resource
pip3 install azure-mgmt-media==3.0.0
```

## <a name="connect-to-the-python-client"></a>Подключение к клиенту Python

1. Создание файла с `.py` расширением
1. Откройте файл в любимом редакторе.
1. Добавьте следующий код в файл. Код импортирует необходимые модули и создает объект учетных данных Active Directory, необходимый для подключения к службам мультимедиа.

      Задайте значения переменных, полученные из [API доступа](./access-api-howto.md) .

      ```
      import adal
      from msrestazure.azure_active_directory import AdalAuthentication
      from msrestazure.azure_cloud import AZURE_PUBLIC_CLOUD
      from azure.mgmt.media import AzureMediaServices
      from azure.mgmt.media.models import MediaService

      RESOURCE = 'https://management.core.windows.net/'
      # Tenant ID for your Azure Subscription
      TENANT_ID = '00000000-0000-0000-000000000000'
      # Your Service Principal App ID
      CLIENT = '00000000-0000-0000-000000000000'
      # Your Service Principal Password
      KEY = '00000000-0000-0000-000000000000'
      # Your Azure Subscription ID
      SUBSCRIPTION_ID = '00000000-0000-0000-000000000000'
      # Your Azure Media Service (AMS) Account Name
      ACCOUNT_NAME = 'amsv3account'
      # Your Resource Group Name
      RESOUCE_GROUP_NAME = 'AMSv3ResourceGroup'

      LOGIN_ENDPOINT = AZURE_PUBLIC_CLOUD.endpoints.active_directory
      RESOURCE = AZURE_PUBLIC_CLOUD.endpoints.active_directory_resource_id

      context = adal.AuthenticationContext(LOGIN_ENDPOINT + '/' + TENANT_ID)
      credentials = AdalAuthentication(
          context.acquire_token_with_client_credentials,
          RESOURCE,
          CLIENT,
          KEY
      )

      # The AMS Client
      # You can now use this object to perform different operations to your AMS account.
      client = AzureMediaServices(credentials, SUBSCRIPTION_ID)

      print("signed in")

      # Now that you are authenticated, you can manipulate the entities.
      # For example, list assets in your AMS account
      print (client.assets.list(RESOUCE_GROUP_NAME, ACCOUNT_NAME).get(0))
      ```

1. Запуск файла

## <a name="next-steps"></a>Дальнейшие действия

- Используйте [пакет SDK для Python](https://aka.ms/ams-v3-python-sdk).
- Ознакомьтесь со справочной документации по [Python](/python/api/overview/azure/mediaservices/management) Служб мультимедиа.
