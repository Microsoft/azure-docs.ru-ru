---
title: Использовать сетку событий для уведомлений об изменении данных конфигурации приложения
description: Узнайте, как использовать подписки на события конфигурации приложений Azure для отправки событий изменения ключа в веб-конечную точку.
services: azure-app-configuration
author: AlexandraKemperMS
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: how-to
ms.date: 03/04/2020
ms.author: alkemper
ms.custom: devx-track-azurecli
ms.openlocfilehash: c188a4b7fe8e9223faa1cdeb52ae01ed83b94d84
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99549785"
---
# <a name="use-event-grid-for-app-configuration-data-change-notifications"></a>Использовать сетку событий для уведомлений об изменении данных конфигурации приложения

Из этой статьи вы узнаете, как настроить подписки на события настройки приложений Azure для отправки событий изменения значения ключа в конечную точку веб-узла. Пользователи конфигурации приложений Azure могут подписываться на события, созданные при каждом изменении значений ключей. Эти события могут активировать веб-перехватчики, функции Azure, очереди службы хранилища Azure или любой другой обработчик событий, поддерживаемый службой "Сетка событий Azure". Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. Но в этой статье для простоты события отправляются в веб-приложение, которое собирает и отображает сообщения.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/). При желании вы также можете использовать Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать CLI локально, для работы с этой статьей требуется последняя версия Azure CLI (2.0.70 или более поздней версии). Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Если вы не используете Cloud Shell, сначала выполните вход с помощью `az login`.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Темами событий сетки являются ресурсы Azure, которые необходимо поместить в группу ресурсов Azure. Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). 

В следующем примере создается группа ресурсов с именем `<resource_group_name>` в расположении *westus*.  Замените `<resource_group_name>` уникальным именем для группы ресурсов.

```azurecli-interactive
az group create --name <resource_group_name> --location westus
```

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

Замените `<appconfig_name>` уникальным именем хранилища конфигураций и `<resource_group_name>` группой ресурсов, созданной ранее. Имя должно быть уникальным, поскольку оно используется в качестве имени DNS.

```azurecli-interactive
az appconfig create \
  --name <appconfig_name> \
  --location westus \
  --resource-group <resource_group_name> \
  --sku free
```

## <a name="create-a-message-endpoint"></a>Создание конечной точки сообщения

Перед подпиской на раздел необходимо создать конечную точку для сообщения о событии. Обычно конечная точка выполняет действия на основе данных событий. Чтобы упростить работу с этим руководством, разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

Замените `<your-site-name>` уникальным именем для вашего веб-приложения. Имя веб-приложения должно быть уникальным, так как оно включается в запись DNS.

```azurecli-interactive
$sitename=<your-site-name>

az deployment group create \
  --resource-group <resource_group_name> \
  --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
  --parameters siteName=$sitename hostingPlanName=viewerhost
```

Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

Вы увидите сайт, на котором сейчас не отображаются никакие сообщения.

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-your-app-configuration-store"></a>Подпишитесь на хранилище конфигураций приложений

Подписка на раздел предоставляет Сетке событий Azure информацию о том, какие события вы намерены отслеживать и куда их следует отправлять. В следующем примере создается подписка на созданную Конфигурацию приложений и передается URL-адрес веб-приложения в качестве конечной точки для уведомления о событиях. Замените `<event_subscription_name>` именем подписки на событие. Для `<resource_group_name>` и `<appconfig_name>` используйте созданные ранее значения.

Конечная точка веб-приложения должна содержать суффикс `/api/updates/`.

```azurecli-interactive
appconfigId=$(az appconfig show --name <appconfig_name> --resource-group <resource_group_name> --query id --output tsv)
endpoint=https://$sitename.azurewebsites.net/api/updates

az eventgrid event-subscription create \
  --resource-id $appconfigId \
  --name <event_subscription_name> \
  --endpoint $endpoint
```

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Щелкните значок с изображением глаза, чтобы развернуть данные события. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Веб-приложение содержит код для проверки подписки.

![Просмотр события подписки](./media/quickstarts/event-grid/view-subscription-event.png)

## <a name="trigger-an-app-configuration-event"></a>Активация события Конфигурации приложений

Теперь необходимо активировать событие, чтобы увидеть, как Сетка событий Azure распределяет сообщение к вашей конечной точке. Создайте значение ключа с помощью предыдущего `<appconfig_name>`.

```azurecli-interactive
az appconfig kv set --name <appconfig_name> --key Foo --value Bar --yes
```

Вы активировали событие, а служба "Сетка событий" отправила сообщение в конечную точку, настроенную вами при оформлении подписки. Откройте веб-приложение и просмотрите в нем отправленные события.

```json
[{
  "id": "deb8e00d-8c64-4b6e-9cab-282259c7674f",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/eventDemoGroup/providers/microsoft.appconfiguration/configurationstores/{appconfig-name}",
  "subject": "https://{appconfig-name}.azconfig.io/kv/Foo",
  "data": {
    "key": "Foo",
    "etag": "a1LIDdNEIV6wCnfv3xaip7fMXD3"
  },
  "eventType": "Microsoft.AppConfiguration.KeyValueModified",
  "eventTime": "2019-05-31T18:59:54Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы планируете продолжать работу с этой Конфигурацией приложений и подпиской на события, не очищайте ресурсы, созданные при работе с этой статьей. Если вы не планируете продолжать работу, удалите все созданные ресурсы в этой статье.

Замените `<resource_group_name>` именем группы ресурсов, созданной ранее.

```azurecli-interactive
az group delete --name <resource_group_name>
```

## <a name="next-steps"></a>Следующие шаги

Теперь, когда вы знаете, как создавать разделы и подписки на события, ознакомьтесь с дополнительными сведениями о событиях "ключ — значение" и возможностях службы "Сетка событий".

- [Реагирование на события конфигурации приложения Azure](concept-app-configuration-event.md)
- [An introduction to Azure Event Grid](../event-grid/overview.md) (Общие сведения о службе "Сетка событий Azure")
- [Обработчики Сетки событий Azure](../event-grid/event-handlers.md)
