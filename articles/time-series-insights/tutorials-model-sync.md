---
title: Синхронизация моделей между Azure Digital двойников и службой "аналитика временных рядов" | Документация Майкрософт
description: Рекомендации и средства, используемые для перевода модели активов в ADT в модель активов в Azure TSI
ms.service: time-series-insights
services: time-series-insights
author: msrsrinivasan
ms.author: radhsrin
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 01/19/2021
ms.custom: dpalled
ms.openlocfilehash: 5e8e5367daa532fa0cc36a18cd28b382e7cd8f78
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98682908"
---
# <a name="model-synchronization-between-azure-digital-twins-and-time-series-insights-gen2"></a>Синхронизация моделей между Azure Digital Twins и Аналитикой временных рядов 2-го поколения

В этой статье рассматриваются рекомендации и средства, используемые для перевода модели активов в Azure Digital двойников (ADT) в модель активов в службе "аналитика временных рядов Azure" (TSI).  Эта статья является второй частью серии руководств, посвященной интеграции Azure Digital двойников со службой "аналитика временных рядов Azure". Интеграция Azure Digital двойников со службой "аналитика временных рядов" позволяет архивировать и отслеживать историю телеметрии и вычисляемых свойств цифрового двойников. Эта серия руководств предназначена для разработчиков, которые работают над интеграцией временных рядов в Azure Digital двойников. В части 1 объясняется  [, как установить конвейер данных, который переносит данные из Azure Digital двойников в службу "аналитика](../digital-twins/how-to-integrate-time-series-insights.md) временных рядов", а вторая часть серии руководств посвящена синхронизации модели активов между Azure Digital двойников и службой "аналитика временных рядов". В этом учебнике объясняются рекомендации по выбору и установке соглашения об именовании ИДЕНТИФИКАТОРов временных рядов (TS ID) и установке иерархий в модели временных рядов (ТСМ) вручную.

## <a name="choosing-a-time-series-id"></a>Выбор идентификатора временного ряда

Идентификатор временных рядов — это уникальный идентификатор, используемый для идентификации ресурсов в "аналитике временных рядов". Данные временных рядов (телеметрии из поля, представляющие собой пары "время-значение") представлены с помощью переменных, перечисленных в поле TSID. В Azure Digital двойников свойства двойника и телеметрии используются для представления состояния двойника и измерений, созданных двойника соответственно. Начиная с текущего дизайна ТСМ, Тсидс должны быть уникальными. Использование идентификаторов двойника двойников в Azure Digital двойников или в сочетании с именем свойства или телеметрии всегда будет выполнять уникальный идентификатор TS в ТСМ. В типичном случае — это **_`<Twin ID>`_** Код TSID, а имя свойства и телеметрии будут переменными в ТСМ. Однако существуют случаи использования, когда предпочтительнее выполнять сведение иерархий активов в аналитике временных рядов с помощью формата составных ключей, например **_`<Twin ID>+ <Delimiter of Choice> + <Name of the Property or Telemetry>`_** . Рассмотрим пример, чтобы объяснить более поздний вариант. Рассмотрим комнату в построении, смоделированную как двойника и двойника ID Room22. Его свойство температуры должно быть переведено как значение **_TSID Room22_TempSetting_** и измерение температуры, которое преобразуется в **_Room22_TempMea_** в ТСМ.

[![Выбор идентификатора временного ряда](media/tutorials-model-sync/choosing-tsid.png)](media/tutorials-model-sync/choosing-tsid.png#lightbox)

## <a name="contextualizing-time-series"></a>Временной ряд изучение

Контекстуальность данных (в основном пространственные в природе) в аналитике временных рядов достигается с помощью иерархий ресурсов, и они используются для простоты навигации по данным через представление в виде дерева в обозревателе служб Time Series Insights. Типы временных рядов и иерархии определяются с помощью модели временных рядов (ТСМ) в "аналитика временных рядов". Типы в ТСМ помогают определить переменные, тогда как уровни иерархии и значения полей экземпляров используются для создания древовидного представления в обозревателе "аналитика временных рядов". Дополнительные сведения о ТСМ см. в [документации по Time Series Insights в Интернете](./concepts-model-overview.md).

В Azure Digital двойников подключение между ресурсами выражается с помощью связей двойника. Отношения двойника — это просто граф подключенных ресурсов. Однако во время анализа временных рядов отношения между ресурсами являются иерархическими. То есть ресурсы совместно используют отношение типа «родители-потомки» OD и представлены с помощью древовидной структуры. Чтобы перевести сведения о связях из Azure Digital двойников в иерархии службы "аналитика временных рядов", необходимо выбрать соответствующие иерархические отношения из Azure Digital двойников. В Azure Digital двойников используется открытый стандартный язык моделирования, называемый Digital двойника Definition Language (ДТДЛ). В моделях ДТДЛ описываются с помощью варианта JSON, который называется JSON-LD. Полные сведения о спецификации см. в [документации по дтдл](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) .

[![Подключение между ресурсами](media/tutorials-model-sync/asset-connection.png)](media/tutorials-model-sync/asset-connection.png#lightbox)

## <a name="translating-graph-representation-in-azure-digital-twins-to-tree-structure-in-time-series-insights"></a>Преобразование представления диаграммы Digital двойников в древовидную структуру в службе "аналитика временных рядов"

В следующих разделах руководства описано несколько основных сценариев преобразования структуры графа в Azure Digital двойников в древовидную структуру в службе "аналитика временных рядов".

Образец системы. в этом учебнике используется следующий пример для объяснения концепций, обсуждаемых ниже. Это простая, вымышленная система управления здания с одним этажем и двумя комнатами. Он имеет три термостатов, по одному в каждой из комнат, а другое — в пол. Кроме того, он также имеет счетчик потока воды, измеряющий поток воды от Room21 до Room22 через канал подключения между комнатами. Просмотр пространственной связи между двойников и имеет оба типа отношений.

1. Наиболее распространенная иерархическая связь. Например, Building40-> Floor01-> Флуртс *->ная температура
1. Не настолько распространенная, циклическая связь. Например, начиная с Building40, Фловмтр можно отслеживать с помощью двух разных путей.

   1. Building40-> Floor01-> Room21-> Фловмтр * — > Flow
   1. Building40-> Floor01-> Room22-> Фловмтр * — > Flow  
      Где "Flow" — это фактическая телеметрии, измеряющая поток воды между Room21 и Room22

> [!Note]
>
> `*` Флуртс означает Флурсермостат, а Фловмтр — индикатор потока, который обычно выбирается в качестве TSID. Температура и поток — это необработанные данные телеметрии, называемые переменными в аналитике временных рядов.

Учитывая текущее ограничение на время анализа временных рядов, что один ресурс не может быть представлен в нескольких ветвях, в следующих разделах описывается моделирование иерархических и циклических связей в аналитике временных рядов.

## <a name="case-1-hierarchical-parent-child-relationship"></a>Вариант 1. иерархическая связь (родитель-потомок)

Это наиболее распространенный тип связи между двойников. моделирование чистого отношения "родители-потомки" объясняется на следующем рисунке. Поля экземпляров и TSID являются производными от идентификаторов двойника, как показано ниже. Хотя поля экземпляров можно вручную обновить с помощью обозревателя службы "аналитика временных рядов", раздел под названием "Обновление полей экземпляра с помощью API" объясняет, как прослушивать изменения модели в Azure Digital двойников и обновлять поля экземпляров в службе "аналитика временных рядов" с помощью функций Azure.

[![Сопоставление идентификаторов двойника](media/tutorials-model-sync/mapping-twin-ids.png)](media/tutorials-model-sync/mapping-twin-ids.png#lightbox)

[![Сопоставление идентификаторов двойника 2](media/tutorials-model-sync/mapping-twin-ids2.png)](media/tutorials-model-sync/mapping-twin-ids2.png#lightbox)

## <a name="case-2-circular-relationship"></a>Вариант 2. циклическая связь

### <a name="circular-relationship-in-azure-digital-twins-to-single-hierarchy-relationship-in-time-series-insights"></a>Циклическая связь в связи Azure Digital двойников с единственной иерархией в службе "аналитика временных рядов"

Учитывая, что код TSID должен быть уникальным и может быть представлен только в одной иерархии, этот вариант представляет _"фловмтр"_ с телеметрии с именем _"Flow"_ непосредственно под двойника _"Room21"_. В будущем, когда аналитика временных рядов может поддерживать несколько представлений временных рядов в ТСМ, данные телеметрии _"Flow"_ будут представлены в _комнате 21_ и в _комнате 22_ .

На следующем снимке экрана показано, как вручную сопоставить идентификаторы двойника в поле экземпляра Azure Digital двойников to instance в ТСМ и в итоговой иерархии в службе "аналитика временных рядов".

[![Сопоставление идентификаторов двойника в Azure Digital двойников](media/tutorials-model-sync/mapping-twin-ids-adt.png)](media/tutorials-model-sync/mapping-twin-ids-adt.png#lightbox)

### <a name="circular-relationship-in-azure-digital-twins-to-multiple-hierarchies-in-time-series-insights-using-duplicates"></a>Циклическое отношение в Azure Digital двойников к нескольким иерархиям в службе "аналитика временных рядов" с использованием дубликатов

В части 1 этого руководства объясняется, как клиентская программа (функция Azure) помогает передавать данные телеметрии из центра Интернета вещей или из других источников событий в Azure Digital двойников. Этот подход предполагает использование той же клиентской программы для обновления соответствующих свойств родительского двойников. В данном примере, при считывании данных телеметрии Фловмтр из центра Интернета вещей и обновлении свойства Flow в Фловмтр двойника, программа также может обновлять соответствующие свойства во всех возможных родительских двойников источника. В нашем примере это будет "аутфловмеа" (определяется с использованием отношения "Flow") свойства Room21 и "инфловмеа" Room22.  На снимке экрана ниже показан окончательный пользовательский интерфейс в обозревателе "аналитика временных рядов". Следует отметить, что у нас есть дубликаты данных, используя этот подход.

[![Обозреватель аналитики временных рядов](media/tutorials-model-sync/tsi-explorer.png)](media/tutorials-model-sync/tsi-explorer.png#lightbox)

Фрагмент кода ниже показывает, как клиентское приложение может перемещаться по связи двойника с помощью интерфейсов API цифровых двойников Azure.

> [!Note]
>
> В этом примере фрагмента кода предполагается, что читатели знакомы с [частью 01](../digital-twins/tutorial-end-to-end.md#set-up-the-sample-function-app) учебника, и это изменение кода было внесено в функцию "процесшубтодтевентс".

```csharp
if (propertyPath.Equals("/Flow"))
{
//Update the flow value property of the flow meter
await AdtUtilities.UpdateTwinProperty(client, twinId, "replace",
propertyPath, "double", propertyValue, log);

//also update the sending end flow
string parentIdOutflow = await AdtUtilities.FindParent(client, twinId,
"outflow", log);
if (parentIdOutflow != null)
await AdtUtilities.UpdateTwinProperty(client, parentIdOutflow, "replace", "outflow", "double", propertyValue, log);
else
log.LogInformation("Unable to find Parent with outflow
relationship for " + twinId );
//and receiving end flow value
string parentIdinflow = await AdtUtilities.FindParent(client, twinId,
"inflow", log);
if (parentIdinflow != null)

await AdtUtilities.UpdateTwinProperty(client, parentIdinflow,
"replace", "inflow", "double", propertyValue, log);
else
log.LogInformation("Unable to find Parent with inflow
relationship for " + twinId);
}
```

## <a name="updating-instance-fields-using-apis"></a>Обновление полей экземпляров с помощью интерфейсов API

В этом разделе руководства рассказывается о том, как прослушивать изменения в Двойниковах Azure Digital, такие как создание, удаление двойников или изменение связей между двойников и обновлением полей экземпляров и иерархий программными средствами с помощью API модели временных рядов Insights. Этот метод обновления модели "аналитика временных рядов" обычно достигается с помощью функций Azure. В Azure Digital двойников уведомления о событиях, такие как добавление и удаление двойника, могут маршрутизировать подчиненные службы, такие как концентраторы событий, которые, в свою очередь, могут быть переданы в функции Azure. Дополнительные сведения о маршрутизации и фильтрации событий см. [здесь](../digital-twins/how-to-manage-routes-portal.md).  В оставшейся части этого раздела объясняется использование API-интерфейсов модели временных рядов Insights в функциях Azure для обновления модели "аналитика временных рядов" в ответ на добавление двойника (один тип изменения модели) в Azure Digital двойников.

### <a name="receiving-and-identifying-twin-addition-event-notification"></a>Получение и определение уведомления о событии сложения двойника

```csharp
[FunctionName("RouteEventsToTsi")]
public async Task Run([EventGridTrigger]EventGridEvent eventGridEvent)
{
    try
    {
        if (eventGridEvent != null && eventGridEvent.Data != null)
        {
            logger.LogInformation($"EventType: {eventGridEvent.EventType}");
            logger.LogInformation($"EventGridEvent: {JsonConvert.SerializeObject(eventGridEvent)}");

            //Shape event and Send event data to event hub and tsi
            await SendEventToEventHubAsync(eventGridEvent).ConfigureAwait(false);

            //If a new twin was created, update the newly created instance in TSI with info retrieved from ADT
            if (eventGridEvent.EventType == Constants.TwinCreateEventType)
            {
                //retrieve building, floor and room of value twin
                var twinInfo = await RetrieveTwinInfoAsync(eventGridEvent).ConfigureAwait(false);
                //Update Tsi instance with type(sensor type), hierarchy(space hierarchy) and instance fields(twin info retrieved above)
                var instance = await CreateInstanceToSendAsync(twinInfo).ConfigureAwait(false);
                var instanceToUpdate = new List<TimeSeriesInstance>() { instance };
                var response = await tsiClient.TimeSeriesInstances.ExecuteBatchAsync(new InstancesBatchRequest(update: instanceToUpdate)).ConfigureAwait(false);
            }
        }
    }
    catch (Exception ex)
    {
        logger.LogError($"Exception: {ex.Message}");
    }
}
```

### <a name="creating-time-series-insights-client-and-adding-instance-details"></a>Создание клиента "аналитика временных рядов" и Добавление сведений об экземпляре

```csharp
private async Task<TimeSeriesInstance> CreateInstanceToSendAsync(Dictionary<string, string> twinInfo)
{
    try
    {
        tsiClient = await GetTSIClientAsync().ConfigureAwait(false);

        var timeSeriesId = new object[] { twinInfo[Constants.DtId] };
        var instances = await tsiClient.TimeSeriesInstances.ExecuteBatchAsync(
            new InstancesBatchRequest(
                get: new InstancesRequestBatchGetOrDelete(
                    new IList<object>[] { timeSeriesId }))).ConfigureAwait(false);
        var instance = instances.Get.First().Instance;

        if (instance != null)
        {
            instance = await AddHierarchyToInstanceAsync(instance).ConfigureAwait(false);
            instance = await AddTypeToInstanceAsync(instance, twinInfo[Constants.TwinType]).ConfigureAwait(false);

            instance.InstanceFields = new Dictionary<string, object>
                            {
                                { "Building", twinInfo[Constants.BuildingName] },
                                { "Floor", twinInfo[Constants.FloorName] },
                                { "Room", twinInfo[Constants.ParentName] }
                            };

            //If value twin is a sensor value twin, add sensor type instance field to diff from spatial value twin
            if (twinInfo[Constants.ParentType] == Constants.Sensor)
            {
                instance.InstanceFields.Add(Constants.SensorType, twinInfo[Constants.TwinType]);
            }
            return instance;
        }
        else
        {
            logger.LogError($"instance with id {twinInfo[Constants.DtId]} not found");
            return new TimeSeriesInstance();
        }
    }
    catch (Exception e)
    {
        logger.LogError(e.Message);
        throw;
    }
}
```

### <a name="applying-hierarchy-info-to-instance"></a>Применение сведений об иерархии к экземпляру

```csharp
private async Task<TimeSeriesInstance> AddHierarchyToInstanceAsync(TimeSeriesInstance instance)
{
    if (instance.HierarchyIds == null)
    {
        TimeSeriesHierarchy spacesHierarchy = null;
        try
        {
            var hierarchy = await RunGetHierarchiesAsync(Constants.SpaceHierarchy).ConfigureAwait(false);
            spacesHierarchy = hierarchy.First(h => h.Name.Equals(Constants.SpaceHierarchy));
            instance.HierarchyIds = new List<Guid?>();
            instance.HierarchyIds.Add(spacesHierarchy.Id);
        }
        catch (Exception ex)
        {
            logger.LogWarning($"Hierarchy 'space hierarchy' not found, {ex}");
            throw;
        }
    }
    return instance;
}
```

## <a name="next-steps"></a>Дальнейшие действия

В третьем ряде руководств вы узнаете, как запрашивать исторические данные из Azure Digital двойников с помощью интерфейсов API службы "аналитика временных рядов". Это выполняемая работа, и раздел будет обновлен при готовности. В то же время читатели должны ссылаться на [документацию по API запросов данных в службе "аналитика временных рядов](./concepts-query-overview.md)".
