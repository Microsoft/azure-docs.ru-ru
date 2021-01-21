---
title: Общие сведения об интеграции служебной шины Azure со службой "Сетка событий" | Документация Майкрософт
description: В этой статье содержится описание интеграции обмена сообщениями служебной шины Azure со службой "Сетка событий Azure".
documentationcenter: .net
author: spelluru
ms.topic: conceptual
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 71ee21c971b71c4000a123d1561e7e93d21203e1
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/21/2021
ms.locfileid: "98629153"
---
# <a name="azure-service-bus-to-event-grid-integration-overview"></a>Общие сведения об интеграции служебной шины Azure со службой "Сетка событий"

В служебной шине Azure появилась возможность интеграции со службой "Сетка событий Azure". Благодаря этой возможности для очередей или подписок служебной шины c небольшим количеством сообщений теперь не требуется получатель для непрерывного опроса сообщений. 

Теперь, когда в подписке или в очереди есть сообщения, но получатели отсутствуют, служебная шина может выдавать события для службы "Сетка событий". Вы можете создавать подписки на службу "Сетка событий" для пространств имен служебной шины, ожидать передачи этих событий и реагировать на них, запустив получатель. Благодаря этой возможности служебную шину можно использовать в моделях реактивного программирования.

Чтобы включить эту функцию, вам понадобится следующее:

* Пространство имен служебной шины уровня "Премиум" минимум с одной очередью служебной шины или раздел служебной шины минимум с одной подпиской.
* Доступ к пространству имен служебной шины с правами участника.
* Подписка на службу "Сетка событий" для пространства имен служебной шины. Эта подписка необходима для получения из службы "Сетка событий" уведомления о том, что существуют сообщения, ожидающие действий. Типичными подписчиками могут быть компоненты Logic Apps службы приложений Azure или решения "Функции Azure" либо веб-перехватчики, связанные с веб-приложением. Следовательно, подписчики обрабатывают сообщения. 

![19][]


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="verify-that-you-have-contributor-access"></a>Проверка наличия доступа с правами участника
Перейдите к пространству имен служебной шины, выберите **Управление доступом (IAM)** и перейдите на вкладку **назначения ролей** . Убедитесь в наличии у участника доступа к пространству имен. 

### <a name="events-and-event-schemas"></a>События и схемы событий

Сейчас служебная шина отправляет события в двух сценариях:

