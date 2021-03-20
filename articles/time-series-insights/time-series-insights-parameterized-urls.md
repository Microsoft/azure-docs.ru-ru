---
title: Совместное использование пользовательских представлений с параметризованными URL-адресами Azure Time Series Insights | Документация Майкрософт
description: Узнайте, как создавать параметризованные URL-адреса для быстрого доступа к пользовательским представлениям обозревателя в службе "аналитика временных рядов Azure".
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.topic: conceptual
ms.workload: big-data
ms.date: 10/02/2020
ms.custom: seodec18
ms.openlocfilehash: 9bf857a66643b1e95ea2559601761a7217babad4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91665333"
---
# <a name="share-a-custom-view-using-a-parameterized-url"></a>Предоставление общего доступа к пользовательскому представлению с помощью параметризованного URL-адреса

Чтобы предоставить общий доступ к пользовательскому представлению в обозревателе службы "аналитика временных рядов Azure", можно программно создать параметризованный URL-адрес пользовательского представления.

Обозреватель службы "аналитика временных рядов Azure" поддерживает параметры запроса URL-адреса для указания представлений в интерфейсе непосредственно из URL-адреса. Например, используя только URL-адрес, можно указать целевую среду, предикат поиска и нужный период времени. Когда пользователь выбирает настраиваемый URL-адрес, интерфейс предоставляет ссылку непосредственно на этот ресурс на портале службы "аналитика временных рядов Azure". Применяются политики доступа к данным.

