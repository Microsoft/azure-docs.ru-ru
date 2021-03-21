---
title: Журналы диагностики центров уведомлений Azure | Документация Майкрософт
description: В этой статье представлен обзор всех журналов операционных и диагностических операций, доступных для центров уведомлений Azure.
author: brannon
ms.author: brjones
ms.service: notification-hubs
ms.topic: article
ms.date: 01/29/2021
ms.openlocfilehash: b98a04a70062461cec603bea83052c4f1224819e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101736242"
---
# <a name="enable-diagnostics-logs-for-notification-hubs"></a>Включение журналов диагностики для концентраторов уведомлений

Когда вы начинаете использовать пространство имен центров уведомлений Azure, может потребоваться отслеживать, как и когда пространство имен создается, удаляется или к которому осуществляется доступ. В этой статье приводятся общие сведения обо всех доступных журналах операционной и диагностики.

Центры уведомлений Azure в настоящее время поддерживают журналы операций и операций, которые фиксируют *операции управления* , выполняемые в пространстве имен концентраторов уведомлений Azure.

## <a name="diagnostic-logs-schema"></a>Схема журналов диагностики

Все журналы хранятся в формате нотация объектов JavaScript (JSON) в следующих двух расположениях:

- **AzureActivity**. отображает журналы операций и действий, выполняемых для пространства имен в портал Azure или с помощью развертываний шаблонов Azure Resource Manager.
- **AzureDiagnostics**. отображает журналы операций и действий, выполняемых в пространстве имен, с помощью API или клиентов управления на языке SDK.

Строки JSON журнала диагностики включают элементы, перечисленные в следующей таблице.

| Имя | Описание |
| ------- | ------- |
| time | Метка времени в формате UTC для журнала |
| resourceId | Относительный путь к ресурсу Azure |
| operationName | Имя операции управления |
| категория | Категория журнала. Допустимые значения: `OperationalLogs` |
| каллеридентити | Удостоверение вызывающего объекта, инициировавшего операцию управления |
| resultType | Состояние операции управления. Допустимые значения: `Succeeded` или `Failed` |
| resultDescription | Описание операции управления |
| correlationId | Идентификатор корреляции операции управления (если указано) |
| callerIpAddress | IP-адрес вызывающего. Пусто для вызовов, порожденных от портал Azure |

Ниже приведен пример строки JSON операционного журнала.

```json
{
    "operationName": "Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/action",
    "resourceId": "/SUBSCRIPTIONS/2CAC2A14-BA6B-46A6-BCE8-2D9781A41BA2/RESOURCEGROUPS/SAMPLES/PROVIDERS/MICROSOFT.NOTIFICATIONHUBS/NAMESPACES/SAMPLE-NS",
    "time": "1/1/2021 5:16:32 AM +00:00",
    "category": "OperationalLogs",
    "resultType": "Succeeded",
    "resultDescription": "Gets Hub Authorization Rules",
    "correlationId": "00000000-0000-0000-0000-000000000000",
    "callerIdentity": "{ \"identityType\": \"Portal\", \"identity\": \"\" }"
}
```

`callerIdentity`Поле может быть пустым или строкой JSON в одном из следующих форматов.

Для вызовов, исходящих из портал Azure `identity` поле пусто. Журнал можно сопоставить с журналами действий, чтобы определить пользователя, выполнившего вход.
```json
{
    "identityType": "Portal",
    "identity": ""
}
```

Для вызовов, выполненных с помощью Azure Resource Manager `identity` поле будет содержать имя пользователя, выполнившего вход в систему.
```json
{
   "identityType": "Username",
   "identity": "test@foo.com"
}
```

Для вызовов к центрам уведомлений REST API `identity` поле будет содержать имя политики доступа, используемой для создания токена SharedAccessSignature.
```json
{
   "identityType": "KeyName",
   "identity": "SharedAccessRootKey2"
}
```

## <a name="events-and-operations-captured-in-operational-logs"></a>События и операции, захваченные в операционных журналах

Операционные журналы записывают все операции управления, которые выполняются в пространстве имен центров уведомлений Azure. Операции с данными не фиксируются из-за большого объема операций с данными, проводимых в центрах уведомлений Azure.

Следующие операции управления фиксируются в операционных журналах: 

| Область | Имя операции | Описание операции |
| :-- | :-- | :-- |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/authorizationRules/action | Список правил авторизации |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/authorizationRules/delete | Удаление правила авторизации |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/authorizationRules/listkeys/action | Отображает ключи. |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/authorizationRules/read | Получение правила авторизации |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/authorizationRules/regenerateKeys/action | Повторное создание ключей |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/authorizationRules/write | Создание или обновление правила авторизации |
| Пространство имен | Microsoft. NotificationHubs/пространства имен/удаление | Удалить пространство имен |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/read | Получить пространство имен |
| Пространство имен | Microsoft.NotificationHubs/Namespaces/write | Создание или обновление пространства имен |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/action | Список правил авторизации |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/delete | Удаление правила авторизации |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/listkeys/action | Отображает ключи. |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/read | Чтение правила авторизации |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/regenerateKeys/action | Повторное создание ключей |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules/write | Создание или обновление правила авторизации |
| Центр уведомлений | Microsoft. NotificationHubs/пространства имен/NotificationHubs/Delete | Удаление концентратора уведомлений |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/pnsCredentials/action | Создание, обновление или получение учетных данных PNS |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/read | Получение центра уведомлений |
| Центр уведомлений | Microsoft.NotificationHubs/Namespaces/NotificationHubs/write | Создание или обновление центра уведомлений |

## <a name="enable-operational-logs"></a>Включить Операционные журналы

Операционные журналы по умолчанию отключены. Чтобы включить журналы, выполните следующие действия.

1. В [портал Azure](https://portal.azure.com)перейдите к пространству имен концентраторов уведомлений Azure, а затем в разделе **мониторинг** выберите  **параметры диагностики**.

   ![Ссылка "параметры диагностики"](./media/notification-hubs-diagnostic-logs/image-1.png)

1. В области **параметры диагностики** выберите **Добавить параметр диагностики**.  

   ![Ссылка "добавить параметр диагностики"](./media/notification-hubs-diagnostic-logs/image-2.png)

1. Настройте параметры диагностики, выполнив следующие действия.

   а. В поле **имя** введите имя для параметров диагностики.  

   b. Выберите одно из следующих трех назначений для журналов диагностики:  
   - Если выбран параметр **отправить в log Analytics рабочую область**, необходимо указать, какой экземпляр log Analytics будет отправляться диагностика.  
   - При выборе **архива в учетной записи хранения** необходимо настроить учетную запись хранения, в которой будут храниться журналы диагностики.  
   - Если выбрать **поток в концентраторе событий**, необходимо настроить концентратор событий, в который нужно передать журналы диагностики.

   c. Установите флажок **OperationalLogs** .

    ![Панель "параметры диагностики"](./media/notification-hubs-diagnostic-logs/image-3.png)

1. Щелкните **Сохранить**.

Новые параметры вступят в силу примерно через 10 минут. Журналы отображаются в настроенном целевом объекте архивации в области **журналы диагностики** .

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настройке параметров диагностики см. в следующих статьях:
* [Обзор журналов системы диагностики Azure](../azure-monitor/essentials/platform-logs-overview.md).

Дополнительные сведения о центрах уведомлений Azure см. в следующих статьях:
* [Что такое Центры уведомлений Azure?](notification-hubs-push-notification-overview.md)