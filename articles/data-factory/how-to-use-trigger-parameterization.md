---
title: Передача сведений о триггере в конвейер
description: Сведения о ссылках на метаданные триггера в конвейере
ms.service: data-factory
author: chez-charlie
ms.author: chez
ms.reviewer: ''
ms.topic: conceptual
ms.date: 03/02/2021
ms.openlocfilehash: 50a9f9cd59ebeecae89580c878442eb20788f462
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104593651"
---
# <a name="reference-trigger-metadata-in-pipeline-runs"></a>Ссылки на метаданные триггера при выполнении конвейера

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как можно использовать метаданные триггера, такие как время запуска триггера, при выполнении конвейера.

Иногда конвейеру необходимо понимать и считывать метаданные из триггера, который вызывает его. Например, при запуске триггера окна "переворачивающегося" на основе времени начала и окончания работы окна конвейер будет обрабатывать различные срезы данных или папки. В фабрике данных Azure мы используем параметризации и [системную переменную](control-flow-system-variables.md) для передачи метаданных из триггера в конвейер.

Этот шаблон особенно полезен для [триггера окна "переворачивающегося"](how-to-create-tumbling-window-trigger.md), где триггер предоставляет время начала и окончания работы окна, а также [пользовательский триггер события](how-to-create-custom-event-trigger.md), где триггеры анализируют и обрабатывают значения в [настраиваемом поле _данных_](../event-grid/event-schema.md).

> [!NOTE]
> Другой тип триггера предоставляет различные сведения о метаданных. Дополнительные сведения см. в разделе [системная переменная](control-flow-system-variables.md) .

## <a name="data-factory-ui"></a>Пользовательский интерфейс Фабрики данных

В этом разделе показано, как передать сведения о метаданных из триггера в конвейер в пользовательском интерфейсе фабрики данных Azure.

1. Переход на **холст разработки** и изменение конвейера

1. Щелкните пустой холст, чтобы открыть параметры конвейера. Не выбирайте никаких действий. Возможно, потребуется извлечь панель параметров из нижней части холста, так как она могла быть свернута.

1. Выберите раздел **параметров** и нажмите кнопку **+ создать** , чтобы добавить параметры.

    :::image type="content" source="media/how-to-use-trigger-parameterization/01-create-parameter.png" alt-text="Снимок экрана: Настройка конвейера, показывающая, как определить параметры в конвейере.":::

1. Добавьте триггеры в конвейер, щелкнув **+ триггер**.

1. Создайте или подключите триггер к конвейеру и нажмите кнопку **ОК** .

1. На следующей странице заполните метаданные триггера для каждого параметра. Используйте формат, определенный в [системной переменной](control-flow-system-variables.md) , для получения сведений о триггере. Вам не нужно заполнять данные для всех параметров, просто те, которые будут считать значения метаданных триггера. Например, здесь мы присваиваем время начала запуска триггера *parameter_1*.

    :::image type="content" source="media/how-to-use-trigger-parameterization/02-pass-in-system-variable.png" alt-text="Снимок экрана со страницей определения триггера, показывающий, как передать сведения об активации в параметры конвейера.":::

1. Для использования значений в конвейере используйте параметры _@pipeline (). parameters. ParameterName_, а __не__ системную переменную в определениях конвейера. Например, в нашем случае для считывания времени начала триггера мы будем ссылаться @pipeline () .parameters.parameter_1.

## <a name="json-schema"></a>Схема JSON

Чтобы передать сведения о триггере в запуск конвейера, необходимо обновить и триггер, и код JSON конвейера с помощью раздела _Parameters_ .

### <a name="pipeline-definition"></a>Определение конвейера

В разделе **свойств** Добавьте определения параметров в раздел **параметров** .

```json
{
    "name": "demo_pipeline",
    "properties": {
        "activities": [
            {
                "name": "demo_activity",
                "type": "WebActivity",
                "dependsOn": [],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "url": {
                        "value": "@pipeline().parameters.parameter_2",
                        "type": "Expression"
                    },
                    "method": "GET"
                }
            }
        ],
        "parameters": {
            "parameter_1": {
                "type": "string"
            },
            "parameter_2": {
                "type": "string"
            },
            "parameter_3": {
                "type": "string"
            },
            "parameter_4": {
                "type": "string"
            },
            "parameter_5": {
                "type": "string"
            }
        },
        "annotations": [],
        "lastPublishTime": "2021-02-24T03:06:23Z"
    },
    "type": "Microsoft.DataFactory/factories/pipelines"
}
```

### <a name="trigger-definition"></a>Определение триггера

В разделе **конвейеры** назначьте значения параметров в разделе **Параметры** . Вам не нужно заполнять данные для всех параметров, просто те, которые будут считать значения метаданных триггера.

```json
{
    "name": "trigger1",
    "properties": {
        "annotations": [],
        "runtimeState": "Started",
        "pipelines": [
            {
                "pipelineReference": {
                    "referenceName": "demo_pipeline",
                    "type": "PipelineReference"
                },
                "parameters": {
                    "parameter_1": "@trigger().startTime"
                }
            }
        ],
        "type": "ScheduleTrigger",
        "typeProperties": {
            "recurrence": {
                "frequency": "Minute",
                "interval": 15,
                "startTime": "2021-03-03T04:38:00Z",
                "timeZone": "UTC"
            }
        }
    }
}
```

### <a name="use-trigger-information-in-pipeline"></a>Использование сведений о триггерах в конвейере

Для использования значений в конвейере используйте параметры _@pipeline (). parameters. ParameterName_, а __не__ системную переменную в определениях конвейера.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в руководстве по [выполнению конвейера и использованию триггеров](concepts-pipeline-execution-triggers.md#trigger-execution).