> [!TIP]
>
> * Просмотрите бесплатную [демонстрацию службы "аналитика временных рядов Azure](https://insights.timeseries.azure.com/samples)".
> * Ознакомьтесь с сопроводительной документацией по [обозревателю службы "аналитика временных рядов Azure](./time-series-insights-explorer.md) ".

## <a name="environment-id"></a>Идентификатор среды

Параметр `environmentId=<guid>` указывает идентификатор целевой среды. Это компонент полного доменного имени для доступа к данным, и его можно найти в правом верхнем углу окна "Общие сведения о среде" в портал Azure. Это все, что предшествует `env.timeseries.azure.com` .

Пример параметра идентификатора среды: `?environmentId=10000000-0000-0000-0000-100000000108`.

## <a name="time"></a>время;

С помощью параметризованного URL-адреса можно указать абсолютное или относительное значение времени.

### <a name="absolute-time-values"></a>Абсолютные значения времени

Для абсолютных значений времени используйте параметры `from=<integer>` и `to=<integer>`.

* `from=<integer>` — это значение начала времени на JavaScript (в миллисекундах) для периода поиска.
* `to=<integer>` — это значение времени окончания на JavaScript (в миллисекундах) для периода поиска.

> [!TIP]
> Чтобы легко преобразовать даты в миллисекунды JavaScript, попробуйте [преобразователь отметки времени & UNIX](https://www.freeformatter.com/epoch-timestamp-to-date-converter.html).

### <a name="relative-time-values"></a>Относительные значения времени

Для относительного значения времени используйте `relativeMillis=<value>` , где *значение* находится в миллисекундах JavaScript из самой последней метки времени, полученной от API.

Например, `&relativeMillis=3600000` позволяет отобразить данные за последние 60 минут.

Допустимые значения соответствуют меню **быстрого времени** в обозревателе службы "аналитика временных рядов Azure" и включают:

* `1800000` (Последние 30 минут)
* `3600000` (Последние 60 мин.)
* `10800000` (Последние 3 часа)
* `21600000` (Последние 6 часов)
* `43200000` (Последние 12 часов)
* `86400000` (Последние 24 часа)
* `604800000` (За последние 7 дней)
* `2592000000` (Последние 30 часов)

### <a name="optional-parameters"></a>Необязательные параметры

`timeSeriesDefinitions=<collection of term objects>`Параметр задает термины предиката, которые будут отображаться в представлении "аналитика временных рядов Azure":

| Параметр | Элемент URL-адреса | Описание |
| --- | --- | --- |
| **name** | `\<string>` | имя *условия*; |
| **сплитби** | `\<string>` | имя столбца, по которому выполняется *разбиение*; |
| **measureName** | `\<string>` | имя столбца *меры*; |
| **предикат** | `\<string>` | предложение *where* для фильтрации на стороне сервера. |
| **усесум** | `true` | Необязательный параметр, который задает использование функции Sum для меры. |

> [!NOTE]
> Если `Events` выбрана мера **усесум** , то счетчик будет выбран по умолчанию.  
> Если `Events` параметр не выбран, по умолчанию выбирается Average. |

* `multiChartStack=<true/false>`Пара "ключ-значение" позволяет включить в диаграмму стек.
* `multiChartSameScale=<true/false>`Пара "ключ-значение" обеспечивает одинаковую шкалу по оси Y для разных терминов в пределах необязательного параметра.  
* `timeBucketUnit=<Unit>&timeBucketSize=<integer>`Позволяет настроить ползунок интервала, чтобы обеспечить более детализированное или более гладкое, более обобщенное представление диаграммы.  
* `timezoneOffset=<integer>`Параметр позволяет задать часовой пояс для просматриваемой диаграммы в качестве смещения в формате UTC.

| Пары | Описание |
| --- | --- |
| `multiChartStack=false` | `true` параметр включен по умолчанию, поэтому он передается `false` в стек. |
| `multiChartStack=false&multiChartSameScale=true` | Чтобы использовать одну и ту же шкалу оси Y для разных условий, должно быть включено наложение.  Это `false` по умолчанию, поэтому передача `true` включает эту функцию. |
| `timeBucketUnit=<Unit>&timeBucketSize=<integer>` | Единицы измерения = `days` , `hours` ,, `minutes` `seconds` , `milliseconds` .  Единицы всегда следует писать прописными буквами. </br> Определите количество единиц, передав нужное целое число для **timeBucketSize**.  |
| `timezoneOffset=-<integer>` | Значение целого числа всегда указывается в миллисекундах. |

> [!NOTE]
> значения **timeBucketUnit** можно сгладить до 7 дней.
> значения **timezoneOffset** не задаются ни в формате UTC, ни в местном времени.

### <a name="examples"></a>Примеры

Чтобы добавить определения временных рядов в среду "аналитика временных рядов Azure" в качестве параметра URL-адреса, добавьте:

```URL parameter
&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},
{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

Используйте примеры определений временных рядов для:

* Идентификатор среды
* Последние 60 минут данных
* Термины (**F1PressureID**, **F2TempStation** и **F3VibrationPL**), составляющие необязательные параметры

Для представления можно создать следующий параметризованный URL-адрес:

```URL
https://insights.timeseries.azure.com/classic/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

[![Параметризованный URL-адрес обозревателя службы "аналитика временных рядов Azure"](media/parameterized-url/share-parameterized-url.png)](media/parameterized-url/share-parameterized-url.png#lightbox)

> [!TIP]
> См. Обозреватель в режиме реального времени, используя приведенный выше пример [URL-адреса](https://insights.timeseries.azure.com/classic/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[%7B%22name%22:%22F1PressureId%22,%22splitBy%22:%22Id%22,%22measureName%22:%22Pressure%22,%22predicate%22:%22%27Factory1%27%22%7D,%7B%22name%22:%22F2TempStation%22,%22splitBy%22:%22Station%22,%22measureName%22:%22Temperature%22,%22predicate%22:%22%27Factory2%27%22%7D,%7B%22name%22:%22F3VibrationPL%22,%22splitBy%22:%22ProductionLine%22,%22measureName%22:%22Vibration%22,%22predicate%22:%22%27Factory3%27%22%7D]
) .

Приведенный выше URL-адрес описывает и отображает параметризованное представление обозревателя службы "аналитика временных рядов Azure".

* Параметризованные предикаты.

  [![Параметризованные предикаты в обозревателе службы "аналитика временных рядов Azure".](media/parameterized-url/share-parameterized-url-predicates.png)](media/parameterized-url/share-parameterized-url-predicates.png#lightbox)

* Общее полное представление диаграммы.

  [![Общее полное представление диаграммы.](media/parameterized-url/share-parameterized-url-full-chart.png)](media/parameterized-url/share-parameterized-url-full-chart.png#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [запрашивать данные с помощью C#](time-series-insights-query-data-csharp.md).

* Сведения о обозревателе службы " [аналитика временных рядов Azure](./time-series-insights-explorer.md)".
