---
title: Azure Service Fabric CLI — параметры телеметрии sfctl
description: Сведения о sfctl, интерфейсе командной строки Azure Service Fabric. Содержит список команд для настройки телеметрии sfctl.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: ef720a14617b4131474d50875701d0ef27df4151
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86245519"
---
# <a name="sfctl-settings-telemetry"></a>sfctl settings telemetry
Настройте локальные параметры телеметрии этого экземпляра sfctl.

sfctl telemetry собирает имя команды без предоставляемых параметров или их значений, версии sfctl, типа операционной системы, версии Python, данных об успешном выполнении или сбое команды, возвращенного сообщения об ошибке.

## <a name="commands"></a>Команды

|Команда|Описание|
| --- | --- |
| set-telemetry | Включение и отключение телеметрии. |

## <a name="sfctl-settings-telemetry-set-telemetry"></a>sfctl settings telemetry set-telemetry
Включение и отключение телеметрии.

### <a name="arguments"></a>Аргументы

|Аргумент|Описание|
| --- | --- |
| --off | Отключение телеметрии. |
| --on | Включение телеметрии. Это значение по умолчанию. |

### <a name="global-arguments"></a>Глобальные аргументы

|Аргумент|Описание|
| --- | --- |
| --debug | Повышение уровня детализации журнала для включения всех журналов отладки. |
| --help -h | Отображение этого справочного сообщения и выход. |
| --output -o | Формат вывода.  Допустимые значения\: json, jsonc, table, tsv.  Значение по умолчанию\: json. |
| --query | Строка запроса JMESPath. Дополнительные сведения и примеры см. на веб-сайте http\://jmespath.org/. |
| --verbose | Повышение уровня детализации журнала. Чтобы включить полные журналы отладки, используйте параметр --debug. |

### <a name="examples"></a>Примеры

Отключение телеметрии.

```
sfctl settings telemetry set_telemetry --off
```

Включение телеметрии.

```
sfctl settings telemetry set_telemetry --on
```


## <a name="next-steps"></a>Дальнейшие действия
- [Настройте](service-fabric-cli.md) Service Fabric CLI.
- Узнайте, как использовать интерфейс командной строки Service Fabric, с помощью [примеров сценариев](./scripts/sfctl-upgrade-application.md).
