---
title: Обновление с версии 2. x — Azure Monitor Application Insights Java
description: Обновление с Azure Monitor Application Insights Java 2. x
ms.topic: conceptual
ms.date: 11/25/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 6e1c7e15ff77fd75ff2fb70a6741ea2dd9a4cab8
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102040249"
---
# <a name="upgrading-from-application-insights-java-2x-sdk"></a>Обновление с Application Insights пакета SDK для Java 2. x

Если вы уже используете пакет SDK для Java 2. x Application Insights в приложении, его не нужно удалять.
Агент Java 3,0 определит его, а затем запишет и сопоставьте любые пользовательские данные телеметрии, отправляемые через пакет SDK 2. x, и подавление всех автоматических коллекций, выполненных с помощью пакета SDK 2. x, для предотвращения дублирования данных телеметрии.

Если используется агент Application Insights 2. x, необходимо удалить `-javaagent:` аргумент виртуальной машины Java, указывающий на агент 2. x.

В оставшейся части документа описываются ограничения и изменения, которые могут возникнуть при обновлении 2. x до 3,0, а также некоторые обходные пути, которые могут оказаться полезными.

## <a name="telemetryinitializers-and-telemetryprocessors"></a>Telemetryinitializer и TelemetryProcessors

Пакет SDK 2. x Telemetryinitializer и TelemetryProcessors не будет выполняться при использовании агента 3,0.
Многие из вариантов использования, которые ранее были необходимы, можно разрешить в 3,0, настроив [пользовательские измерения](./java-standalone-config.md#custom-dimensions) или настроив [обработчики данных телеметрии](./java-standalone-telemetry-processors.md).

## <a name="multiple-applications-in-a-single-jvm"></a>Несколько приложений в одном ВИРТУАЛЬНОЙ машины Java

В настоящее время 3,0 поддерживает только одну [строку подключения и имя роли](./java-standalone-config.md#connection-string-and-role-name) для каждого выполняемого процесса. В частности, у вас не может быть нескольких Tomcat веб-приложений в одном развертывании Tomcat, использующих разные строки подключения или имена ролей.

## <a name="operation-names"></a>Имена операций

Имена операций в 3,0 были изменены в целом, чтобы обеспечить лучшее агрегированное представление на Application Insights портале U/X.

:::image type="content" source="media/java-ipa/upgrade-from-2x/operation-names-3-0.png" alt-text="Имена операций в 3,0":::

Однако для некоторых приложений вы по-прежнему предпочитаете агрегированное представление в U/X, предоставленное предыдущими именами операций. в этом случае для репликации предыдущего поведения можно использовать функцию [обработчиков данных телеметрии](./java-standalone-telemetry-processors.md) (Предварительная версия) в 3,0.

### <a name="prefix-the-operation-name-with-the-http-method-get-post-etc"></a>Добавьте префикс для имени операции с помощью метода HTTP ( `GET` , `POST` и т. д.).

В пакете SDK для 2. x имена операций начинаются с префикса http ( `GET` , `POST` и т. д.), например

:::image type="content" source="media/java-ipa/upgrade-from-2x/operation-names-prefixed-by-http-method.png" alt-text="Имена операций с префиксом http":::

Приведенный ниже фрагмент кода настраивает 3 процессора телеметрии, которые комбинируются для репликации предыдущего поведения.
Обработчики данных телеметрии выполняют следующие действия (по порядку):

1. Первый обработчик данных телеметрии является процессором Span (имеет тип `span` ), что означает, что он применяется к `requests` и `dependencies` .

   Он будет соответствовать любому диапазону, имеющему атрибут с именем `http.method` и имеющего имя диапазона, которое начинается с `/` .

   Затем имя этого диапазона будет извлечено в атрибут с именем `tempName` .

2. Второй обработчик данных телеметрии также является процессором Span.

   Он будет соответствовать любому диапазону, имеющему атрибут с именем `tempName` .

   Затем имя диапазона будет обновляться путем сцепления двух атрибутов `http.method` и `tempName` , разделенных пробелом.

3. Последний обработчик данных телеметрии является обработчиком атрибутов (имеет тип `attribute` ), что означает, что он применяется ко всем данным телеметрии, имеющим атрибуты (в настоящее время `requests` `dependencies` и `traces` ).

   Он будет соответствовать любым данным телеметрии, имеющим атрибут с именем `tempName` .

   Затем он удалит атрибут с именем `tempName` , чтобы он не был передан как пользовательское измерение.

```
{
  "preview": {
    "processors": [
      {
        "type": "span",
        "include": {
          "matchType": "regexp",
          "attributes": [
            { "key": "http.method", "value": "" }
          ],
          "spanNames": [ "^/" ]
        },
        "name": {
          "toAttributes": {
            "rules": [ "^(?<tempName>.*)$" ]
          }
        }
      },
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "tempName" }
          ]
        },
        "name": {
          "fromAttributes": [ "http.method", "tempName" ],
          "separator": " "
        }
      },
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "tempName" }
          ]
        },
        "actions": [
          { "key": "tempName", "action": "delete" }
        ]
      }
    ]
  }
}
```

### <a name="set-the-operation-name-to-the-full-path"></a>В качестве имени операции укажите полный путь

Кроме того, в пакете SDK для 2. x в некоторых случаях имена операций содержат полный путь, например

:::image type="content" source="media/java-ipa/upgrade-from-2x/operation-names-with-full-path.png" alt-text="Имена операций с полным путем":::

Приведенный ниже фрагмент кода настраивает 4 процессора телеметрии, которые комбинируются для репликации предыдущего поведения.
Обработчики данных телеметрии выполняют следующие действия (по порядку):

1. Первый обработчик данных телеметрии является процессором Span (имеет тип `span` ), что означает, что он применяется к `requests` и `dependencies` .

   Он будет соответствовать любому диапазону, имеющему атрибут с именем `http.url` .

   Затем имя диапазона будет обновляться `http.url` значением атрибута.

   Это было бы до конца, за исключением того, что он `http.url` выглядит примерно так `http://host:port/path` , и вполне вероятно, что нужна только `/path` часть.

2. Второй обработчик данных телеметрии также является процессором Span.

   Он будет соответствовать любому диапазону, имеющему атрибут с именем `http.url` (иными словами, любой диапазон, которому соответствует первый процессор).

   Затем часть имени диапазона будет извлечена в атрибут с именем `tempName` .

3. Третьим обработчиком данных телеметрии также является процессор Span.

   Он будет соответствовать любому диапазону, имеющему атрибут с именем `tempPath` .

   Затем имя диапазона будет обновляться из атрибута `tempPath` .

4. Последний обработчик данных телеметрии является обработчиком атрибутов (имеет тип `attribute` ), что означает, что он применяется ко всем данным телеметрии, имеющим атрибуты (в настоящее время `requests` `dependencies` и `traces` ).

   Он будет соответствовать любым данным телеметрии, имеющим атрибут с именем `tempPath` .

   Затем он удалит атрибут с именем `tempPath` , чтобы он не был передан как пользовательское измерение.

```
{
  "preview": {
    "processors": [
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "http.url" }
          ]
        },
        "name": {
          "fromAttributes": [ "http.url" ]
        }
      },
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "http.url" }
          ]
        },
        "name": {
          "toAttributes": {
            "rules": [ "https?://[^/]+(?<tempPath>/[^?]*)" ]
          }
        }
      },
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "tempPath" }
          ]
        },
        "name": {
          "fromAttributes": [ "tempPath" ]
        }
      },
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "tempPath" }
          ]
        },
        "actions": [
          { "key": "tempPath", "action": "delete" }
        ]
      }
    ]
  }
}
```

## <a name="dependency-names"></a>Имена зависимостей

Имена зависимостей в 3,0 также изменились, и в общем случае лучше предоставить более эффективное агрегированное представление на портале Application Insights U/X.

Опять же, для некоторых приложений вы по-прежнему предпочитаете агрегированное представление в U/X, которое было предоставлено предыдущими именами зависимостей. в этом случае можно использовать аналогичные методы, описанные выше, для репликации предыдущего поведения.

## <a name="operation-name-on-dependencies"></a>Имя операции для зависимостей

Ранее в пакете SDK для 2. x имя операции из телеметрии запроса было также задано в телеметрии зависимостей.
Application Insights Java 3,0 больше не заполняет имя операции в телеметрии зависимостей.
Если необходимо просмотреть имя операции для запроса, который является родительским по отношению к телеметрии зависимостей, можно написать запрос журналов (Kusto) для объединения из таблицы зависимостей в таблицу запроса, например

```
let start = datetime('...');
let end = datetime('...');
dependencies
| where timestamp between (start .. end)
| project timestamp, type, name, operation_Id
| join (requests
    | where timestamp between (start .. end)
    | project operation_Name, operation_Id)
    on $left.operation_Id == $right.operation_Id
| summarize count() by operation_Name, type, name
```

## <a name="2x-sdk-logging-appenders"></a>Добавление в журнал пакета SDK для 2. x

Агент 3,0 выполняет [Автоматический сбор данных журнала](./java-standalone-config.md#auto-collected-logging) без необходимости настраивать какие-либо дописывать журналы.
Если вы используете дополнение к ведению журнала пакета SDK 2. x, их можно удалить, так как в любом случае они будут подавлены агентом 3,0.

## <a name="2x-sdk-spring-boot-starter"></a>2. x пакет SDK для пружинной загрузки

Начальная загрузка с пружиной 3,0 отсутствует.
Программа установки и настройки агента 3,0 выполняет те же [простые действия](./java-in-process-agent.md#quickstart) , что и при использовании пружинной загрузки.

При обновлении с версии 2. x пакета SDK для начальной загрузки Обратите внимание, что имя роли облака больше не будет использоваться по умолчанию `spring.application.name` .
Сведения о настройке имени облачной роли в 3,0 с помощью JSON config или переменной среды см. в документации по [конфигурации 3,0](./java-standalone-config.md#cloud-role-name) .
