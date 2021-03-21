---
title: Примеры обработчиков данных телеметрии — Application Insights для Java в службе Azure Monitor
description: Изучите примеры, демонстрирующие обработчики телеметрии в Azure Monitor Application Insights для Java.
ms.topic: conceptual
ms.date: 12/29/2020
author: kryalama
ms.custom: devx-track-java
ms.author: kryalama
ms.openlocfilehash: 0978bd669855d264ed6dfa5eeddc45ad499aa2a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101734593"
---
# <a name="telemetry-processor-examples---azure-monitor-application-insights-for-java"></a>Примеры обработчиков данных телеметрии — Application Insights для Java в службе Azure Monitor

В этой статье приведены примеры обработчиков данных телеметрии в Application Insights для Java. Вы найдете образцы для конфигураций включения и исключения. Вы также найдете примеры для обработчиков атрибутов и диапазонов процессоров.
## <a name="include-and-exclude-samples"></a>Примеры включения и исключения

В этом разделе вы узнаете, как включать и исключать диапазоны. Вы также узнаете, как исключить несколько диапазонов и применить выборочную обработку.
### <a name="include-spans"></a>Включить диапазоны

В этом разделе показано, как включить диапазоны для обработчика атрибутов. Диапазоны, которые не соответствуют свойствам, не обрабатываются процессором.

Для соответствия необходимо, чтобы имя диапазона было равно `spanA` или `spanB` . 

Эти диапазоны соответствуют свойствам include, и применяются действия процессора:
* Имя Span1: ' span ' Attributes: {env: dev, test_request: 123, credit_card: 1234}
* Имя Span2: атрибуты "Спанб": {env: dev, test_request: false}
* Имя Span3: ' span ' Attributes: {env: 1, test_request: dev, credit_card: 1234}

Этот диапазон не соответствует свойствам include, а действия процессора не применяются:
* Имя Span4: атрибуты "Спанк": {env: dev, test_request: false}

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="exclude-spans"></a>Исключить диапазоны

В этом разделе показано, как исключить диапазоны для обработчика атрибутов. Диапазоны, соответствующие свойствам, не обрабатываются этим процессором.

Для соответствия необходимо, чтобы имя диапазона было равно `spanA` или `spanB` .

Следующие диапазоны соответствуют свойствам исключения, а действия процессора не применяются:
* Имя Span1: ' span ' Attributes: {env: dev, test_request: 123, credit_card: 1234}
* Имя Span2: атрибуты "Спанб": {env: dev, test_request: false}
* Имя Span3: ' span ' Attributes: {env: 1, test_request: dev, credit_card: 1234}

Этот диапазон не соответствует свойствам исключения, и применяются действия процессора:
* Имя Span4: атрибуты "Спанк": {env: dev, test_request: false}

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="exclude-spans-by-using-multiple-criteria"></a>Исключение диапазонов с помощью нескольких критериев

В этом разделе показано, как исключить диапазоны для обработчика атрибутов. Диапазоны, соответствующие свойствам, не обрабатываются этим процессором.

Для соответствия требуется выполнение следующих условий:
* Атрибут (например, `env` или `dev` ) должен существовать в диапазоне.
* Диапазон должен иметь атрибут с ключом `test_request` .

Следующие диапазоны соответствуют свойствам исключения, а действия процессора не применяются.
* Имя Span1: атрибуты "Спанб": {env: dev, test_request: 123, credit_card: 1234}
* Имя Span2: ' span ' Attributes: {env: dev, test_request: false}

Следующий диапазон не соответствует свойствам исключения, и применяются действия процессора:
* Имя Span3: атрибуты "Спанб": {env: 1, test_request: dev, credit_card: 1234}
* Имя Span4: атрибуты "Спанк": {env: dev, dev_request: false}


```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ],
          "attributes": [
            {
              "key": "env",
              "value": "dev"
            },
            {
              "key": "test_request"
            }
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="selective-processing"></a>Выборочная обработка

В этом разделе показано, как задать набор свойств span, которые указывают, к каким диапазонам должен применяться этот процессор. Свойства include указывают, какие диапазоны должны обрабатываться. Свойства исключения отфильтровывают диапазоны, которые не должны обрабатываться.

В следующей конфигурации эти диапазоны соответствуют свойствам, и применяются действия процессора:

* Имя Span1: атрибуты "Спанб": {env: Production, test_request: 123, credit_card: 1234, redact_trace: "false"}
* Имя Span2: ' span ' Attributes: {env: промежуточное хранение, test_request: false, redact_trace: true}

Эти диапазоны не соответствуют свойствам include, а действия процессора не применяются:
* Имя Span3: атрибуты "Спанб": {env: Production, test_request: true, credit_card: 1234, redact_trace: false}
* Имя Span4: атрибуты "Спанк": {env: dev, test_request: false}

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ]
        },
        "exclude": {
          "matchType": "strict",
          "attributes": [
            {
              "key": "redact_trace",
              "value": "false"
            }
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          },
          {
            "key": "duplicate_key",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```
