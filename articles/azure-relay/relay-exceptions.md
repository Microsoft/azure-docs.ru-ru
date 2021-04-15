---
title: Исключения ретранслятора Azure и способы их разрешения | Документация Майкрософт
description: Список исключений ретранслятора Azure и предлагаемые действия по их разрешению.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: e3f9fe2e8aac826bd63d9842bd3380be10e1b87e
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107313952"
---
# <a name="azure-relay-exceptions"></a>Исключения ретранслятора Azure

В этой статье перечислены некоторые исключения, создаваемые интерфейсами API ретранслятора Azure. Этот справочник может измениться, поэтому следите за обновлениями.

## <a name="exception-categories"></a>Категории исключений

Интерфейсы API ретранслятора создают исключения, которые можно разделить на указанные ниже категории. Кроме того, здесь указаны предлагаемые действия, которые можно предпринять для их разрешения.

*   **Ошибка в коде пользователя.**[System.ArgumentException](/dotnet/api/system.argumentexception), [System.InvalidOperationException](/dotnet/api/system.invalidoperationexception), [System.OperationCanceledException](/dotnet/api/system.operationcanceledexception), [System.Runtime.Serialization.SerializationException](/dotnet/api/system.runtime.serialization.serializationexception). 

    **Общее действие.** Прежде чем продолжить, попытайтесь исправить код.
*   **Ошибка настройки или конфигурации.**[System.UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception). 

    **Общее действие.** Проверьте конфигурацию и измените ее при необходимости.
*   **Временные исключения.** [Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception), [Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception), [Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception). 

    **Общее действие.** Повторите операцию или уведомите пользователей.
*   **Другие исключения.**[System.Transactions.TransactionException](/dotnet/api/system.transactions.transactionexception), [System.TimeoutException](/dotnet/api/system.timeoutexception). 

    **Общее действие.** Связано с типом исключения. Ознакомьтесь с таблицей в следующем разделе. 

## <a name="exception-types"></a>Типы исключений

В следующей таблице перечислены типы исключений обмена сообщениями и их причины, а также рекомендуемые действия, которые можно выполнять для их разрешения.

