---
title: Мониторинг событий служб мультимедиа Azure с помощью службы "Сетка событий"
description: В этой статье показано, как подписываться на службу "Сетка событий", чтобы отслеживать события служб мультимедиа Azure с помощью Azure CLI.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 03/17/2021
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 47d079aa5038a5ef09df30f0561c258bfbf6a9f7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609745"
---
# <a name="create-and-monitor-media-services-events-with-event-grid-using-the-azure-cli"></a>Создание и мониторинг событий Служб мультимедиа Azure с помощью Сетки событий и Azure CLI

[!INCLUDE [media services api v3 logo](../includes/v3-hr.md)]

"Сетка событий Azure" — это служба обработки событий для облака. Эта служба использует [подписки на события](../../../event-grid/concepts.md#event-subscriptions) для маршрутизации сообщений о событиях подписчикам. События Служб мультимедиа содержат все сведения, необходимые для реагирования на изменения в данных. Событие Служб мультимедиа можно определить, так как свойство eventType начинается с Microsoft.Media. Дополнительные сведения см. в статье [Схемы службы "Сетка событий Azure" для событий Служб мультимедиа](media-services-event-schemas.md).

В этой статье выполняется подписка на события для учетной записи Служб мультимедиа Azure с помощью Azure CLI. Затем можно активировать события, чтобы увидеть результат. Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. В этой статье события отправляются в веб-приложение, которое собирает и отображает сообщения.

## <a name="prerequisites"></a>Предварительные требования

- Активная подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начинать работу.
- Установите и используйте CLI на локальном компьютере. Для работы с этим руководством вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

    Сейчас в Azure Cloud Shell работают не все команды [интерфейса командной строки Служб мультимедиа версии 3](/cli/azure/ams). Рекомендуется использовать интерфейс командной строки локально.

- [Создание учетной записи Служб мультимедиа](../create-account-howto.md).

    Запишите значения, которые вы использовали в качестве имени группы ресурсов и имени учетной записи Служб мультимедиа.

## <a name="create-a-message-endpoint"></a>Создание конечной точки сообщения

Перед созданием подписки на события учетной записи Служб мультимедиа необходимо создать конечную точку для сообщения о событии. Обычно конечная точка выполняет действия на основе данных событий. В этой статье разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

1. Выберите **Развернуть в Azure**, чтобы развернуть решение в своей подписке. На портале Azure укажите значения остальных параметров.

   [![Изображение с кнопкой "Развернуть в Azure".](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json)

1. Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

При переходе на сайт "Средство просмотра Сетки событий Azure" вы увидите, что в нем пока нет событий.
   
[!INCLUDE [event-grid-register-provider-portal.md](../../../../includes/event-grid-register-provider-portal.md)]

## <a name="set-the-azure-subscription"></a>Настройка подписки Azure

В приведенной ниже команде укажите идентификатор подписки Azure, который необходимо использовать в учетной записи Служб мультимедиа. Чтобы просмотреть список доступных подписок, перейдите на страницу [Подписки](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

```azurecli
az account set --subscription mySubscriptionId
```

## <a name="subscribe-to-media-services-events"></a>Подписка на события Служб мультимедиа Azure

Подпишитесь на статью, чтобы сообщить сетке событий, какие события следует отслеживанию. Следующий пример подписывается на созданную учетную запись служб мультимедиа и передает URL-адрес с созданного веб-сайта в качестве конечной точки для уведомления о событии. 

Замените `<event_subscription_name>` уникальным именем подписки на события. Для `<resource_group_name>` и `<ams_account_name>` используйте значения, которые указали при создании учетной записи Служб мультимедиа. Для `<endpoint_URL>` укажите URL-адрес веб-приложения и добавьте `api/updates` к URL-адресу домашней страницы. Указав конечную точку при подписке, служба "Сетка событий Azure" обрабатывает маршрутизацию событий для этой конечной точки. 

1. Получение идентификатора ресурса

    ```azurecli
    amsResourceId=$(az ams account show --name <ams_account_name> --resource-group <resource_group_name> --query id --output tsv)
    ```

    Пример:

    ```
    amsResourceId=$(az ams account show --name amsaccount --resource-group amsResourceGroup --query id --output tsv)
    ```

2. Оформление подписки на события

    ```azurecli
    az eventgrid event-subscription create \
    --source-resource-id $amsResourceId \
    --name <event_subscription_name> \
    --endpoint <endpoint_URL>
    ```

    Пример:

    ```
    az eventgrid event-subscription create --source-resource-id $amsResourceId --name amsTestEventSubscription --endpoint https://amstesteventgrid.azurewebsites.net/api/updates/
    ```    

    > [!TIP]
    > Вы можете получить предупреждение о подтверждении приема. Подождите несколько минут и проверьте подтверждение приема.

Теперь необходимо активировать события, чтобы увидеть, как Сетка событий Azure распределяет сообщение к вашей конечной точке.

## <a name="send-an-event-to-your-endpoint"></a>Отправка события в конечную точку

Вы можете активировать события для учетной записи Служб мультимедиа, запустив задание кодирования. Чтобы закодировать файл и начать отправку событий, следуйте инструкциям [в этом кратком руководстве](../stream-files-dotnet-quickstart.md). 

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Конечная точка должна задать для `validationResponse` значение `validationCode`. Дополнительные сведения см. в разделе [Сетка событий: безопасность и проверка подлинности](../../../event-grid/security-authentication.md). Вы можете просмотреть код веб-приложения, чтобы увидеть, как он проверяет подписку.

> [!TIP]
> Щелкните значок с изображением глаза, чтобы развернуть данные события. Не обновляйте страницу, если вы хотите просмотреть все события.

![Просмотр события подписки](../media/monitor-events-portal/view-subscription-event.png)

## <a name="next-steps"></a>Дальнейшие действия

[Отправка, кодирование и потоковая передача](../stream-files-tutorial-with-api.md)
