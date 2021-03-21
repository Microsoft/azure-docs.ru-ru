---
title: включить файл
description: включить файл
services: service-bus-messaging, event-hubs
author: spelluru
ms.service: service-bus-messaging, event-hubs
ms.topic: include
ms.date: 12/12/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 9bc641d680d927c44814f6814ebf6a6dde958c9e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98901163"
---
Функции Azure позволяют создавать задачи репликации, предназначенные только для конфигурации и направленные на предварительно созданную точку входа. [Примеры репликации на основе конфигурации для службы "функции Azure](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config) " иллюстрируют использование [готовых вспомогательных](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/src/Azure.Messaging.Replication) функций в коде или старайтесь не обрабатывать код и просто использовать конфигурацию.

## <a name="create-a-replication-task"></a>Создание задачи репликации
Чтобы создать такую задачу репликации, сначала создайте новую папку под папкой проекта. Имя новой папки — это имя функции, например, `europeToAsia` или `telemetry` . Имя не имеет прямой корреляции с используемыми сущностями обмена сообщениями и служит только для их обнаружения. В одном проекте можно создавать десятки функций.

Затем создайте `function.json` файл в папке. Файл настраивает функцию. Начните со следующего содержимого:

``` JSON
{
    "configurationSource": "config",
    "bindings" : [
        {
            "direction": "in",
            "name": "input" 
        },
        {
            "direction": "out",
            "name": "output"
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": -1,
        "minimumInterval": "00:00:05",
        "maximumInterval": "00:05:00"
    },
    "disabled": false,
    "scriptFile": "../bin/Azure.Messaging.Replication.dll",
    "entryPoint": "Azure.Messaging.Replication.*"
}
```

В этом файле необходимо выполнить три шага настройки, которые зависят от того, какие сущности необходимо подключить.

1. [Настройка направления ввода](#configure-the-input-direction)
2. [Настройка направления вывода](#configure-the-output-direction)
3. [Настройка точки входа](#configure-the-entry-point)

### <a name="configure-the-input-direction"></a>Настройка направления ввода

#### <a name="event-hub-input"></a>Входные данные концентратора событий

Если вы хотите получить события из концентратора событий, добавьте сведения о конфигурации в верхний раздел в разделе "привязки", который задает

* **Type** — тип "eventHubTrigger".
* **Подключение** — имя значения параметров приложения для строки подключения к концентратору событий. 
* **eventHubName** — имя концентратора событий в пространстве имен, определяемом строкой подключения.
* **группы потребителей** — имя группы потребителей. Обратите внимание, что имя заключено в знаки процента и, следовательно, также относится к значению параметров приложения.

```JSON
    ...
    "bindings" : [
        {
            "direction": "in",
            "type": "eventHubTrigger",
            "connection": "functionname-source-connection",
            "eventHubName": "eventHubA",
            "consumerGroup" : "%functionname-source-consumergroup%",
            "name": "input" 
        }
    ...
```

#### <a name="service-bus-queue-input"></a>Входные данные очереди служебной шины

Если вы хотите получить события из очереди служебной шины, добавьте сведения о конфигурации в верхний раздел в разделе "привязки", который задает

* **Type** — тип "serviceBusTrigger".
* **Подключение** — имя значения параметров приложения для строки подключения служебной шины.
* **QueueName** — имя очереди служебной шины в пространстве имен, определяемом строкой подключения.

```JSON
    ...
    "bindings" : [
        {
            "direction": "in",
            "type": "serviceBusTrigger",
            "connection": "functionname-source-connection",
            "queueName": "queue-a",
            "name": "input" 
        }
    ...
```

#### <a name="service-bus-topic-input"></a>Входные данные раздела служебной шины

Если вы хотите получить события из раздела служебной шины, добавьте сведения о конфигурации в верхний раздел в разделе "привязки", который задает

* **Type** — тип "serviceBusTrigger".
* **Подключение** — имя значения параметров приложения для строки подключения служебной шины. Это значение должно быть, `{FunctionName}-source-connection` Если вы хотите использовать предоставленные скрипты.
* **топикнаме** — имя раздела служебной шины в пространстве имен, определяемом строкой подключения.
* **subscriptionName** — имя подписки служебной шины для данного раздела в пространстве имен, определяемом строкой подключения.

```JSON
    ...
    "bindings" : [
        {
            "direction": "in",
            "type": "serviceBusTrigger",
            "connection": "functionname-source-connection",
            "topicName": "topic-x",
            "subscriptionName" : "sub-y",
            "name": "input" 
        }
    ...
```

### <a name="configure-the-output-direction"></a>Настройка направления вывода

#### <a name="event-hub-output"></a>Выходные данные концентратора событий

Если вы хотите перенаправить события в концентратор событий, добавьте сведения о конфигурации в нижний раздел в разделе "привязки", который задает

* **Type** — тип "eventHub".
* **Подключение** — имя значения параметров приложения для строки подключения к концентратору событий. 
* **eventHubName** — имя концентратора событий в пространстве имен, определяемом строкой подключения.

```JSON
    ...
    "bindings" : [
        {
            ...
        },
        {
            "direction": "out",
            "type": "eventHub",
            "connection": "functionname-target-connection",
            "eventHubName": "eventHubB",
            "name": "output" 
        }
    ...
```

#### <a name="service-bus-queue-output"></a>Выходные данные очереди служебной шины

Если вы хотите перенаправить события в очередь служебной шины, добавьте сведения о конфигурации в нижний раздел в разделе "привязки", который задает

* **Type** — тип "serviceBus".
* **Подключение** — имя значения параметров приложения для строки подключения служебной шины. 
* **QueueName** — имя очереди служебной шины в пространстве имен, определяемом строкой подключения.

```JSON
    ...
    "bindings" : [
        {
            ...
        },
        {
            "direction": "out",
            "type": "serviceBus",
            "connection": "functionname-target-connection",
            "queueName": "queue-b",
            "name": "output" 
        }
    ...
```

#### <a name="service-bus-topic-output"></a>Выходные данные раздела служебной шины

Если вы хотите перенаправить события в раздел служебной шины, добавьте сведения о конфигурации в нижний раздел в разделе "привязки", который задает

* **Type** — тип "serviceBus".
* **Подключение** — имя значения параметров приложения для строки подключения служебной шины. 
* **топикнаме** — имя раздела служебной шины в пространстве имен, определяемом строкой подключения.

```JSON
    ...
    "bindings" : [
        {
            ...
        },
        {
            "direction": "out",
            "type": "serviceBus",
            "connection": "functionname-target-connection",
            "topicName": "topic-b",
            "name": "output" 
        }
    ...
```

### <a name="configure-the-entry-point"></a>Настройка точки входа

Конфигурация точки входа выбирает одну из стандартных задач репликации. При изменении `Azure.Messaging.Replication` проекта можно также добавить задачи и ссылаться на них здесь. Например:

```JSON
    ...
    "scriptFile": "../dotnet/bin/Azure.Messaging.Replication.dll",
    "entryPoint": "Azure.Messaging.Replication.EventHubReplicationTasks.ForwardToEventHub"
    ...
```

В следующей таблице приведены правильные значения для сочетаний источников и целей.

| Источник      | Назначение      | Точка входа 
|-------------|-------------|------------------------------------------------------------------------
| Концентратор событий   | Концентратор событий   | `Azure.Messaging.Replication.EventHubReplicationTasks.ForwardToEventHub`
| Концентратор событий   | Cлужебная шина | `Azure.Messaging.Replication.EventHubReplicationTasks.ForwardToServiceBus`
| Cлужебная шина | Концентратор событий   | `Azure.Messaging.Replication.ServiceBusReplicationTasks.ForwardToEventHub`
| Cлужебная шина | Cлужебная шина | `Azure.Messaging.Replication.ServiceBusReplicationTasks.ForwardToServiceBus`

### <a name="retry-policy"></a>Политика повтора

Дополнительные сведения о настройке политики повторных попыток см. в [документации по функциям Azure](../articles/azure-functions/functions-bindings-error-pages.md) . Параметры политики, выбранные во всех проектах в этом репозитории, настраивают стратегию экспоненциальной задержки с интервалами повтора от 5 секунд до 5 минут с бесконечными повторными попытками, чтобы избежать потери данных.

Для служебной шины ознакомьтесь с разделом ["использование поддержки повторов на основе устойчивости триггеров"](../articles/azure-functions/functions-bindings-error-pages.md#using-retry-support-on-top-of-trigger-resilience) , чтобы понять взаимодействие триггеров и максимальное число доставок, определенное для очереди.

### <a name="build-deploy-and-configure"></a>Сборка, развертывание и настройка

Хотя вы занимаетесь настройкой, задачи по-прежнему требуют создания развертываемого приложения и настройки узлов службы "функции Azure" таким, чтобы они получили все необходимые сведения для подключения к указанным конечным точкам. 

Это проиллюстрировано вместе с многократно используемыми скриптами в [примерах репликации на основе конфигурации для функций Azure](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config).