* [ActiveMessagesWithNoListenersAvailable](#active-messages-available-event)
* [DeadletterMessagesAvailable](#deadletter-messages-available-event)
* [активемессажесаваилаблепериодикнотификатионс](#active-messages-available-periodic-notifications)
* [деадлеттермессажесаваилаблепериодикнотификатионс](#deadletter-messages-available-periodic-notifications)

Кроме того, служебная шина использует стандартные механизмы безопасности и [проверки подлинности](../event-grid/security-authentication.md) службы "Сетка событий".

Дополнительные сведения см. в статье [Схема событий службы "Сетка событий Azure"](../event-grid/event-schema.md).

#### <a name="active-messages-available-event"></a>Событие наличия активных сообщений

Это событие создается, если в очереди или подписке присутствуют активные сообщения, а получатели отсутствуют.

Схема для этого события выглядит следующим образом:

```JSON
{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}
```

#### <a name="deadletter-messages-available-event"></a>Событие доступности сообщений о недоставленных сообщениях

На одну очередь недоставленных сообщений приходится минимум одно событие с сообщениями, для которых отсутствуют активные получатели.

Схема для этого события выглядит следующим образом:

```JSON
[{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

#### <a name="active-messages-available-periodic-notifications"></a>Активные сообщения доступные периодические уведомления

Это событие создается периодически при наличии активных сообщений в конкретной очереди или подписке, даже если для этой конкретной очереди или подписки имеются активные прослушиватели.

Схема для события выглядит следующим образом.

```json
[{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailablePeriodicNotifications",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

#### <a name="deadletter-messages-available-periodic-notifications"></a>Доступные периодические уведомления о недоставленных сообщениях

Это событие создается периодически при наличии сообщений о недоставленных сообщениях в конкретной очереди или подписке, даже при наличии активных прослушивателей в недоставленной сущности этой конкретной очереди или подписки.

Схема для события выглядит следующим образом.

```json
[{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailablePeriodicNotifications",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="how-many-events-are-emitted-and-how-often"></a>Как часто и в каком количестве выдаются события?

При наличии нескольких очередей и разделов или подписок в пространстве имен можно получить по крайней мере одно событие в очереди и одно в подписке. События выдаются незамедлительно, если в сущности служебной шины нет сообщений и приходит новое сообщение. Или они выдаются каждые две минуты, если служебная шина не обнаружит активного получателя. События не прерываются при просмотре сообщений.

По умолчанию служебная шина выдает события для всех сущностей в пространстве имен. Если необходимо получать события только для определенных сущностей, ознакомьтесь со следующим разделом.

### <a name="use-filters-to-limit-where-you-get-events-from"></a>Использование фильтров для ограничения источников получения событий

Если нужно получать события, например только из одной очереди или одной подписки в пространстве имен, используйте в службе "Сетка событий" фильтры *Начинается с* или *Оканчивается на*. В некоторых интерфейсах фильтры называются *префиксами* и фильтрами *суффиксов* . Если требуется получить события для нескольких, но не всех очередей и подписок, создайте несколько подписок на службу "Сетка событий" и укажите для каждой из них фильтр.

## <a name="create-event-grid-subscriptions-for-service-bus-namespaces"></a>Создание подписок на службу "Сетка событий" для пространств имен служебной шины

Вы можете создать подписки на службу "Сетка событий" для пространств имен служебной шины тремя способами:

* На портале Azure
* В [Azure CLI](#azure-cli-instructions)
* В [PowerShell](#powershell-instructions)

## <a name="azure-portal-instructions"></a>Инструкции по созданию подписки на портале Azure

Чтобы создать подписку на службу "Сетка событий", выполните следующие действия:
1. На портале Azure перейдите к своему пространству имен.
2. В области слева щелкните **Сетка событий**. 
3. Выберите **Подписка на события**.  

   На следующем изображении показано пространство имен, которое содержит подписку на службу "Сетка событий":

   ![Подписки на службу "Сетка событий"](./media/service-bus-to-event-grid-integration-concept/sbtoeventgridportal.png)

   На следующем изображении показано, как подписаться на функцию или веб-перехватчик без применения конкретной фильтрации:

   ![21][]

## <a name="azure-cli-instructions"></a>Инструкции по работе в Azure CLI

Для начала установите Azure CLI 2.0 или более поздней версии. [Скачайте установщик](/cli/azure/install-azure-cli). Выберите **Windows + X**, а затем откройте новую консоль PowerShell с разрешениями администратора. В качестве альтернативы можно использовать командную оболочку на портале Azure.

Выполните следующий код.

 ```azurecli-interactive
az login

az account set -s "<Azure subscription name>"

namespaceid=$(az resource show --namespace Microsoft.ServiceBus --resource-type namespaces --name "<service bus namespace>" --resource-group "<resource group that contains the service bus namespace>" --query id --output tsv

az eventgrid event-subscription create --resource-id $namespaceid --name "<YOUR EVENT GRID SUBSCRIPTION NAME (CAN BE ANY NOT EXISTING)>" --endpoint "<your_function_url>" --subject-ends-with "<YOUR SERVICE BUS SUBSCRIPTION NAME>"
```

Если вы используете BASH 

## <a name="powershell-instructions"></a>Инструкции по работе в среде PowerShell

Убедитесь в наличии Azure PowerShell. [Скачайте установщик](/powershell/azure/install-Az-ps). Нажмите клавиши **Windows+X** и откройте новую консоль PowerShell с разрешениями администратора. В качестве альтернативы можно использовать командную оболочку на портале Azure.

```powershell-interactive
Connect-AzAccount

Select-AzSubscription -SubscriptionName "<YOUR SUBSCRIPTION NAME>"

# This might be installed already
Install-Module Az.ServiceBus

$NSID = (Get-AzServiceBusNamespace -ResourceGroupName "<YOUR RESOURCE GROUP NAME>" -Na
mespaceName "<YOUR NAMESPACE NAME>").Id

New-AzEVentGridSubscription -EventSubscriptionName "<YOUR EVENT GRID SUBSCRIPTION NAME (CAN BE ANY NOT EXISTING)>" -ResourceId $NSID -Endpoint "<YOUR FUNCTION URL>” -SubjectEndsWith "<YOUR SERVICE BUS SUBSCRIPTION NAME>"
```

Здесь вы можете просмотреть остальные параметры установки или выполнить тестирование поступления событий.

## <a name="next-steps"></a>Дальнейшие действия

* [Примеры](service-bus-to-event-grid-integration-example.md) использования служебной шины со службой "Сетка событий".
* Дополнительные сведения о [службе "Сетка событий Azure"](../event-grid/index.yml).
* Дополнительные сведения о решении "Функции Azure" см. [здесь](../azure-functions/index.yml).
* Дополнительные сведения о [Logic Apps](../logic-apps/index.yml).
* Дополнительные сведения о [служебной шине](/azure/service-bus/).

[1]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgrid1.png
[стр]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgriddiagram.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[открыт]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png
