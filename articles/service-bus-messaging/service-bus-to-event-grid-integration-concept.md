---
title: Общие сведения об интеграции служебной шины Azure со службой "Сетка событий" | Документация Майкрософт
description: В этой статье содержится описание интеграции обмена сообщениями служебной шины Azure со службой "Сетка событий Azure".
documentationcenter: .net
author: spelluru
ms.topic: conceptual
ms.date: 02/11/2021
ms.author: spelluru
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 658107bb74396891c8e6e05a9e8074a9416a5f6f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100369668"
---
# <a name="azure-service-bus-to-event-grid-integration-overview"></a>Общие сведения об интеграции служебной шины Azure со службой "Сетка событий"
Теперь, когда в подписке или в очереди есть сообщения, но получатели отсутствуют, служебная шина может выдавать события для службы "Сетка событий". Вы можете создавать подписки на службу "Сетка событий" для пространств имен служебной шины, ожидать передачи этих событий и реагировать на них, запустив получатель. Благодаря этой возможности служебную шину можно использовать в моделях реактивного программирования. Благодаря этой возможности для очередей или подписок служебной шины c небольшим количеством сообщений теперь не требуется получатель для непрерывного опроса сообщений. 

Чтобы включить эту функцию, вам понадобится следующее:

* Пространство имен служебной шины уровня "Премиум" минимум с одной очередью служебной шины или раздел служебной шины минимум с одной подпиской.
* Доступ к пространству имен служебной шины с правами участника. Перейдите к пространству имен служебной шины в портал Azure, а затем выберите **Управление доступом (IAM)** и перейдите на вкладку **назначения ролей** . Убедитесь в наличии у участника доступа к пространству имен. 
* Подписка на службу "Сетка событий" для пространства имен служебной шины. Эта подписка необходима для получения из службы "Сетка событий" уведомления о том, что существуют сообщения, ожидающие действий. Типичными подписчиками могут быть компоненты Logic Apps службы приложений Azure или решения "Функции Azure" либо веб-перехватчики, связанные с веб-приложением. Следовательно, подписчики обрабатывают сообщения. 

![19][]

[!INCLUDE [event-grid-service-bus.md](../../includes/event-grid-service-bus.md)]

## <a name="event-grid-subscriptions-for-service-bus-namespaces"></a>Подписки сетки событий для пространств имен служебной шины
Вы можете создать подписки на службу "Сетка событий" для пространств имен служебной шины тремя способами:

- портал Azure. Ознакомьтесь со следующими учебниками, чтобы узнать, как использовать портал Azure для создания подписок на сетку событий для событий служебной шины с Azure Logic Apps и функциями Azure в качестве обработчиков. 
    - [Azure Logic Apps](service-bus-to-event-grid-integration-example.md#receive-messages-by-using-logic-apps)
    - [Функции Azure](service-bus-to-event-grid-integration-function.md#connect-the-function-and-namespace-via-event-grid)
* Azure CLI. В следующем примере интерфейса командной строки показано, как создать подписку на функции Azure для [системного раздела](../event-grid/system-topics.md) , созданного пространством имен служебной шины.

     ```azurecli-interactive
    namespaceid=$(az resource show --namespace Microsoft.ServiceBus --resource-type namespaces --name "<service bus namespace>" --resource-group "<resource group that contains the service bus namespace>" --query id --output tsv
    
    az eventgrid event-subscription create --resource-id $namespaceid --name "<YOUR EVENT GRID SUBSCRIPTION NAME>" --endpoint "<your_endpoint_url>" --subject-ends-with "<YOUR SERVICE BUS SUBSCRIPTION NAME>"
    ```
- PowerShell. Приведем пример:
    ```powershell-interactive
    $namespaceID = (Get-AzServiceBusNamespace -ResourceGroupName "<YOUR RESOURCE GROUP NAME>" -NamespaceName "<YOUR NAMESPACE NAME>").Id
    
    New-AzEVentGridSubscription -EventSubscriptionName "<YOUR EVENT GRID SUBSCRIPTION NAME>" -ResourceId $namespaceID -Endpoint "<YOUR ENDPOINT URL>” -SubjectEndsWith "<YOUR SERVICE BUS SUBSCRIPTION NAME>"
    ```
### <a name="how-many-events-are-emitted-and-how-often"></a>Как часто и в каком количестве выдаются события?

При наличии нескольких очередей и разделов или подписок в пространстве имен можно получить по крайней мере одно событие в очереди и одно в подписке. События выдаются незамедлительно, если в сущности служебной шины нет сообщений и приходит новое сообщение. Или они выдаются каждые две минуты, если служебная шина не обнаружит активного получателя. События не прерываются при просмотре сообщений.

По умолчанию служебная шина выдает события для всех сущностей в пространстве имен. Если необходимо получать события только для определенных сущностей, ознакомьтесь со следующим разделом.

### <a name="use-filters-to-limit-where-you-get-events-from"></a>Использование фильтров для ограничения источников получения событий

Если нужно получать события, например только из одной очереди или одной подписки в пространстве имен, используйте в службе "Сетка событий" фильтры *Начинается с* или *Оканчивается на*. В некоторых интерфейсах фильтры называются *префиксами* и фильтрами *суффиксов* . Если требуется получить события для нескольких, но не всех очередей и подписок, создайте несколько подписок на службу "Сетка событий" и укажите для каждой из них фильтр.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими руководствами: 
- [Azure Logic Apps для управления сообщениями служебной шины, полученными через службу "Сетка событий"](service-bus-to-event-grid-integration-example.md#receive-messages-by-using-logic-apps)
- [Функции Azure для работы с сообщениями служебной шины, полученными через службу "Сетка событий"](service-bus-to-event-grid-integration-function.md#connect-the-function-and-namespace-via-event-grid)

[1]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgrid1.png
[стр]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgriddiagram.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[21]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png