| **Тип исключения** | **Описание** | **Рекомендуемое действие** | **Примечание к автоматическому или немедленному повтору** |
| --- | --- | --- | --- |
| [Timeout](/dotnet/api/system.timeoutexception) |Сервер не ответил на запрошенную операцию в течение указанного времени, которое задается параметром [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout). Возможно, сервер выполнил запрошенную операцию. Это может произойти из-за сетевых или других задержек в инфраструктуре. |Проверьте состояние системы для обеспечения согласованности и при необходимости повторите попытку. Ознакомьтесь с разделом [TimeoutException](#timeoutexception). |Повторная попытка может помочь в некоторых случаях. Добавьте в код логику повторных попыток. |
| [Недопустимая операция](/dotnet/api/system.invalidoperationexception) |Запрошенная пользователем операция не допускается в пределах сервера или службы. Подробные сведения см. в сообщении об исключении. |Проверьте код и обратитесь к документации. Убедитесь, что запрошенная операция допустима. |Повторная попытка не поможет. |
| [Операция отменена](/dotnet/api/system.operationcanceledexception) |Предпринята попытка вызвать операцию с объектом, который уже закрыт, прерван или удален. Иногда это означает, что внешняя транзакция уже удалена. |Проверьте код и убедитесь, что он не вызывает операции с удаленным объектом. |Повторная попытка не поможет. |
| [Несанкционированный доступ](/dotnet/api/system.unauthorizedaccessexception) |Объекту [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) не удалось получить маркер. Маркер является недопустимым или не содержит утверждений, необходимых для выполнения операции. |Убедитесь, что поставщик токенов создан с использованием правильных значений. Проверьте конфигурацию службы контроля доступа. |Повторная попытка может помочь в некоторых случаях. Добавьте в код логику повторных попыток. |
| [Исключение аргумента](/dotnet/api/system.argumentexception)<br /> [Аргумент Null](/dotnet/api/system.argumentnullexception)<br />[Аргумент вне допустимого диапазона](/dotnet/api/system.argumentoutofrangeexception) |Далее приведены причины возникновения исключений:<br />Для этого метода предоставлен один или несколько недопустимых аргументов.<br /> Универсальный код ресурса (URI), переданный в [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) или [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory.create), содержит один или несколько сегментов пути.<br />В [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) или [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory.create) передана недопустимая схема универсального кода ресурса (URI). <br />Значение свойства превышает 32 КБ. |Проверьте вызывающий код и убедитесь, что аргументы заданы правильно. |Повторная попытка не поможет. |
| [Сервер занят](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) |В настоящее время служба не может обработать запрос. |Клиент может некоторое время подождать, после чего следует повторить операцию. |Клиент может повторить операцию через определенный промежуток времени. Если в результате повтора возникает другое исключение, проверьте поведение повтора этого исключения. |
| [Превышена квота](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) |Сущность обмена сообщениями достигла максимально допустимого размера. |Создайте в сущности пространство, получая сообщения из сущности или ее подочередей. Ознакомьтесь с разделом [Исключение QuotaExceededException](#quotaexceededexception). |Повторная попытка может помочь, если сообщения были удалены. |
| [Превышен размер сообщения](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |Полезные данные сообщения превышают предел в 256 КБ. Обратите внимание, что предел в 256 КБ является общим размером сообщения с учетом системных свойств и служебных данных Microsoft .NET. |Сократите размер полезных данных сообщения, а затем повторите операцию. |Повторная попытка не поможет. |

## <a name="quotaexceededexception"></a>QuotaExceededException

[QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) указывает на превышение квоты для конкретного объекта.

Для ретранслятора это исключение служит оболочкой для исключения [System.ServiceModel.QuotaExceededException](/dotnet/api/system.servicemodel.quotaexceededexception), указывающего, что для этой конечной точки превышено максимальное число прослушивателей. Это указывается в значении поля **MaximumListenersPerEndpoint** сообщения об исключении.

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](/dotnet/api/system.timeoutexception) указывает, что запущенная пользователем операция занимает больше времени, чем время ожидания операции. 

Проверьте значение свойства [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit#System_Net_ServicePointManager_DefaultConnectionLimit). Достижение этого ограничения также может породить исключение [TimeoutException](/dotnet/api/system.timeoutexception).

Для ретранслятора исключения времени ожидания могут отображаться при первом открытии подключения отправителя ретрансляции. Существует две основные причины этого исключения:

*   слишком маленькое значение [OpenTimeout](/previous-versions/) (вплоть до доли секунды);
* неотвечающие локальные прослушиватели ретрансляции (или проблемы, связанные с тем, что правила брандмауэра запрещают прослушивателям принимать новые клиентские подключения) и значение [OpenTimeout](/previous-versions/) меньше 20 секунд.

Пример.

```
'System.TimeoutException’: The operation did not complete within the allotted timeout of 00:00:10.
The time allotted to this operation may have been a portion of a longer timeout.
```

### <a name="common-causes"></a>Основные причины
Существует две основные причины этой ошибки.

*   **Неправильная конфигурация**
    
    Время ожидания операции может быть слишком коротким для состояния операции. Время ожидания операции в пакете SDK клиента по умолчанию составляет 60 секунд. Проверьте, не задано ли в вашем коде слишком маленькое значение. Обратите внимание, что загрузка ЦП и состояние сети могут повлиять на время, необходимое для выполнения операции. Мы рекомендуем не задавать слишком маленькое значение времени ожидания операции.
*   **Временная ошибка службы**

    Иногда в службе ретранслятора могут происходить задержки при обработке запросов. Например, это может произойти в период интенсивной нагрузки сети. В этом случае можно настроить повторную попытку выполнения операции через некоторое время до ее успешного завершения. Если же операцию по-прежнему не удается выполнить после нескольких попыток, посетите [сайт состояния служб Azure](https://azure.microsoft.com/status/), чтобы получить сведения об известных простоях служб.

## <a name="next-steps"></a>Дальнейшие действия
* [Часто задаваемые вопросы о ретрансляторе Azure](relay-faq.yml)
* [Создание пространства имен ретранслятора с помощью портала Azure](relay-create-namespace-portal.md)
* [Приступая к работе с гибридными подключениями к ретранслятору](relay-hybrid-connections-dotnet-get-started.md)
* [Приступая к работе с гибридными подключениями к ретранслятору](relay-hybrid-connections-node-get-started.md)