## <a name="attribute-processor-samples"></a>Образцы обработчика атрибутов

### <a name="insert"></a>Вставить

В следующем примере новый атрибут вставляется `{"attribute1": "attributeValue1"}` в диапазоны, где ключ `attribute1` не существует.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "attribute1",
            "value": "attributeValue1",
            "action": "insert"
          }
        ]
      }
    ]
  }
}
```

### <a name="insert-from-another-key"></a>Вставить из другого ключа

В следующем примере используется значение атрибута, `anotherkey` чтобы вставить новый атрибут `{"newKey": "<value from attribute anotherkey>"}` в диапазоны, где ключ `newKey` не существует. Если атрибут `anotherkey` не существует, новый атрибут не вставляется в диапазоны.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "newKey",
            "fromAttribute": "anotherKey",
            "action": "insert"
          }
        ]
      }
    ]
  }
}
```

### <a name="update"></a>Update

В следующем примере атрибут обновляется на `{"db.secret": "redacted"}` . Он обновляет атрибут `boo` , используя значение атрибута `foo` . Диапазоны без атрибута `boo` не изменяются.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "db.secret",
            "value": "redacted",
            "action": "update"
          },
          {
            "key": "boo",
            "fromAttribute": "foo",
            "action": "update" 
          }
        ]
      }
    ]
  }
}
```

### <a name="delete"></a>Удалить

В следующем примере показано, как удалить атрибут, имеющий ключ `credit_card` .

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="hash"></a>Хэш

В следующем примере показано, как хэшировать существующие значения атрибутов.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "user.email",
            "action": "hash"
          }
        ]
      }
    ]
  }
}
```

### <a name="extract"></a>Extract

В следующем примере показано, как использовать регулярное выражение (Regex) для создания новых атрибутов на основе значения другого атрибута.
Например, `http.url = http://example.com/path?queryParam1=value1,queryParam2=value2` вставляются следующие атрибуты:
* Хттппротокол: `http`
* Хттпдомаин: `example.com`
* HttpPath `path`
* Хттпкуерипарамс: `queryParam1=value1,queryParam2=value2`
* HTTP. URL: *нет* изменений

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "http.url",
            "pattern": "^(?<httpProtocol>.*):\\/\\/(?<httpDomain>.*)\\/(?<httpPath>.*)(\\?|\\&)(?<httpQueryParams>.*)",
            "action": "extract"
          }
        ]
      }
    ]
  }
}
```

В следующем примере показано, как обрабатывать диапазоны с именем диапазона, которое соответствует шаблонам регулярного выражения.
Этот процессор удаляет `token` атрибут. Он скрывает `password` атрибут в диапазонах, где имя диапазона совпадает `auth.*` с именем области и где имя диапазона не совпадает `login.*` .

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "regexp",
          "spanNames": [
            "auth.*"
          ]
        },
        "exclude": {
          "matchType": "regexp",
          "spanNames": [
            "login.*"
          ]
        },
        "actions": [
          {
            "key": "password",
            "value": "obfuscated",
            "action": "update"
          },
          {
            "key": "token",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```


## <a name="span-processor-samples"></a>Примеры процессоров span

### <a name="name-a-span"></a>Назовите диапазон

В следующем примере задаются значения атрибутов `db.svc` , `operation` и `id` . Он формирует новое имя диапазона с помощью этих атрибутов в указанном порядке, разделенных значением `::` .
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "name": {
          "fromAttributes": [
            "db.svc",
            "operation",
            "id"
          ],
          "separator": "::"
        }
      }
    ]
  }
}
```

### <a name="extract-attributes-from-a-span-name"></a>Извлечь атрибуты из имени диапазона

Предположим, что имя диапазона входных данных — `/api/v1/document/12345678/update` . В следующем примере результатом является имя выходного диапазона `/api/v1/document/{documentId}/update` . Он добавляет новый атрибут `documentId=12345678` к диапазону.
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "name": {
          "toAttributes": {
            "rules": [
              "^/api/v1/document/(?<documentId>.*)/update$"
            ]
          }
        }
      }
    ]
  }
}
```

### <a name="extract-attributes-from-a-span-name-by-using-include-and-exclude"></a>Извлечение атрибутов из имени диапазона с помощью include и Exclude

В следующем примере показано, как изменить имя диапазона на `{operation_website}` . Он добавляет атрибут с ключом `operation_website` и значением, `{oldSpanName}` Если диапазон имеет следующие свойства:
- Имя диапазона содержит `/` любое место в строке.
- Имя диапазона не является `donot/change` .
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "include": {
          "matchType": "regexp",
          "spanNames": [
            "^(.*?)/(.*?)$"
          ]
        },
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "donot/change"
          ]
        },
        "name": {
          "toAttributes": {
            "rules": [
              "(?<operation_website>.*?)$"
            ]
          }
        }
      }
    ]
  }
}
```
