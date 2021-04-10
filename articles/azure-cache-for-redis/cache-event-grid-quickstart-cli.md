---
title: Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью Azure CLI
description: Сведения об использовании Сетки событий Azure для создания подписки на события Кэша Azure для Redis, активации такого события и просмотра полученных результатов.
author: curib
ms.author: cauribeg
ms.date: 1/5/2021
ms.topic: quickstart
ms.service: cache
ms.openlocfilehash: 7f33ca0043400962054fabb1aadb1da612fe5426
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99806432"
---
# <a name="quickstart-route-azure-cache-for-redis-events-to-web-endpoint-with-azure-cli"></a>Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью Azure CLI

"Сетка событий Azure" — это служба обработки событий для облака. В этом кратком руководстве показано, как с помощью Azure CLI подписаться на события Кэша Azure для Redis, активировать такое событие и просмотреть полученные результаты.

Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. Для простоты в этом кратком руководстве описана отправка событий непосредственно в веб-приложение, которое будет собирать и отображать эти сообщения. Выполнив описанные в этом кратком руководстве действия, вы увидите, что данные событий отправлены в веб-приложение.

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этим кратким руководством вам понадобится Azure CLI последней версии, то есть 2.0.70 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Если вы не используете Cloud Shell, сначала выполните вход с помощью `az login`.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Разделы Сетки событий развертываются как отдельные ресурсы Azure и должны быть подготовлены в группе ресурсов Azure. Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). 

В следующем примере создается группа ресурсов с именем `<resource_group_name>` в расположении *westcentralus*.  Замените `<resource_group_name>` уникальным именем для группы ресурсов.

```azurecli-interactive
az group create --name <resource_group_name> --location westcentralus
```

## <a name="create-a-cache-instance"></a>Создание экземпляра кэша

```azurecli-interactive
#/bin/bash

# Create a Basic C0 (256 MB) Azure Cache for Redis instance
az redis create --name <cache_name> --resource-group <resource_group_name> --location westcentralus --sku Basic --vm-size C0
```


## <a name="create-a-message-endpoint"></a>Создание конечной точки сообщения

Перед подпиской на раздел необходимо создать конечную точку для сообщения о событии. Обычно конечная точка выполняет действия на основе данных событий. Чтобы упростить работу с этим руководством, разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

Замените `<your-site-name>` уникальным именем для вашего веб-приложения. Имя веб-приложения должно быть уникальным, так как оно включается в запись DNS.

```azurecli-interactive
sitename=<your-site-name>

az group deployment create \
  --resource-group <resource_group_name> \
  --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
  --parameters siteName=$sitename hostingPlanName=viewerhost
```

Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

Вы увидите сайт, на котором сейчас не отображаются никакие сообщения.

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-your-azure-cache-for-redis-instance"></a>Подписка на экземпляр Кэша Azure для Redis

На этом шаге вы создадите подписку на раздел, чтобы Сетке событий было известно, какие события вы намерены отслеживать и куда их следует отправлять. В следующем примере создается подписка на экземпляр Кэша Azure для Redis, который вы создали ранее, и передается URL-адрес веб-приложения в качестве конечной точки для уведомления о событиях. Замените `<event_subscription_name>` именем подписки на событие. Для `<resource_group_name>` и `<cache_name>` используйте созданные ранее значения.

Конечная точка веб-приложения должна содержать суффикс `/api/updates/`.

```azurecli-interactive
cacheId=$(az redis show --name <cache_name> --resource-group <resource_group_name> --query id --subscription <subscription_id>)
endpoint=https://$sitename.azurewebsites.net/api/updates

az eventgrid event-subscription create \
  --source-resource-id $cacheId \
  --name <event_subscription_name> \
  --endpoint $endpoint
```

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Щелкните значок с изображением глаза, чтобы развернуть данные события. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Веб-приложение содержит код для проверки подписки.

  :::image type="content" source="media/cache-event-grid-portal/subscription-event.png" alt-text="Средство просмотра Сетки событий Azure.":::

## <a name="trigger-an-event-from-azure-cache-for-redis"></a>Активация события из Кэша Azure для Redis

Теперь необходимо активировать событие, чтобы увидеть, как Сетка событий Azure распределяет сообщение к вашей конечной точке. Давайте экспортируем данные, которые хранятся в экземпляре Кэша Azure для Redis. Здесь нам снова потребуются значения `{cache_name}` и `{resource_group_name}`, созданные ранее.

```azurecli-interactive
az redis export  --ids '/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}' \
    --prefix '<prefix_for_exported_files>' \
    --container '<SAS_url>'  
```

Вы активировали событие, а служба "Сетка событий" отправила сообщение в конечную точку, настроенную вами при оформлении подписки. Откройте веб-приложение и просмотрите в нем отправленные события.


```json
[{
"id": "e1ceb52d-575c-4ce4-8056-115dec723cff",
  "eventType": "Microsoft.Cache.ExportRDBCompleted",
  "topic": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
  "data": {
    "name": "ExportRDBCompleted",
    "timestamp": "2020-12-10T18:07:54.4937063+00:00",
    "status": "Succeeded"
  },
  "subject": "ExportRDBCompleted",
  "dataversion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-12-10T18:07:54.4937063+00:00"
}]

```

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы планируете продолжать работу с этим экземпляром Кэша Azure для Redis и с этой подпиской на события, не удаляйте ресурсы, созданные при работе с этим кратким руководством. Если вы не планируете продолжать работу, удалите все созданные в этом кратком руководстве ресурсы.

Замените `<resource_group_name>` именем группы ресурсов, созданной ранее.

```azurecli-interactive
az group delete --name <resource_group_name>
```

## <a name="next-steps"></a>Следующие шаги

Теперь, когда вы знаете, как создавать разделы и подписки на события, ознакомьтесь с дополнительными сведениями о событиях Кэша Azure для Redis и возможностях службы "Сетка событий".

- [Реагирование на события Кэша Azure для Redis](cache-event-grid.md)
- [An introduction to Azure Event Grid](../event-grid/overview.md) (Общие сведения о службе "Сетка событий Azure")
