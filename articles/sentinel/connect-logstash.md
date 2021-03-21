---
title: Подключение источников данных с помощью Logstash к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать Logstash для пересылки журналов из внешних источников данных в Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/10/2020
ms.author: yelevin
ms.openlocfilehash: da7d540a4b7982c7f743a7ae968515485b45aa5a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102035434"
---
# <a name="use-logstash-to-connect-data-sources-to-azure-sentinel"></a>Использование Logstash для подключения источников данных к Azure Sentinel

> [!IMPORTANT]
> Прием данных с помощью подключаемого модуля вывода Logstash в настоящее время находится в общедоступной предварительной версии. Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Используя подключаемый модуль выходных данных Sentinel в Azure для **обработчика Logstash Data Collection**, вы можете отправить любой тип журнала, который вы хотите передать через Logstash, непосредственно в рабочую область log Analytics в Azure Sentinel. Ваши журналы будут отправлены в настраиваемую таблицу, которая будет определяться с помощью подключаемого модуля вывода.

Дополнительные сведения о работе с подсистемой сбора данных Logstash см. в статье [Приступая к работе с Logstash](https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html).

## <a name="overview"></a>Обзор

### <a name="architecture-and-background"></a>Архитектура и фон

![Схема архитектуры "скрытый журнал".](./media/connect-logstash/logstash-architecture.png)

Подсистема Logstash состоит из трех компонентов:

- Подключаемые модули ввода: настроенный набор данных из различных источников.
- Подключаемые модули фильтров: обработка и нормализация данных в соответствии с указанными критериями.
- Выходные подключаемые модули: настроенная отправка собранных и обработанных данных различным местам назначения.

> [!NOTE]
> Azure Sentinel поддерживает только собственный предоставленный подключаемый модуль вывода. Он не поддерживает сторонние подключаемые модули вывода для маркеров Azure или любого другого подключаемого модуля Logstash любого типа.

Подключаемый модуль выходных данных Azure Sentinel для Logstash отправляет данные в формате JSON в рабочую область Log Analytics с помощью REST API сборщика данных Log Analytics HTTP. Данные принимаются в пользовательские журналы.

- Дополнительные сведения о [Log Analytics REST API](/rest/api/loganalytics/create-request).
- Дополнительные сведения о [пользовательских журналах](../azure-monitor/agents/data-sources-custom-logs.md).

## <a name="deploy-the-azure-sentinel-output-plugin-in-logstash"></a>Развертывание подключаемого модуля выходных данных Azure Sentinel в Logstash

### <a name="step-1-installation"></a>Шаг 1. Установка

Подключаемый модуль выходных данных Azure-Sentinel доступен в коллекции Logstash.

- Чтобы установить подключаемый модуль ***[Microsoft-Logstash-Output-Azure-loganalytics](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/microsoft-logstash-output-azure-loganalytics)*** , следуйте инструкциям в документе Logstash [Working with plugins](https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html) .
   
