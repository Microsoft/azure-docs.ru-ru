---
title: Мониторинг приложений Python с помощью Azure Monitor | Документация Майкрософт
description: Содержит инструкции по подключению OpenCensus для Python к Azure Monitor
ms.topic: conceptual
ms.date: 09/24/2020
ms.reviewer: mbullwin
ms.custom: devx-track-python
ms.openlocfilehash: 69472da4f774a1dfae86e1891255907ad711175a
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105047428"
---
# <a name="set-up-azure-monitor-for-your-python-application"></a>Настройка Azure Monitor для приложения Python

Azure Monitor теперь поддерживает распределенную трассировку, сбор метрик и ведение журнала приложений Python за счет интеграции с [OpenCensus](https://opencensus.io). В этой статье описывается процесс настройки Опенценсус для Python и отправки данных мониторинга в Azure Monitor.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
- Установка Python. В этой статье используется [Python 3.7.0](https://www.python.org/downloads/release/python-370/), хотя другие версии, скорее всего, будут работать с незначительными изменениями. Пакет SDK поддерживает только версии Python 2,7 и 3.6 +.
- Создайте ресурс [Application Insights](./create-new-resource.md). Вам будет назначен собственный ключ инструментирования (iKey) для ресурса.

## <a name="instrument-with-opencensus-python-sdk-for-azure-monitor"></a>Инструментирование с помощью пакета SDK OpenCensus для Python для Azure Monitor

Установите средства экспорта OpenCensus Azure Monitor.

```console
python -m pip install opencensus-ext-azure
```

> [!NOTE]
> Команда `python -m pip install opencensus-ext-azure` предполагает, что для установки Python у вас задана переменная среды `PATH`. Если эта переменная не настроена, необходимо предоставить полный путь каталога к расположению исполняемого файла Python. Результатом будет команда вроде следующей: `C:\Users\Administrator\AppData\Local\Programs\Python\Python37-32\python.exe -m pip install opencensus-ext-azure`.

Пакет SDK использует три Azure Monitor экспорта для отправки различных типов телеметрии в Azure Monitor. Они отслеживаются, метрики и журналы. Дополнительные сведения об этих типах телеметрии см. в [обзоре платформы данных](../data-platform.md). Используйте следующие инструкции для отправки этих типов телеметрии с помощью трех средств экспорта.

## <a name="telemetry-type-mappings"></a>Сопоставление типов данных телеметрии

Ниже приведены программы экспорта, которые Опенценсус сопоставляются с типами телеметрии, отображаемыми в Azure Monitor.

| Основы наблюдаемости | Тип телеметрии в Azure Monitor    | Объяснение                                         |
|-------------------------|------------------------------------|-----------------------------------------------------|
| Журналы                    | Трассировки, исключения, customEvents   | Данные телеметрии журналов, телеметрии исключений, данные телеметрии событий |
| Метрики                 | customMetrics, performanceCounters | Счетчики производительности пользовательских метрик                |
| Трассировка                 | Зависимости запросов             | Входящие запросы, исходящие запросы                |

### <a name="logs"></a>Журналы

1. Сначала давайте создадим данные метрик локально.

    ```python
    import logging

    logger = logging.getLogger(__name__)

    def valuePrompt():
        line = input("Enter a value: ")
        logger.warning(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Код постоянно запрашивает ввод значения. Для каждого введенного значения выдается запись журнала.

    ```output
    Enter a value: 24
    24
    Enter a value: 55
    55
    Enter a value: 123
    123
    Enter a value: 90
    90
    ```

1. Хотя ввод значений полезен в демонстрационных целях, в конечном счете мы хотим выдать данные журнала для Azure Monitor. Передайте строку подключения непосредственно в средство экспорта. Можно также указать его в переменной среды `APPLICATIONINSIGHTS_CONNECTION_STRING` . Измените код из предыдущего шага, отталкиваясь от следующего примера кода.

    ```python
    import logging
    from opencensus.ext.azure.log_exporter import AzureLogHandler

    logger = logging.getLogger(__name__)

    # TODO: replace the all-zero GUID with your instrumentation key.
    logger.addHandler(AzureLogHandler(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
    )

    def valuePrompt():
        line = input("Enter a value: ")
        logger.warning(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Программа экспорта отправляет данные журнала в Azure Monitor. Эти данные можно найти в `traces`. 

    > [!NOTE]
    > В этом контексте `traces` не совпадает с `tracing` . Здесь `traces` относится тип телеметрии, который вы увидите в Azure Monitor при использовании `AzureLogHandler` . Но `tracing` ссылается на концепцию в опенценсус и относится к [распределенной трассировке](./distributed-tracing.md).

    > [!NOTE]
    > Для корневого средства ведения журнала настраивается уровень ПРЕДУПРЕЖДЕНИй. Это означает, что все отправляемые журналы с меньшим уровнем серьезности игнорируются, а в свою очередь не отправляются на Azure Monitor. Дополнительные сведения см. в [этой документации](https://docs.python.org/3/library/logging.html#logging.Logger.setLevel).

1. Кроме того, можно добавить пользовательские свойства в сообщения журнала в аргументе *ключевого слова, используя* поле custom_dimensions. Эти свойства отображаются в виде пар "ключ-значение" в `customDimensions` Azure Monitor.
    > [!NOTE]
    > Чтобы эта функция работала, необходимо передать словарь в поле custom_dimensions. При передаче аргументов любого другого типа средство ведения журнала игнорирует их.

    ```python
    import logging

    from opencensus.ext.azure.log_exporter import AzureLogHandler

    logger = logging.getLogger(__name__)
    # TODO: replace the all-zero GUID with your instrumentation key.
    logger.addHandler(AzureLogHandler(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
    )

    properties = {'custom_dimensions': {'key_1': 'value_1', 'key_2': 'value_2'}}

    # Use properties in logging statements
    logger.warning('action', extra=properties)
    ```

#### <a name="configure-logging-for-django-applications"></a>Настройка ведения журнала для приложений Django

Вы можете явно настроить ведение журнала в коде приложения, как описано выше, для приложений Django или указать его в конфигурации ведения журнала Django. Этот код может попасть в любой файл, используемый для настройки параметров Django. Сведения о настройке параметров Django см. в разделе [Параметры Django](https://docs.djangoproject.com/en/3.0/topics/settings/). Дополнительные сведения о настройке ведения журнала см. в разделе [Django Logging](https://docs.djangoproject.com/en/3.0/topics/logging/).

```json
LOGGING = {
    "handlers": {
        "azure": {
            "level": "DEBUG",
        "class": "opencensus.ext.azure.log_exporter.AzureLogHandler",
            "instrumentation_key": "<your-ikey-here>",
         },
        "console": {
            "level": "DEBUG",
            "class": "logging.StreamHandler",
            "stream": sys.stdout,
         },
      },
    "loggers": {
        "logger_name": {"handlers": ["azure", "console"]},
    },
}
```

Убедитесь, что используется средство ведения журнала с тем же именем, которое указано в конфигурации.

```python
import logging

logger = logging.getLogger("logger_name")
logger.warning("this will be tracked")
```

#### <a name="send-exceptions"></a>Отправить исключения

Опенценсус Python не выполняет автоматическую отслеживание и отправку данных `exception` телеметрии. Они передаются через исключения с помощью `AzureLogHandler` библиотеки ведения журнала Python. Вы можете добавлять пользовательские свойства так же, как и для обычного ведения журнала.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)
# TODO: replace the all-zero GUID with your instrumentation key.
logger.addHandler(AzureLogHandler(
    connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
)

properties = {'custom_dimensions': {'key_1': 'value_1', 'key_2': 'value_2'}}

# Use properties in exception logs
try:
    result = 1 / 0  # generate a ZeroDivisionError
except Exception:
    logger.exception('Captured an exception.', extra=properties)
```
Поскольку исключения необходимо записывать в журнал явным образом, пользователю нужно заносить в журнал необработанные исключения. Опенценсус не помещает ограничений на то, как пользователь хочет это сделать, если они явно заносить данные телеметрии исключений.

#### <a name="send-events"></a>Отправка событий

Вы можете отправить `customEvent` данные телеметрии точно так же, как и при отправке `trace` телеметрии, за исключением использования `AzureEventHandler` вместо.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureEventHandler

logger = logging.getLogger(__name__)
logger.addHandler(AzureEventHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>'))
logger.setLevel(logging.INFO)
logger.info('Hello, World!')
```

#### <a name="sampling"></a>Выборка

Сведения о выборке в OpenCensus см. в разделе о [выборке в OpenCensus](sampling.md#configuring-fixed-rate-sampling-for-opencensus-python-applications).

#### <a name="log-correlation"></a>Корреляция журнала

Дополнительные сведения о том, как обогатить журналы данными контекста трассировки, см. в разделе [интеграции журналов](./correlation.md#log-correlation) OpenCensus для Python.

#### <a name="modify-telemetry"></a>Изменение телеметрии

Дополнительные сведения об изменении отслеживания телеметрии перед отправкой в Azure Monitor см. в разделе Опенценсус Python [телеметрии обработчики](./api-filtering-sampling.md#opencensus-python-telemetry-processors).


### <a name="metrics"></a>Метрики

Опенценсус. stats поддерживает 4 метода агрегирования, но обеспечивает частичную поддержку для Azure Monitor:

- **Число:** Число точек измерения. Значение является кумулятивным, при перезапуске может увеличиваться и сбрасываться только до 0. 
- **Сумма:** Сумма точек измерения. Значение является кумулятивным, при перезапуске может увеличиваться и сбрасываться только до 0. 
- **Ластвалуе:** Сохраняет последнее записанное значение, удаляет все остальное.
- **Распространение:** Распределение гистограммы точек измерения. Этот метод **не поддерживается программой экспорта Azure**.

### <a name="count-aggregation-example"></a>Пример агрегирования количества

1. Сначала давайте создадим данные метрик локально. Мы создадим простую метрику, чтобы отчислить, сколько раз пользователь нажимает клавишу **Ввод** .

    ```python
    from datetime import datetime
    from opencensus.stats import aggregation as aggregation_module
    from opencensus.stats import measure as measure_module
    from opencensus.stats import stats as stats_module
    from opencensus.stats import view as view_module
    from opencensus.tags import tag_map as tag_map_module

    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder

    prompt_measure = measure_module.MeasureInt("prompts",
                                               "number of prompts",
                                               "prompts")
    prompt_view = view_module.View("prompt view",
                                   "number of prompts",
                                   [],
                                   prompt_measure,
                                   aggregation_module.CountAggregation())
    view_manager.register_view(prompt_view)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    def prompt():
        input("Press enter.")
        mmap.measure_int_put(prompt_measure, 1)
        mmap.record(tmap)
        metrics = list(mmap.measure_to_view_map.get_metrics(datetime.utcnow()))
        print(metrics[0].time_series[0].points[0])

    def main():
        while True:
            prompt()

    if __name__ == "__main__":
        main()
    ```
1. При повторном выполнении кода появится запрос на нажатие клавиши **Ввод**. Метрика создается для контроля числа выбранных операций **ввода** . При каждой записи значение увеличивается, а сведения о метриках отображаются в консоли. Эти сведения включают текущее значение и текущую отметку времени при обновлении метрики.

    ```output
    Press enter.
    Point(value=ValueLong(5), timestamp=2019-10-09 20:58:04.930426)
    Press enter.
    Point(value=ValueLong(6), timestamp=2019-10-09 20:58:06.570167)
    Press enter.
    Point(value=ValueLong(7), timestamp=2019-10-09 20:58:07.138614)
    ```

1. Хотя ввод значений полезен в демонстрационных целях, в конечном счете мы хотим выдать данные метрики для Azure Monitor. Передайте строку подключения непосредственно в средство экспорта. Можно также указать его в переменной среды `APPLICATIONINSIGHTS_CONNECTION_STRING` . Измените код из предыдущего шага, отталкиваясь от следующего примера кода.

    ```python
    from datetime import datetime
    from opencensus.ext.azure import metrics_exporter
    from opencensus.stats import aggregation as aggregation_module
    from opencensus.stats import measure as measure_module
    from opencensus.stats import stats as stats_module
    from opencensus.stats import view as view_module
    from opencensus.tags import tag_map as tag_map_module

    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder

    prompt_measure = measure_module.MeasureInt("prompts",
                                               "number of prompts",
                                               "prompts")
    prompt_view = view_module.View("prompt view",
                                   "number of prompts",
                                   [],
                                   prompt_measure,
                                   aggregation_module.CountAggregation())
    view_manager.register_view(prompt_view)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    # TODO: replace the all-zero GUID with your instrumentation key.
    exporter = metrics_exporter.new_metrics_exporter(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')

    view_manager.register_exporter(exporter)

    def prompt():
        input("Press enter.")
        mmap.measure_int_put(prompt_measure, 1)
        mmap.record(tmap)
        metrics = list(mmap.measure_to_view_map.get_metrics(datetime.utcnow()))
        print(metrics[0].time_series[0].points[0])

    def main():
        while True:
            prompt()

    if __name__ == "__main__":
        main()
    ```

1. Программа экспорта отправляет данные метрик в Azure Monitor с фиксированным интервалом. Значение по умолчанию — каждые 15 секунд. Мы отслеживаем одну метрику, поэтому данные метрик с любыми значениями и отметками времени, которые она содержит, отправляются каждый интервал. Значение является кумулятивным, при перезапуске может увеличиваться и сбрасываться только до 0. Данные можно найти в `customMetrics` , но `customMetrics` Свойства Валуекаунт, Валуесум, Валуемин, Валуемакс и валуестддев не используются эффективно.

#### <a name="performance-counters"></a>Счетчики производительности

По умолчанию средство экспорта метрик отправляет набор счетчиков производительности в Azure Monitor. Это можно отключить, задав для флага `enable_standard_metrics` значение `False` в конструкторе средства экспорта метрик.

```python
...
exporter = metrics_exporter.new_metrics_exporter(
  enable_standard_metrics=False,
  connection_string='InstrumentationKey=<your-instrumentation-key-here>')
...
```

В настоящее время отправляются следующие счетчики производительности:

- Объем доступной памяти (в байтах)
- Процессорное время ЦП (в процентах)
- Частота входящих запросов (в секунду)
- Среднее время выполнения входящего запроса (в миллисекундах)
- Использования ЦП процессом (процент)
- Количество байтов исключительного использования процесса (в байтах)

Эти метрики должны быть доступны в `performanceCounters`. Дополнительные сведения см. в статье [Счетчики производительности](./performance-counters.md).

#### <a name="modify-telemetry"></a>Изменение телеметрии

Сведения о том, как изменить записанную телеметрию перед отправкой в Azure Monitor, см. в разделе Опенценсус Python [телеметрии обработчики](./api-filtering-sampling.md#opencensus-python-telemetry-processors).

### <a name="tracing"></a>Трассировка

> [!NOTE]
> В Опенценсус `tracing` относится к [распределенной трассировке](./distributed-tracing.md). `AzureExporter` отправляет данные телеметрии `requests` и `dependency` в Azure Monitor.

1. Сначала давайте создадим данные трассировки локально. В Python IDLE или любом выбранном текстовом редакторе введите следующий код:

    ```python
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer

    tracer = Tracer(sampler=ProbabilitySampler(1.0))

    def valuePrompt():
        with tracer.span(name="test") as span:
            line = input("Enter a value: ")
            print(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. При многократном запуске кода вам будет предложено ввести значение. При каждой записи значение распечатывается в оболочке. Модуль Опенценсус Python создает соответствующий фрагмент `SpanData` . Проект OpenCensus определяет [трассировку в виде дерева диапазонов](https://opencensus.io/core-concepts/tracing/).
    
    ```output
    Enter a value: 4
    4
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='15ac5123ac1f6847', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:22.805429Z', end_time='2019-06-27T18:21:44.933405Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    Enter a value: 25
    25
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='2e512f846ba342de', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:44.933405Z', end_time='2019-06-27T18:21:46.156787Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    Enter a value: 100
    100
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='f3f9f9ee6db4740a', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:46.157732Z', end_time='2019-06-27T18:21:47.269583Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    ```

1. Хотя ввод значений полезен в демонстрационных целях, в конечном итоге мы хотим выдавать `SpanData` их в Azure Monitor. Передайте строку подключения непосредственно в средство экспорта. Можно также указать его в переменной среды `APPLICATIONINSIGHTS_CONNECTION_STRING` . Измените код из предыдущего шага, отталкиваясь от следующего примера кода.

    ```python
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer

    # TODO: replace the all-zero GUID with your instrumentation key.
    tracer = Tracer(
        exporter=AzureExporter(
            connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000'),
        sampler=ProbabilitySampler(1.0),
    )

    def valuePrompt():
        with tracer.span(name="test") as span:
            line = input("Enter a value: ")
            print(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Теперь при запуске сценария Python вы по-прежнему будете получать запрос на ввод значений, но только значение будет выводиться в оболочке. Созданный объект `SpanData` отправляется в Azure Monitor. Выданные данные диапазонов можно найти в разделе `dependencies`. Дополнительные сведения об исходящих запросах см. в разделе [зависимости](./opencensus-python-dependency.md)Python опенценсус.
Дополнительные сведения о входящих запросах см. в разделе [запросы](./opencensus-python-request.md)Python опенценсус.

#### <a name="sampling"></a>Выборка

Сведения о выборке в OpenCensus см. в разделе о [выборке в OpenCensus](sampling.md#configuring-fixed-rate-sampling-for-opencensus-python-applications).

#### <a name="trace-correlation"></a>Корреляция трассировки

Дополнительные сведения о корреляции телеметрии в данных трассировки см. в Опенценсусе [корреляции телеметрии](./correlation.md#telemetry-correlation-in-opencensus-python)Python.

#### <a name="modify-telemetry"></a>Изменение телеметрии

Дополнительные сведения об изменении отслеживания телеметрии перед отправкой в Azure Monitor см. в разделе Опенценсус Python [телеметрии обработчики](./api-filtering-sampling.md#opencensus-python-telemetry-processors).

## <a name="configure-azure-monitor-exporters"></a>Настройка Azure Monitor экспорта

Как показано ниже, существуют три разных Azure Monitor экспорта, которые поддерживают Опенценсус. Каждый из них отправляет различные типы данных телеметрии в Azure Monitor. Чтобы узнать, какие типы данных телеметрии отправляет каждый экспортируемый объект, см. следующий список.

Каждый из этих средств экспорта принимает те же аргументы для конфигурации, которые передаются через конструкторы. Сведения о каждой из них можно просмотреть здесь:

- `connection_string`: Строка подключения, используемая для подключения к ресурсу Azure Monitor. Имеет приоритет над `instrumentation_key` .
- `enable_standard_metrics`: Используется для `AzureMetricsExporter` . Оповещает средство экспорта о том, что метрики [счетчиков производительности](../essentials/app-insights-metrics.md#performance-counters) автоматически отправляются Azure Monitor. По умолчанию — `True`.
- `export_interval`: Используется для указания частоты экспорта в секундах.
- `instrumentation_key`— Ключ инструментирования, используемый для подключения к ресурсу Azure Monitor.
- `logging_sampling_rate`: Используется для `AzureLogHandler` . Предоставляет частоту выборки [0, 1.0] для экспорта журналов. Значение по умолчанию — 1,0.
- `max_batch_size`: Указывает максимальный размер данных телеметрии, которые экспортируются одновременно.
- `proxies`: Указывает последовательность прокси-серверов, которые следует использовать для отправки данных в Azure Monitor. Дополнительные сведения см. в разделе [учетные записи-посредники](https://requests.readthedocs.io/en/master/user/advanced/#proxies).
- `storage_path`: Путь к папке локального хранилища (неотправленные данные телеметрии). Начиная с `opencensus-ext-azure` v 1.0.3, путь по умолчанию — это временный каталог ОС + `opencensus-python`  +  `your-ikey` . До версии 1.0.3 путь по умолчанию — $USER + `.opencensus`  +  `.azure`  +  `python-file-name` .

## <a name="view-your-data-with-queries"></a>Фильтрация данных с помощью запросов

Данные телеметрии, отправленные из приложения, можно просмотреть на вкладке **Журналы (аналитика)** .

![Снимок экрана панели обзора со вкладкой «Журналы (аналитика)», выделенной красной рамкой](./media/opencensus-python/0010-logs-query.png)

В списке в разделе **Активные**

- Для данных телеметрии, отправляемых с помощью средства экспорта трассировки Azure Monitor, входящие запросы отображаются в разделе `requests`. Исходящие или внутрипроцессные запросы отображаются в разделе `dependencies`.
- Для данных телеметрии, отправляемых с помощью средства экспорта метрик Azure Monitor, отправленные метрики отображаются в разделе `customMetrics`.
- Для данных телеметрии, отправляемых с помощью средства экспорта журналов Azure Monitor, журналы отображаются в разделе `traces`. Исключения отображаются в `exceptions`.

Более подробные сведения об использовании запросов и журналов см. в [Журналы в Azure Monitor](../logs/data-platform-logs.md).

## <a name="learn-more-about-opencensus-for-python"></a>Дополнительные сведения об OpenCensus для Python

* [OpenCensus для Python на GitHub](https://github.com/census-instrumentation/opencensus-python)
* [Настройка](https://github.com/census-instrumentation/opencensus-python/blob/master/README.rst#customization)
* [Средства экспорта Azure Monitor Exporter на GitHub](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)
* [Интеграции OpenCensus](https://github.com/census-instrumentation/opencensus-python#extensions)
* [Примеры приложений Azure Monitor](https://github.com/Azure-Samples/azure-monitor-opencensus-python)

## <a name="next-steps"></a>Дальнейшие действия

* [Отслеживание входящих запросов](./opencensus-python-dependency.md)
* [Отслеживание исходящих запросов](./opencensus-python-request.md)
* [Сопоставление приложений](./app-map.md)
* [Поиск и диагностика проблем производительности с помощью Azure Application Insights](../app/tutorial-performance.md)

### <a name="alerts"></a>видны узлы

* [Тесты доступности](./monitor-web-app-availability.md). Создавайте тесты, позволяющие проверить, доступен ли ваш сайт в Интернете.
* [Интеллектуальная диагностика](./proactive-diagnostics.md). Эти тесты выполняются автоматически, поэтому вам не нужно их настраивать. Благодаря ей вы узнаете о необычном количестве неудачных запросов.
* [Оповещения о метриках](../alerts/alerts-log.md). Настройте оповещения, чтобы получать уведомления в случае, если метрика превысила пороговое значение. Их можно настроить для пользовательских метрик, добавляемых в код приложения.

