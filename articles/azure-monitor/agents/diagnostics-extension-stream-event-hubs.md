---
title: Отправка данных из расширения системы диагностики Microsoft Azure в концентраторы событий Azure
description: Настройте расширение системы диагностики в Azure Monitor для отправки данных в концентратор событий Azure, чтобы их можно было перенаправить в расположения за пределами Azure.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/18/2020
ms.openlocfilehash: 19c39632a1ed040636372d6bad53bf6859960b94
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101732043"
---
# <a name="send-data-from-windows-azure-diagnostics-extension-to-azure-event-hubs"></a>Отправка данных из расширения системы диагностики Microsoft Azure в концентраторы событий Azure
Расширение "Диагностика Microsoft Azure" — это агент в Azure Monitor, собирающий данные мониторинга из операционной системы на виртуальной машине и рабочих нагрузок виртуальных машин Azure и других вычислительных ресурсов. В этой статье описывается, как отправлять данные из расширения системы диагностики Microsoft Azure (WAD) в [концентраторы событий Azure](https://azure.microsoft.com/services/event-hubs/) , чтобы вы могли пересылать их в расположения за пределами Azure.

## <a name="supported-data"></a>Поддерживаемые данные

Данные, собранные из операционной системы на виртуальной машине, которые могут быть отправлены в концентраторы событий, включают следующие. Другие источники данных, собранные WAD, включая журналы IIS и аварийные дампы, не могут быть отправлены в концентраторы событий.

* Трассировка событий Windows (ETW)
* Счетчики производительности
* Журналы событий Windows, включая журналы приложений в журнале событий Windows
* Журналы инфраструктуры системы диагностики Azure

## <a name="prerequisites"></a>Предварительные условия

* Расширение диагностики Windows 1,6 или более поздней версии. Дополнительные сведения о поддерживаемых ресурсах см. в разделе [версии схемы конфигурации система диагностики Azure и истории](diagnostics-extension-versions.md) для журнала версий и в [обзоре расширения система диагностики Azure](diagnostics-extension-overview.md) .
* Пространство имен концентраторов событий всегда должно быть подготовлено. Дополнительные сведения см. в статье Начало [работы с концентраторами событий](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md) .


## <a name="configuration-schema"></a>Схема конфигурации
В разделе [Установка и настройка расширения системы диагностики Microsoft Azure (WAD)](diagnostics-extension-windows-install.md) доступны различные параметры для включения и настройки расширения диагностики и [система диагностики Azure схемы конфигурации](diagnostics-extension-schema-windows.md) для ссылки на схему конфигурации. В оставшейся части этой статьи описывается, как использовать эту конфигурацию для отправки данных в концентратор событий. 

Система диагностики Azure всегда отправляет журналы и метрики в учетную запись хранения Azure. Можно настроить один или несколько *приемников данных* , которые отправляют данные в дополнительные расположения. Каждый приемник определяется в [элементе SinksConfig](diagnostics-extension-schema-windows.md#sinksconfig-element) общедоступной конфигурации с конфиденциальной информацией в частной конфигурации. Эта конфигурация для концентраторов событий использует значения, приведенные в следующей таблице.

| Свойство. | Описание |
|:---|:---|
| Имя | Описательное имя приемника. Используется в конфигурации для указания источников данных для отправки в приемник. |
| Url  | URL-адрес концентратора событий в формате \<event-hubs-namespace\> . servicebus.Windows.NET/ \<event-hub-name\> .          |
| SharedAccessKeyName | Имя политики общего доступа для концентратора событий, который имеет по крайней мере центр **отправки** . |
| SharedAccessKey     | Первичный или вторичный ключ из политики общего доступа для концентратора событий. |

Ниже приведены примеры общедоступных и частных конфигураций. Это минимальная конфигурация с одним счетчиком производительности и журналом событий, иллюстрирующими настройку и использование приемника данных концентратора событий. Более сложный пример см. в разделе [Схема конфигурации система диагностики Azure](diagnostics-extension-schema-windows.md) .

### <a name="public-configuration"></a>Открытая конфигурация

```JSON
{
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 5120,
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "myEventHub",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                        "sampleRate": "PT3M"
                    }
                ]
            },
            "WindowsEventLog": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "myEventHub",
                    "DataSource": [
                    {
                        "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                    }
                ]
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "myEventHub",
                    "EventHub": {
                        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
                        "SharedAccessKeyName": "SendRule"
                    }
                }
            ]
        }
    },
    "StorageAccount": "mystorageaccount",
}
```


### <a name="private-configuration"></a>Закрытая конфигурация

```JSON
{
    "storageAccountName": "mystorageaccount",
    "storageAccountKey": "{base64 encoded key}",
    "storageAccountEndPoint": "https://core.windows.net",
    "EventHub": {
        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "{base64 encoded key}"
    }
}
```



## <a name="configuration-options"></a>Варианты настройки
Чтобы отправить данные в приемник данных, необходимо указать атрибут **приемников** в узле источника данных. Место размещения атрибута « **приемники** » определяет область назначения. В следующем примере атрибут **приемников** определен для узла **PerformanceCounter** , который вызывает отправку всех дочерних счетчиков производительности в концентратор событий.

```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "sinks": "MyEventHub",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        }
    ]
}
```


В следующем примере атрибут **приемники** применяется непосредственно к трем счетчикам, что приведет к отправке только этих счетчиков производительности в концентратор событий. 

```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Rejected",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Queued",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        }
    ]
}
```

## <a name="validating-configuration"></a>Проверка конфигурации
Для проверки отправки данных в концентратор событий можно использовать различные методы. Ne простой способ — использовать запись концентраторов событий, как описано в статье [Захват событий через концентраторы событий Azure в хранилище BLOB-объектов Azure или Azure Data Lake Storage](../../event-hubs/event-hubs-capture-overview.md). 


## <a name="troubleshoot-event-hubs-sinks"></a>Устранение неполадок приемников Центров событий

- Просмотрите таблицу службы хранилища Azure **WADDiagnosticInfrastructureLogsTable** , содержащую журналы и ошибки для система диагностики Azure. Для этого можно использовать, например, [обозреватель хранилищ Azure](https://www.storageexplorer.com). Подключитесь к соответствующей учетной записи хранения, просмотрите эту таблицу и выполните запрос по метке времени за последние 24 часа. С помощью этого инструмента можно экспортировать информацию в CSV-файл и открыть его в таком приложении, как Microsoft Excel. Excel упрощает поиск строк телефонных карточек, таких как **EventHubs**, чтобы узнать, какая обнаружена ошибка.  

- Убедитесь в том, что концентратор событий успешно подготовлен. Все сведения о соединении в разделе **PrivateConfig** конфигурации должны соответствовать значениям ресурса, как показано на портале. Убедитесь, что на портале определена политика SAS (*SendRule* в примере) и что предоставлено разрешение *Send* .  

## <a name="next-steps"></a>Дальнейшие действия

* [Общие сведения о Центрах событий](../../event-hubs/event-hubs-about.md)
* [Создание концентратора событий](../../event-hubs/event-hubs-create.md)
* [Часто задаваемые вопросы о Центрах событий](../../event-hubs/event-hubs-faq.md)

<!-- Images. -->
[0]: ../../event-hubs/media/event-hubs-streaming-azure-diags-data/dashboard.png



