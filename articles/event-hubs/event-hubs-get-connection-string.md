---
title: Получение строки подключения к службе "Центры событий Azure" | Документация Майкрософт
description: В этой статье приводятся инструкции по получению строки подключения, с помощью которой клиенты могут установить соединение со службой "Центры событий Azure".
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 5ae6c66ddbbf4b9946e7037e1a7723043bf60507
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86537196"
---
# <a name="get-an-event-hubs-connection-string"></a>Получение строки подключения Центров событий

Чтобы использовать Центры событий, следует создать пространство имен Центров событий. Пространство имен — это контейнер определенной области для нескольких концентраторов событий и тем Kafka. Это пространство имен предоставляет уникальное [полное доменное имя (FQDN)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name). После создания пространства имен можно получить строку подключения, необходимую для взаимодействия с Центрами событий.

Строка подключения для Центров событий Azure состоит из следующих компонентов, внедренных в него.

* FQDN = FQDN созданного пространства имен Центров событий (будет включать имя пространства имен Центров событий, за которым следует servicebus.windows.net)
* SharedAccessKeyName = имя, выбранное для ключей SAS вашего приложения.
* SharedAccessKey = сформированное значение ключа.

Строка подключения выглядит следующим образом.
```
Endpoint=sb://<FQDN>/;SharedAccessKeyName=<KeyName>;SharedAccessKey=<KeyValue>
```

Пример строки подключения может выглядеть `Endpoint=sb://dummynamespace.servicebus.windows.net/;SharedAccessKeyName=DummyAccessKeyName;SharedAccessKey=5dOntTRytoC24opYThisAsit3is2B+OGY1US/fuL3ly=`.

Получения строки подключения различными способами описано в этой статье.

## <a name="get-connection-string-from-the-portal"></a>Получение строки подключения на портале
1. Войдите на [портал Azure](https://portal.azure.com). 
2. В меню слева выберите **Все службы**. 
3. В разделе **Аналитика** выберите **Центры событий**. 
4. Выберите свой концентратор событий из списка концентраторов.
6. На странице **Пространство имен Центров событий** выберите **Политики общего доступа** в меню слева.

    ![Элемент меню "Политики общего доступа"](./media/event-hubs-get-connection-string/event-hubs-get-connection-string1.png)
7. В списке политик выберите **Политика общего доступа**. По умолчанию он имеет имя: **рутманажешаредакцессполици**. Вы можете добавить политику с соответствующими разрешениями (чтения, записи) и использовать ее. 

    ![Политики общего доступа Центров событий](./media/event-hubs-get-connection-string/event-hubs-get-connection-string2.png)
8. Щелкните кнопку **копирования** справа от текстового поля **Connection string-primary key** (Первичный ключ строки подключения). 

    ![Получение строки подключения для Центров событий](./media/event-hubs-get-connection-string/event-hubs-get-connection-string3.png)

## <a name="getting-the-connection-string-with-azure-powershell"></a>Получение строки подключения с помощью Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Вы можете использовать [Get-азевенсубкэй](/powershell/module/az.eventhub/get-azeventhubkey) , чтобы получить строку подключения для конкретной политики или имени правила, как показано ниже.

```azurepowershell-interactive
Get-AzEventHubKey -ResourceGroupName dummyresourcegroup -NamespaceName dummynamespace -AuthorizationRuleName RootManageSharedAccessKey
```

## <a name="getting-the-connection-string-with-azure-cli"></a>Получение строки подключения с помощью Azure CLI
Получение строки подключения см. ниже.

```azurecli-interactive
az eventhubs namespace authorization-rule keys list --resource-group dummyresourcegroup --namespace-name dummynamespace --name RootManageSharedAccessKey
```

Чтобы получить строку подключения для сущности EventHub, можно также использовать следующую команду:

```azurecli-interactive
az eventhubs eventhub authorization-rule keys list --resource-group dummyresourcegroup --namespace-name dummynamespace --eventhub-name dummyeventhub --name RootManageSharedAccessKey
```

Дополнительные сведения о командах интерфейса командной строки Azure для Центра событий см. в статье [az eventhubs](/cli/azure/eventhubs).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Центрах событий см. в следующих источниках:

* [Общие сведения о Центрах событий](./event-hubs-about.md)
* [Создание концентратора событий](event-hubs-create.md)
