---
title: Создание триггеров окон "переворачивающегося" в фабрике данных Azure
description: Узнайте, как создать триггер в фабрике данных Azure, который запускает конвейер в "переворачивающемся" окне.
author: chez-charlie
ms.author: chez
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/25/2020
ms.openlocfilehash: b961939516ac4848da00a3cd01c754c90da805cb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102177731"
---
# <a name="create-a-trigger-that-runs-a-pipeline-on-a-tumbling-window"></a>Создание триггера, который запускает конвейер в "переворачивающемся" окне

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Эта статья содержит шаги по созданию, запуску и мониторингу триггера "переворачивающегося" окна. Дополнительные сведения о триггерах и поддерживаемых типах см. в статье [Выполнение конвейера и триггеры в фабрике данных Azure](concepts-pipeline-execution-triggers.md).

Триггер "переворачивающегося" окна — это тип триггера, который активируется с определенным интервалом начиная с указанного времени запуска, сохраняя состояние. "Переворачивающиеся" окна — это ряд неперекрывающихся и несоприкасающихся интервалов времени фиксированного размера. Триггер "переворачивающегося" окна имеет связь "один к одному" с конвейером и может ссылаться только на один конвейер. Триггер окна "переворачивающегося" — более высокий вес альтернативы триггеру Schedule, предлагающий набор функций для сложных сценариев ([зависимость от других триггеров окон "переворачивающегося"](#tumbling-window-trigger-dependency), повторный [Запуск невыполненного задания](tumbling-window-trigger-dependency.md#monitor-dependencies) и [задание повторной попытки пользователя для конвейеров](#user-assigned-retries-of-pipelines)). Дополнительные сведения о разнице между триггером Schedule и триггером окна "переворачивающегося" см. [здесь](concepts-pipeline-execution-triggers.md#trigger-type-comparison).

## <a name="data-factory-ui"></a>Пользовательский интерфейс Фабрики данных

1. Чтобы создать триггер окна "переворачивающегося" в пользовательском интерфейсе фабрики данных, перейдите на вкладку **триггеры** и нажмите кнопку **создать**. 
1. После открытия панели конфигурация триггера выберите **окно "переворачивающегося"** и определите свойства триггера окна "переворачивающегося". 
1. Когда все будет готово, нажмите кнопку **Сохранить**.

![Создание триггера "переворачивающегося" окна на портале Azure](media/how-to-create-tumbling-window-trigger/create-tumbling-window-trigger.png)

## <a name="tumbling-window-trigger-type-properties"></a>Свойства типа триггера "переворачивающегося" окна

"Переворачивающееся" окно имеет следующие свойства типа триггеров:

```json
{
    "name": "MyTriggerName",
    "properties": {
        "type": "TumblingWindowTrigger",
        "runtimeState": "<<Started/Stopped/Disabled - readonly>>",
        "typeProperties": {
            "frequency": <<Minute/Hour>>,
            "interval": <<int>>,
            "startTime": "<<datetime>>",
            "endTime": <<datetime – optional>>,
            "delay": <<timespan – optional>>,
            "maxConcurrency": <<int>> (required, max allowed: 50),
            "retryPolicy": {
                "count": <<int - optional, default: 0>>,
                "intervalInSeconds": <<int>>,
            },
            "dependsOn": [
                {
                    "type": "TumblingWindowTriggerDependencyReference",
                    "size": <<timespan – optional>>,
                    "offset": <<timespan – optional>>,
                    "referenceTrigger": {
                        "referenceName": "MyTumblingWindowDependency1",
                        "type": "TriggerReference"
                    }
                },
                {
                    "type": "SelfDependencyTumblingWindowTriggerReference",
                    "size": <<timespan – optional>>,
                    "offset": <<timespan>>
                }
            ]
        },
        "pipeline": {
            "pipelineReference": {
                "type": "PipelineReference",
                "referenceName": "MyPipelineName"
            },
            "parameters": {
                "parameter1": {
                    "type": "Expression",
                    "value": "@{concat('output',formatDateTime(trigger().outputs.windowStartTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                },
                "parameter2": {
                    "type": "Expression",
                    "value": "@{concat('output',formatDateTime(trigger().outputs.windowEndTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                },
                "parameter3": "https://mydemo.azurewebsites.net/api/demoapi"
            }
        }
    }
}
```

Таблица ниже содержит обзор основных элементов JSON, связанных с периодичностью выполнения и расписанием триггера "переворачивающегося" окна.

| Элемент JSON | Описание | Type | Допустимые значения | Обязательно |
|:--- |:--- |:--- |:--- |:--- |
| **type** | Тип триггера. Тип является фиксированным значением "Тумблингвиндовтригжер". | Строка | "TumblingWindowTrigger" | Да |
| **runtimeState** | Текущее состояние времени выполнения триггера.<br/>**Примечание**. это элемент \<readOnly> . | Строка | "Started," "Stopped," "Disabled" | Да |
| **импульс** | Строка, представляющая единицу частоты (минуты или часы), с которой выполняется триггер. Если значения даты **startTime** являются более детализированными, чем значение **частоты**, даты **startTime** учитываются при вычислении границ окна. Например, если значение **частоты** соответствует ежечасному выполнению, а значение **startTime** — 2017-09-01T10:10:10Z, первое окно будет (2017-09-01T10:10:10Z, 2017-09-01T11:10:10Z). | Строка | "minute," "hour"  | Да |
| **пределах** | Положительное целое число, указывающее интервал для значения **frequency**, которое определяет, как часто выполняется триггер. Например, если **interval** имеет значение 3, а для элемента **frequency** выбран вариант "hour", триггер будет выполняться один раз каждые 3 часа. <br/>**Примечание**. Минимальный интервал окна составляет 5 минут. | Целое число | Положительное целое число. | Да |
| **startTime**| Первое возникновение, которое может быть в прошлом. Первым интервалом триггера является (**startTime**, **startTime** + **interval**). | Дата/время | Значение даты и времени. | Да |
| **Завершения**| Последнее возникновение, которое может быть в прошлом. | Дата/время | Значение даты и времени. | Да |
| **держивая** | Время задержки до начала обработки данных окна. Запуск конвейера начинается после истечения ожидаемого времени выполнения плюс время **задержки**. **Задержка** определяет, как долго триггер ожидает, прежде чем начать новое выполнение по окончании предыдущего. **Задержка** не изменяет окно **startTime**. Например, значение **задержки** 00:10:00 подразумевает задержку длительностью 10 минут. | Временной диапазон<br/>(чч:мм:сс)  | Значение времени, где время по умолчанию — 00:00:00. | Нет |
| **maxConcurrency** | Количество одновременных выполнений триггеров, запущенных в окнах, которые готовы. Например, чтобы заполнить ежечасные запуски для вчерашних результатов в 24 окнах. Если **maxConcurrency** равно 10, события триггера активируются только для первых 10 окон (00:00–01:00 — 09:00–10:00). После завершения первых 10 активированных выполнений конвейера выполнения триггер запускается для следующих 10 окон (10:00–11:00 — 19:00–20:00). Продолжая пример с **maxConcurrency** равным 10, если 10 окон готовы, значит есть всего 10 выполнений конвейера. Если готово только одно окно, значит готово только 1 выполнение конвейера. | Целое число | Целое число от 1 до 50. | Да |
| **retryPolicy: Count** | Число повторных попыток, после которых выполнение конвейера будет помечено как "Failed".  | Целое число | Целое число, в котором значение по умолчанию — 0 (повторы отсутствуют). | Нет |
| **retryPolicy: intervalInSeconds** | Задержка между повторными попытками (в секундах). | Целое число | Количество секунд, где значение по умолчанию — 30. | Нет |
| **dependsOn: тип** | Тип Тумблингвиндовтригжерреференце. Требуется, если задана зависимость. | Строка |  "Тумблингвиндовтригжердепенденциреференце", "Селфдепенденцитумблингвиндовтригжерреференце" | Нет |
| **dependsOn: размер** | Размер окна "переворачивающегося" зависимостей. | Временной диапазон<br/>(чч:мм:сс)  | Положительное значение TimeSpan, где по умолчанию — размер окна дочернего триггера  | Нет |
| **dependsOn: смещение** | Смещение триггера зависимостей. | Временной диапазон<br/>(чч:мм:сс) |  Значение TimeSpan, которое должно быть отрицательным в самостоятельной зависимости. Если значение не указано, окно совпадает с самим триггером. | Самостоятельная зависимость: Да<br/>Другое: нет  |

> [!NOTE]
> После публикации триггера окна "переворачивающегося" **интервал** и **Частота** не могут быть изменены.

### <a name="windowstart-and-windowend-system-variables"></a>Системные переменные WindowStart и WindowEnd

Системные переменные **WindowStart** и **WindowEnd** триггера "переворачивающегося" окна можно использовать в определении **конвейера** (то есть для части запроса). Передайте системные переменные в качестве параметров конвейера в определении **триггера**. В следующем примере показано, как передавать эти переменные в качестве параметров:

```json
{
    "name": "MyTriggerName",
    "properties": {
        "type": "TumblingWindowTrigger",
            ...
        "pipeline": {
            "pipelineReference": {
                "type": "PipelineReference",
                "referenceName": "MyPipelineName"
            },
            "parameters": {
                "MyWindowStart": {
                    "type": "Expression",
                    "value": "@{concat('output',formatDateTime(trigger().outputs.windowStartTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                },
                "MyWindowEnd": {
                    "type": "Expression",
                    "value": "@{concat('output',formatDateTime(trigger().outputs.windowEndTime,'-dd-MM-yyyy-HH-mm-ss-ffff'))}"
                }
            }
        }
    }
}
```

Чтобы использовать значения системных переменных **WindowStart** и **WindowEnd** в определении конвейера, используйте параметры "MyWindowStart" и "MyWindowEnd" соответственно.

### <a name="execution-order-of-windows-in-a-backfill-scenario"></a>Порядок выполнения окон в сценарии с обратным заполнением

Если время срабатывания триггера в прошлом, то в зависимости от этой формулы, M = (CurrentTime-Тригжерстарттиме)/Тумблингвиндовсизе, триггер будет создавать {M} обратная засыпку (прошлая) выполняется параллельно, учитывая параллелизм триггеров перед выполнением будущих запусков. Порядок выполнения для Windows детерминирован, от старых к новым интервалам. В настоящее время это поведение изменить невозможно.

### <a name="existing-triggerresource-elements"></a>Имеющиеся элементы TriggerResource

Следующие моменты применимы к обновлению существующих элементов **тригжерресаурце** :

* Значение элемента **периодичности** (или размера окна) триггера вместе с элементом **Interval** не может быть изменено после создания триггера. Это необходимо для правильной работы Тригжеррун повторов и оценок зависимостей.
* Если значение для элемента **endTime** триггера изменяется (добавляется или обновляется), состояние окон, которые уже обрабатываются, *не* сбрасывается. Триггер использует новое значение **endTime**. Триггер останавливается, если новое значение **endTime** предшествует окну, которое уже выполняется. В противном случае триггер останавливается, когда встречается новое значение **endTime**.

### <a name="user-assigned-retries-of-pipelines"></a>Число повторных попыток конвейеров, назначенных пользователем

В случае сбоев конвейера триггер окна "переворачивающегося" может автоматически повторить выполнение указанного конвейера, используя те же входные параметры без вмешательства пользователя. Это можно указать с помощью свойства "retryPolicy" в определении триггера.

### <a name="tumbling-window-trigger-dependency"></a>Зависимость триггера окна "переворачивающегося"

Если вы хотите убедиться, что триггер окна "переворачивающегося" выполняется только после успешного выполнения другого триггера "переворачивающегося" окна в фабрике данных, [создайте зависимость окна "переворачивающегося"](tumbling-window-trigger-dependency.md).

### <a name="cancel-tumbling-window-run"></a>Отменить запуск окна "переворачивающегося"

Можно отменить запуски для триггера окна "переворачивающегося", если конкретное окно находится в _состоянии ожидания_, _ожидает зависимости_ или состояние _выполнения_ .

* Если окно находится в **запущенном** состоянии, отмените связанный с ним _Запуск конвейера_, и запуск триггера будет помечен как _отмененный_ позже.
* Если окно находится в состоянии **ожидания** или **ожидает состояния зависимости** , вы можете отменить это окно от наблюдения:

![Отмена триггера окна "переворачивающегося" на странице мониторинга](media/how-to-create-tumbling-window-trigger/cancel-tumbling-window-trigger.png)

Можно также повторно запустить отмененное окно. Повторное выполнение повлечет за собой _последние_ опубликованные определения триггера, а зависимости для указанного окна будут _повторно оценены_ после повторного запуска.

![Повторное выполнение триггера окна "переворачивающегося" для ранее отмененных запусков](media/how-to-create-tumbling-window-trigger/rerun-tumbling-window-trigger.png)

## <a name="sample-for-azure-powershell"></a>Пример для Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

В этом разделе показано, как использовать Azure PowerShell для создания, запуска и мониторинга триггера.

1. Создайте файл JSON с именем **MyTrigger.json** в папке C:\ADFv2QuickStartPSH со следующим содержимым:

    > [!IMPORTANT]
    > Прежде чем сохранить файл JSON, установите значение элемента **startTime** на текущее время в формате UTC. Установите значение элемента **endTime** на один час после текущего времени UTC.

    ```json
    {
      "name": "PerfTWTrigger",
      "properties": {
        "type": "TumblingWindowTrigger",
        "typeProperties": {
          "frequency": "Minute",
          "interval": "15",
          "startTime": "2017-09-08T05:30:00Z",
          "delay": "00:00:01",
          "retryPolicy": {
            "count": 2,
            "intervalInSeconds": 30
          },
          "maxConcurrency": 50
        },
        "pipeline": {
          "pipelineReference": {
            "type": "PipelineReference",
            "referenceName": "DynamicsToBlobPerfPipeline"
          },
          "parameters": {
            "windowStart": "@trigger().outputs.windowStartTime",
            "windowEnd": "@trigger().outputs.windowEndTime"
          }
        },
        "runtimeState": "Started"
      }
    }
    ```

2. Создайте триггер с помощью командлета **Set-AzDataFactoryV2Trigger** :

    ```powershell
    Set-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger" -DefinitionFile "C:\ADFv2QuickStartPSH\MyTrigger.json"
    ```

3. Убедитесь, что состояние триггера **остановлено** с помощью командлета **Get-AzDataFactoryV2Trigger** :

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

4. Запустите триггер с помощью командлета **Start-AzDataFactoryV2Trigger** :

    ```powershell
    Start-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

5. Убедитесь, что состояние триггера **запущено** с помощью командлета **Get-AzDataFactoryV2Trigger** :

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

6. Получение триггера выполняется в Azure PowerShell с помощью командлета **Get-AzDataFactoryV2TriggerRun** . Чтобы получить сведения о выполнениях триггера, периодически выполняйте следующую команду. Обновите значения **TriggerRunStartedAfter** и **TriggerRunStartedBefore** в соответствии со значениями в определении триггера:

    ```powershell
    Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-08T00:00:00" -TriggerRunStartedBefore "2017-12-08T01:00:00"
    ```

Сведения о том, как отслеживать выполнения триггера и конвейера на портале Azure, см. в разделе [Мониторинг конвейера](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в руководстве по [выполнению конвейера и использованию триггеров](concepts-pipeline-execution-triggers.md#trigger-execution).
* [Создание зависимости триггера окна "переворачивающегося"](tumbling-window-trigger-dependency.md).
* Сведения о ссылках на метаданные триггера в конвейере см. в разделе [метаданные триггера справочника при выполнении конвейера](how-to-use-trigger-parameterization.md)