- Если у вашей системы Logstash нет доступа к Интернету, следуйте инструкциям в документе по [управлению подключаемым модулем Logstash offline](https://www.elastic.co/guide/en/logstash/current/offline-plugins.html) , чтобы подготовить и использовать автономный пакет подключаемого модуля. (Для этого потребуется создать другую систему Logstash с доступом к Интернету.)

### <a name="step-2-configuration"></a>Шаг 2. Настройка

Используйте сведения в структуре Logstash документа [файла конфигурации](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html) и добавьте подключаемый модуль выходных данных Azure Sentinel в конфигурацию со следующими ключами и значениями. (Правильный синтаксис файла конфигурации показан после таблицы.)

| Имя поля | Тип данных | Описание |
|----------------|---------------|-----------------|
| `workspace_id` | строка | Введите идентификатор GUID рабочей области. * |
| `workspace_key` | строка | Введите идентификатор GUID первичного ключа рабочей области. * |
| `custom_log_table_name` | строка | Задайте имя таблицы, в которую будут поступать журналы. Можно настроить только одно имя таблицы для каждого подключаемого модуля вывода. Таблица журнала появится в столбце Метка Azure в разделе **журналы** **в категории** **пользовательские журналы** с `_CL` суффиксом. |
| `endpoint` | строка | Дополнительное поле. По умолчанию это конечная точка Log Analytics. Используйте это поле, чтобы задать альтернативную конечную точку. |
| `time_generated_field` | строка | Дополнительное поле. Это свойство переопределяет поле **timegenerated** по умолчанию в log Analytics. Введите имя поля метки времени в источнике данных. Данные в этом поле должны соответствовать формату ISO 8601 (). `YYYY-MM-DDThh:mm:ssZ` |
| `key_names` | array | Введите список полей выходной схемы Log Analytics. Каждый элемент списка должен быть заключен в одинарные кавычки и элементы, разделенные запятыми, и весь список, заключенный в квадратные скобки. См. пример ниже. |
| `plugin_flush_interval` | number | Дополнительное поле. Задайте значение, чтобы определить максимальный интервал (в секундах) между передачами сообщений в Log Analytics. Значение по умолчанию — 5. |
    | `amount_resizing` | Логическое | True или False. Включите или отключите механизм автоматического масштабирования, который корректирует размер буфера сообщений в соответствии с объемом полученных данных журнала. |
| `max_items` | number | Дополнительное поле. Применяется только в том случае `amount_resizing` , если задано значение "false". Используется для установки ограничения на размер буфера сообщений (в записях). Значение по умолчанию — 2000.  |
| `azure_resource_id` | строка | Дополнительное поле. Определяет идентификатор ресурса Azure, в котором находятся данные. <br>Значение идентификатора ресурса особенно полезно, если используется [контекст ресурса RBAC](resource-context-rbac.md) для предоставления доступа только к конкретным данным. |
| | | |

* ИДЕНТИФИКАТОР рабочей области и первичный ключ можно найти в ресурсе рабочей области в разделе **Управление агентами**.

#### <a name="sample-configurations"></a>Примеры конфигурации

Ниже приведено несколько примеров конфигураций, в которых используется несколько различных параметров.

- Базовая конфигурация, использующая входной канал филебеат:

   ```ruby
    input {
        beats {
            port => "5044"
        }
    }
    filter {
    }
    output {
        microsoft-logstash-output-azure-loganalytics {
          workspace_id => "4g5tad2b-a4u4-147v-a4r7-23148a5f2c21" # <your workspace id>
          workspace_key => "u/saRtY0JGHJ4Ce93g5WQ3Lk50ZnZ8ugfd74nk78RPLPP/KgfnjU5478Ndh64sNfdrsMni975HJP6lp==" # <your workspace key>
          custom_log_table_name => "tableName"
        }
    }
   ```
    
- Базовая конфигурация, использующая входной канал TCP:

   ```ruby
    input {
        tcp {
            port => "514"
            type => syslog #optional, will effect log type in table
        }
    }
    filter {
    }
    output {
        microsoft-logstash-output-azure-loganalytics {
          workspace_id => "4g5tad2b-a4u4-147v-a4r7-23148a5f2c21" # <your workspace id>
          workspace_key => "u/saRtY0JGHJ4Ce93g5WQ3Lk50ZnZ8ugfd74nk78RPLPP/KgfnjU5478Ndh64sNfdrsMni975HJP6lp==" # <your workspace key>
          custom_log_table_name => "tableName"
        }
    }
   ```
  
- Расширенная конфигурация:

   ```ruby    
    input {
        tcp {
            port => 514
            type => syslog
        }
    }
    filter {
        grok {
            match => { "message" => "<%{NUMBER:PRI}>1 (?<TIME_TAG>[0-9]{4}-[0-9]{1,2}-[0-9]{1,2}T[0-9]{1,2}:[0-9]{1,2}:[0-9]{1,2})[^ ]* (?<HOSTNAME>[^ ]*) %{GREEDYDATA:MSG}" }
        }
    }
    output {
        microsoft-logstash-output-azure-loganalytics {
            workspace_id => "<WS_ID>"
            workspace_key => "${WS_KEY}"
            custom_log_table_name => "logstashCustomTable"
            key_names => ['PRI','TIME_TAG','HOSTNAME','MSG']
            plugin_flush_interval => 5
        }
    } 
   ```

   > [!NOTE]
   > Посетите [репозиторий GitHub](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/microsoft-logstash-output-azure-loganalytics) подключаемого модуля для получения дополнительных сведений о внутренних рабочих характеристиках, конфигурации и параметрах производительности.

### <a name="step-3-restart-logstash"></a>Шаг 3. Перезапустите Logstash

### <a name="step-4-view-incoming-logs-in-azure-sentinel"></a>Шаг 4. Просмотр входящих журналов в Azure Sentinel

1. Убедитесь, что сообщения отправляются в подключаемый модуль вывода.

1. В меню навигации по отметкам Azure щелкните **журналы**. В разделе **таблицы** разверните категорию **пользовательские журналы** . Найдите и щелкните имя указанной таблицы (с `_CL` суффиксом) в конфигурации.

   :::image type="content" source="./media/connect-logstash/logstash-custom-logs-menu.png" alt-text="Снимок экрана: журналы с пользовательскими журналами.":::

1. Чтобы просмотреть записи в таблице, запросите таблицу, используя имя таблицы в качестве схемы.

   :::image type="content" source="./media/connect-logstash/logstash-custom-logs-query.png" alt-text="Снимок экрана запроса на пользовательский журнал для скрытия журнала.":::

## <a name="monitor-output-plugin-audit-logs"></a>Мониторинг журналов аудита подключаемого модуля вывода

Чтобы отслеживать подключение и активность подключаемого модуля выходных данных Azure Sentinel, включите соответствующий файл журнала Logstash. Расположение файла журнала см. в документе с [макетом каталога Logstash](https://www.elastic.co/guide/en/logstash/current/dir-layout.html#dir-layout) .

Если вы не видите данные в этом файле журнала, создайте и отправьте некоторые события локально (через подключаемые модули ввода и фильтрации), чтобы убедиться, что подключаемый модуль вывода получает данные. Sentinel Azure будет поддерживать только проблемы, связанные с подключаемым модулем вывода.

## <a name="next-steps"></a>Следующие шаги

В этом документе вы узнали, как использовать Logstash для подключения внешних источников данных к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Приступите к обнаружению угроз с помощью Azure Sentinel, используя [встроенные](tutorial-detect-threats-built-in.md) или [Настраиваемые](tutorial-detect-threats-custom.md) правила.
