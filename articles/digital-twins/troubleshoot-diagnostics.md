---
title: Включение журналов диагностики и обращение к ним
titleSuffix: Azure Digital Twins
description: Узнайте, как включить ведение журнала с помощью параметров диагностики, и запросите журналы для немедленного просмотра.
author: baanders
ms.author: baanders
ms.date: 11/9/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: c600ced8896a3847b80d854c9e230310cca4c98d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100588597"
---
# <a name="troubleshooting-azure-digital-twins-diagnostics-logging"></a>Устранение неполадок в Azure Digital двойников: ведение журнала диагностики

Azure Digital двойников может получать журналы для экземпляра службы, чтобы отслеживать его производительность, доступ и другие данные. Вы можете использовать эти журналы, чтобы понять, что происходит в вашем экземпляре Azure Digital двойников, и выполнить анализ основных причин проблем без необходимости обращения в службу поддержки Azure.

В этой статье показано, как [**настроить параметры диагностики**](#turn-on-diagnostic-settings) в [портал Azure](https://portal.azure.com) , чтобы начать сбор журналов из своего экземпляра Azure Digital двойников. Можно также указать место хранения журналов (например, Log Analytics или учетную запись хранения).

В этой статье также содержится список всех [категорий журналов](#log-categories) и [схем журналов](#log-schemas) , собираемых Azure Digital двойников.

После настройки журналов можно также выполнить запрос к [**журналам**](#view-and-query-logs) , чтобы быстро собрать пользовательские аналитические сведения.

## <a name="turn-on-diagnostic-settings"></a>Включить параметры диагностики 

Включите параметры диагностики, чтобы начать сбор журналов в своем экземпляре Azure Digital двойников. Также можно выбрать место, где должны храниться экспортированные журналы. Вот как можно включить параметры диагностики для своего экземпляра Azure Digital двойников.

1. Войдите в [портал Azure](https://portal.azure.com) и перейдите к своему экземпляру Azure Digital двойников. Его можно найти, введя его имя на панели поиска портала. 

2. Выберите **параметры диагностики** в меню и **добавьте параметр диагностики**.

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings.png" alt-text="Снимок экрана со страницей параметров диагностики и кнопкой &quot;Добавить&quot;" lightbox="media/troubleshoot-diagnostics/diagnostic-settings.png":::

3. На следующей странице введите следующие значения:
     * **Имя параметра диагностики**: задайте имя для параметров диагностики.
     * **Сведения о категории**: выберите операции, которые требуется отслеживать, и установите флажки, чтобы включить диагностику для этих операций. Ниже перечислены операции, для которых доступны параметры диагностики:
        - дигиталтвинсоператион
        - евентраутесоператион
        - моделсоператион
        - Queryoperation класса DataService
        - AllMetrics
        
        Дополнительные сведения об этих категориях и содержащихся в них сведениях см. в разделе [*Категории журнала*](#log-categories) ниже.
     * **Сведения о назначении**: выберите, куда вы хотите отправить журналы. Можно выбрать любое сочетание трех параметров:
        - Отправка в Log Analytics
        - "Архивировать в учетной записи хранения";
        - "Передать в концентратор событий";

        Возможно, вам будет предложено ввести дополнительные сведения, если они необходимы для выбора назначения.  
    
4. Сохраните новые настройки. 

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings-details.png" alt-text="Снимок экрана со страницей параметров диагностики, в которой пользователь заполнил имя параметра диагностики и внес некоторые флажки для сведений о категории и сведений о назначении. Кнопка сохранить выделена." lightbox="media/troubleshoot-diagnostics/diagnostic-settings-details.png":::

Новые параметры вступят в силу в течение 10 минут. После этого журналы отобразятся на настроенном целевом объекте на странице **параметры диагностики** для вашего экземпляра. 

Для получения более подробных сведений о параметрах диагностики и их параметрах настройки можно посетить страницу [*Создание параметров диагностики, чтобы отправить журналы и метрики платформы в разные места назначения*](../azure-monitor/essentials/diagnostic-settings.md).

## <a name="log-categories"></a>Категории журналов

Ниже приведены дополнительные сведения о категориях журналов, собираемых Azure Digital двойников.

| Категория журнала | Описание |
| --- | --- |
| ADTModelsOperation | Регистрировать все вызовы API, относящиеся к моделям |
| ADTQueryOperation | Регистрировать все вызовы API, относящиеся к запросам |
| ADTEventRoutesOperation | Регистрация всех вызовов API, относящихся к маршрутам событий, а также выхода событий из Azure Digital двойников в службу конечной точки, например "Сетка событий", "концентраторы событий" и "служебная шина" |
| ADTDigitalTwinsOperation | Регистрация всех вызовов API, относящихся к Azure Digital двойников |

Каждая категория журнала состоит из операций записи, чтения, удаления и действия.  Эти данные сопоставляются с REST API вызовами следующим образом:

| Тип события | REST API операции |
| --- | --- |
| запись | РАЗМЕЩЕНИЕ и исправление |
| Чтение | GET |
| Удалить | DELETE |
| Действие | POST |

Ниже приведен полный список операций и соответствующих [вызовов Azure Digital двойников REST API](/rest/api/azure-digitaltwins/) , зарегистрированных в каждой категории. 

>[!NOTE]
> Каждая категория журнала содержит несколько операций или вызовов REST API. В приведенной ниже таблице каждая категория журналов сопоставляется со всеми операциями и REST API находящиеся под ней вызовы до тех пор, пока не появится следующая категория журнала. 

| Категория журнала | Операция | REST APIные вызовы и другие события |
| --- | --- | --- |
| ADTModelsOperation | Microsoft. Дигиталтвинс/Models/Write | API обновления моделей цифровых двойника |
|  | Microsoft. Дигиталтвинс/Models/Read | Цифровые двойника модели Get по идентификаторам API-интерфейсам ID и List |
|  | Microsoft. Дигиталтвинс/Models/Delete | API удаления моделей Digital двойника |
|  | Microsoft. Дигиталтвинс/Models/Action | Добавление API цифровых двойника моделей |
| ADTQueryOperation | Microsoft. Дигиталтвинс/запрос/действие | API двойников запросов |
| ADTEventRoutesOperation | Microsoft. Дигиталтвинс/евентраутес/запись | API добавления маршрутов событий |
|  | Microsoft. Дигиталтвинс/евентраутес/Read | Маршруты событий получают интерфейсы API ИДЕНТИФИКАТОРов и списков |
|  | Microsoft. Дигиталтвинс/евентраутес/Delete | API удаления маршрутов событий |
|  | Microsoft. Дигиталтвинс/евентраутес/действие | Сбой при попытке публикации событий в службе конечной точки (не вызове API) |
| ADTDigitalTwinsOperation | Microsoft. Дигиталтвинс/дигиталтвинс/запись | Цифровой двойников Добавление, Добавление связи, обновление, обновление компонента |
|  | Microsoft. Дигиталтвинс/дигиталтвинс/Read | Digital двойников получает по ИДЕНТИФИКАТОРу, получение компонента, получение отношения по ИДЕНТИФИКАТОРу, список входящих отношений, список отношений |
|  | Microsoft. Дигиталтвинс/дигиталтвинс/Delete | Цифровые двойников удаление, удаление связи |
|  | Microsoft. Дигиталтвинс/дигиталтвинс/действие | Отправка данных телеметрии компонента Digital двойников, отправка данных телеметрии |

## <a name="log-schemas"></a>Схемы журналов 

Каждая категория журнала имеет схему, определяющую, как сообщается о событиях в этой категории. Каждая отдельная запись журнала хранится в виде текста и форматируется как большой двоичный объект JSON. Поля в журнале и примеры тела JSON приведены для каждого типа журнала ниже. 

`ADTDigitalTwinsOperation`, `ADTModelsOperation` и `ADTQueryOperation` используют последовательную схему журнала API; `ADTEventRoutesOperation` имеет собственную отдельную схему.

### <a name="api-log-schemas"></a>Схемы журналов API

Эта схема журнала является постоянной для `ADTDigitalTwinsOperation` , `ADTModelsOperation` и `ADTQueryOperation` . Он содержит сведения, относящиеся к вызовам API для экземпляра Digital двойников Azure.

Ниже приведены описания полей и свойств для журналов API.

| Имя поля | Тип данных | Описание |
|-----|------|-------------|
| `Time` | Дата и время | Дата и время возникновения этого события в формате UTC |
| `ResourceID` | Строка | Идентификатор ресурса Azure Resource Manager для ресурса, в котором произошло событие |
| `OperationName` | Строка  | Тип действия, выполняемого во время события |
| `OperationVersion` | Строка | Версия API, использованная во время события |
| `Category` | Строка | Тип выдаваемый ресурса |
| `ResultType` | Строка | Результат события |
| `ResultSignature` | Строка | Код состояния HTTP для события |
| `ResultDescription` | Строка | Дополнительные сведения о событии |
| `DurationMs` | Строка | Время, затраченное на выполнение события в миллисекундах |
| `CallerIpAddress` | Строка | Маскированный исходный IP-адрес для события |
| `CorrelationId` | Guid | Клиент предоставил уникальный идентификатор для события |
| `Level` | Строка | Серьезность ведения журнала события |
| `Location` | Строка | Регион, в котором произошло событие |
| `RequestUri` | URI | Конечная точка, используемая во время события |

Ниже приведены примеры фрагментов JSON для этих типов журналов.

#### <a name="adtdigitaltwinsoperation"></a>ADTDigitalTwinsOperation

```json
{
  "time": "2020-03-14T21:11:14.9918922Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/digitaltwins/write",
  "operationVersion": "2020-10-31",
  "category": "DigitalTwinOperation",
  "resultType": "Success",
  "resultSignature": "200",
  "resultDescription": "",
  "durationMs": "314",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "2f6a8e64-94aa-492a-bc31-16b9f0b16ab3",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/digitaltwins/factory-58d81613-2e54-4faa-a930-d980e6e2a884?api-version=2020-10-31"
}
```

#### <a name="adtmodelsoperation"></a>ADTModelsOperation

```json
{
  "time": "2020-10-29T21:12:24.2337302Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/models/write",
  "operationVersion": "2020-10-31",
  "category": "ModelsOperation",
  "resultType": "Success",
  "resultSignature": "201",
  "resultDescription": "",
  "durationMs": "935",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "9dcb71ea-bb6f-46f2-ab70-78b80db76882",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/Models?api-version=2020-10-31",
}
```

#### <a name="adtqueryoperation"></a>ADTQueryOperation

```json
{
  "time": "2020-12-04T21:11:44.1690031Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/query/action",
  "operationVersion": "2020-10-31",
  "category": "QueryOperation",
  "resultType": "Success",
  "resultSignature": "200",
  "resultDescription": "",
  "durationMs": "255",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "1ee2b6e9-3af4-4873-8c7c-1a698b9ac334",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/query?api-version=2020-10-31",
}
```

### <a name="egress-log-schemas"></a>Схемы исходящего журнала

Это схема для `ADTEventRoutesOperation` журналов. Они содержат подробные сведения об исключениях и операциях API, связанных с конечными точками исходящего трафика, подключенными к экземпляру Digital двойников Azure.

|Имя поля | Тип данных | Описание |
|-----|------|-------------|
| `Time` | Дата и время | Дата и время возникновения этого события в формате UTC |
| `ResourceId` | Строка | Идентификатор ресурса Azure Resource Manager для ресурса, в котором произошло событие |
| `OperationName` | Строка  | Тип действия, выполняемого во время события |
| `Category` | Строка | Тип выдаваемый ресурса |
| `ResultDescription` | Строка | Дополнительные сведения о событии |
| `Level` | Строка | Серьезность ведения журнала события |
| `Location` | Строка | Регион, в котором произошло событие |
| `EndpointName` | Строка | Имя конечной точки исходящего трафика, созданной в Azure Digital двойников |

Ниже приведены примеры фрагментов JSON для этих типов журналов.

#### <a name="adteventroutesoperation"></a>ADTEventRoutesOperation

```json
{
  "time": "2020-11-05T22:18:38.0708705Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/eventroutes/action",
  "category": "EventRoutesOperation",
  "resultDescription": "Unable to send EventGrid message to [my-event-grid.westus-1.eventgrid.azure.net] for event Id [f6f45831-55d0-408b-8366-058e81ca6089].",
  "correlationId": "7f73ab45-14c0-491f-a834-0827dbbf7f8e",
  "level": "3",
  "location": "southcentralus",
  "properties": {
    "endpointName": "endpointEventGridInvalidKey"
  }
}
```

## <a name="view-and-query-logs"></a>Просмотр и запросы журналов

Ранее в этой статье вы настроили типы журналов для хранения и указания места хранения.

Чтобы устранить неполадки и создать аналитические данные из этих журналов, можно создать **пользовательские запросы**. Чтобы приступить к работе, можно также воспользоваться преимуществами нескольких примеров запросов, предоставляемых службой, в которых можно найти ответы на часто задаваемые вопросы о своем экземпляре.

Вот как запрашивать журналы для своего экземпляра.

1. Войдите в [портал Azure](https://portal.azure.com) и перейдите к своему экземпляру Azure Digital двойников. Его можно найти, введя его имя на панели поиска портала. 

2. Выберите **журналы** в меню, чтобы открыть страницу «запрос журнала». Страница открывается в окне *запросы*.

    :::image type="content" source="media/troubleshoot-diagnostics/logs.png" alt-text="Снимок экрана, показывающий страницу журналов для экземпляра цифрового двойников Azure. Он переключается с помощью окна запросов, в котором отображаются готовые запросы с именами, заданными после различных параметров журнала, таких как задержка API Дигиталтвин и задержка API модели." lightbox="media/troubleshoot-diagnostics/logs.png":::

    Это предварительно созданные примеры запросов, написанных для различных журналов. Можно выбрать один из запросов, чтобы загрузить его в редактор запросов, и запустить его, чтобы просмотреть эти журналы для своего экземпляра.

    Можно также закрыть окно *запросы* без выполнения каких-либо действий, чтобы перейти непосредственно на страницу редактора запросов, где можно написать или изменить код пользовательского запроса.

3. После выхода из окна *запросы* вы увидите главную страницу редактора запросов. Здесь можно просматривать и редактировать текст примеров запросов, а также создавать собственные запросы с нуля.
    :::image type="content" source="media/troubleshoot-diagnostics/logs-query.png" alt-text="Снимок экрана, показывающий страницу журналов для экземпляра цифрового двойников Azure. Окно запросы исчезло, а вместо этого имеется список различных журналов, панель редактирования с редактируемым кодом запроса и панель, отображающая журнал запросов." lightbox="media/troubleshoot-diagnostics/logs-query.png":::

    В левой области 
    - На вкладке *таблицы* отображаются различные [категории журналов](#log-categories) Azure Digital двойников, которые можно использовать в запросах. 
    - На вкладке *запросы* содержатся примеры запросов, которые можно загрузить в редактор.
    - Вкладка *Фильтр* позволяет настроить отфильтрованное представление данных, возвращаемых запросом.

Дополнительные сведения о запросах журналов и их написании см. [*в разделе Обзор запросов журналов в Azure Monitor*](../azure-monitor/logs/log-query-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о настройке диагностики см. в статье [*Получение и использование данных журнала из ресурсов Azure*](../azure-monitor/essentials/platform-logs-overview.md).
* Сведения о метриках цифровых двойников Azure см. в разделе [*Устранение неполадок: Просмотр метрик с помощью Azure Monitor*](troubleshoot-metrics.md).
* Сведения о том, как включить оповещения для метрик, см. в разделе [*Устранение неполадок: Настройка оповещений*](troubleshoot-alerts.md).