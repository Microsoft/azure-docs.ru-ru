---
title: Извлечение данных промышленного Интернета вещей (IIoT) Azure в ADX
description: В этом учебнике описано, как извлечь данные IIoT в ADX.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: 4c344dc09ad6c8aa4b2aa431952fc271d946b60d
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104787311"
---
# <a name="tutorial-pull-azure-industrial-iot-data-into-adx"></a>Учебник. Извлечение данных промышленного Интернета вещей (IIoT) Azure в ADX

Платформа Azure для промышленного центра Интернета вещей объединяет модули Edge и облачные микрослужбы с несколькими службами PaaS Azure, предоставляя возможности для обнаружения промышленных активов и получения данных из них с помощью OPC UA. [Azure Data Explorer (ADX)](https://docs.microsoft.com/azure/data-explorer) — это естественное назначение для данных IIoT с функциями анализа данных, которые позволяют выполнять гибкие запросы к полученным данным с серверов OPC UA, подключенных к центру Интернета вещей через издатель OPC. Несмотря на то что кластер ADX может принимать данные непосредственно из Центра Интернета вещей, платформа IIoT выполняет дальнейшую обработку данных, чтобы сделать их более полезными, прежде чем размещать их в концентраторе событий, предоставляемом при использовании полного развертывания микрослужб (см. сведения об архитектуре платформы IIoT).

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Создание таблицы в ADX.
> * Подключение концентратора событий к кластеру ADX.
> * Анализ данных в ADX.

## <a name="how-to-make-the-data-available-in-the-adx-cluster-to-query-it-effectively"></a>Как сделать данные доступными в кластере ADX для эффективного запроса 

Взглянув на формат сообщения из концентратора событий (как указано в классе Microsoft.Azure.IIoT.OpcUa.Subscriber.Models.MonitoredItemMessageModel), вы увидите подсказку для структуры, которая нужна для схемы таблицы ADX.

![Структура](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-1.png)

Ниже приведены шаги, которые необходимо выполнить, чтобы обеспечить доступность данных в кластере ADX для их эффективного запроса.  
1. Создайте кластер ADX. Если у вас нет кластера ADX, уже подготовленного с помощью платформы IIoT, или вы хотите использовать другой кластер, выполните действия, описанные на [этой странице](https://docs.microsoft.com/azure/data-explorer/create-cluster-database-portal#create-a-cluster). 
2. Включите прием потоковой передачи в кластере ADX, как описано [здесь](https://docs.microsoft.com/azure/data-explorer/ingest-data-streaming#enable-streaming-ingestion-on-your-cluster). 
3. Создайте базу данных ADX, выполнив действия, описанные на [этой странице](https://docs.microsoft.com/azure/data-explorer/create-cluster-database-portal#create-a-database).

На следующем шаге мы будем использовать [веб-интерфейс ADX](https://docs.microsoft.com/azure/data-explorer/web-query-data) для выполнения необходимых запросов. Обязательно добавьте кластер в веб-интерфейс, как описано в этой ссылке.  
 
4. Создайте таблицу в ADX, чтобы поместить полученные данные.  Несмотря на то что класс MonitoredItemMessageModel можно использовать для определения схемы таблицы ADX, рекомендуется сначала принять данные в промежуточную таблицу с одним столбцом [динамического](https://docs.microsoft.com/azure/data-explorer/kusto/query/scalar-data-types/dynamic) типа. Это обеспечит больше гибкости при обработке данных и их использованию в других таблицах (которые могут сочетаться с другими источниками данных), служащими для нескольких вариантов использования. Следующий запрос ADX создает промежуточную таблицу iiot_stage с одним столбцом полезных данных.

```
.create table ['iiot_stage']  (['payload']:dynamic)
```

Кроме того, необходимо добавить сопоставление приема данных JSON, чтобы указать кластеру поместить все сообщение JSON из концентратора событий в промежуточную таблицу.

```
.create table ['iiot_stage'] ingestion json mapping 'iiot_stage_mapping' '[{"column":"payload","path":"$","datatype":"dynamic"}]'
```

5. Теперь наша таблица готова к получению данных из концентратора событий. 
6. Используйте приведенные [здесь](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub#connect-to-the-event-hub) инструкции для подключения концентратора событий к кластеру ADX и приема данных в промежуточную таблицу. Нам потребуется лишь создать подключение, так как у нас уже есть концентратор событий, подготовленный платформой IIoT.  
7. После проверки соединения данные начнут поступать в нашу таблицу. После небольшой задержки в ней можно начать анализ данных. Чтобы просмотреть образец данных из 10 строк, используйте приведенный ниже запрос в веб-интерфейсе ADX. Здесь можно проследить сходство полезных данных с упомянутым ранее классом MonitoredItemMessageModel.

![Запрос](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-2.png)

8. Теперь давайте проанализируем динамические данные в столбце "полезных данных" напрямую. В этом примере мы выберем среднее значение телеметрии, определяемое параметром DisplayName:PositiveTrendData, в течение 10 минут во всех записях, полученных с определенного момента времени (определяемого переменной min_t) let min_t = datetime(2020-10-23); iiot_stage | where todatetime(payload.Timestamp) > min_t | where tostring(payload.DisplayName)== 'PositiveTrendData' | summarize event_avg = avg(todouble(payload.Value)) by bin(todatetime(payload.Timestamp), 10 m)
 
Так как наш столбец "полезных данных" содержит динамический тип данных, необходимо выполнить их преобразование во время запроса, чтобы вычисления выполнялись по правильным типам данных.

![Метка времени полезных данных](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-3.png)

Как упоминалось ранее, прием данных OPC UA в промежуточную таблицу с одним динамическим столбцом обеспечивает гибкость. Однако выполнение преобразования типов данных во время выполнения запроса может привести к задержкам при выполнении запросов, особенно в том случае, если объем данных велик и существует множество параллельных запросов. На этом этапе можно создать другую таблицу с уже определенными типами данных, чтобы избежать преобразования типов данных во время выполнения запроса.
 
9. Создайте таблицу для проанализированных данных, состоящую из ограниченного выбора из содержимого динамических полезных данных в промежуточной таблице. Обратите внимание, что мы создали столбец значений для каждого ожидаемого типа в данных телеметрии.

```
.create table ['iiot_parsed']  
    (['Tag_Timestamp']: datetime ,  
    ['Tag_PublisherId']:string ,  
    ['Tag']:string ,
    ['Tag_Datatype']:string ,  
    ['Tag_NodeId']:string,  
    ['Tag_value_long']:long ,  
    ['Tag_value_double']:double,  
    ['Tag_value_boolean']:bool)
```

10. Создайте функцию (на уровне базы данных) для переноса необходимых данных из промежуточной таблицы. Здесь мы выберем элементы Timestamp, PublisherId, DisplayName, Datatype и NodeId из столбца "полезных данных" и перенесем их как Tag_Timestamp, Tag_PublisherId, Tag, Tag_Datatype, Tag_NodeId. Элемент Value переносится как три различные части на основе элемента DataType.

```
.create-or-alter function fn_InflightParseIIoTEvent()
{
iiot_stage
| extend Tag_Timestamp =  todatetime(payload.Timestamp)
| extend Tag_PublisherId = tostring(payload.PublisherId)
| extend Tag = tostring(payload.DisplayName)
| extend Tag_Datatype = tostring(payload.DataType)
| extend Tag_NodeId = tostring(payload.NodeId)
| extend Tag_value_long = case(Tag_Datatype == "Int64", tolong(payload.Value), long(null))
| extend Tag_value_double = case(Tag_Datatype == "Double", todouble(payload.Value), double(null))
| extend Tag_value_boolean = case(Tag_Datatype == "Boolean", tobool(payload.Value), bool(null))
| project Tag_Timestamp, Tag_PublisherId, Tag, Tag_Datatype, Tag_NodeId, Tag_value_long, Tag_value_double, Tag_value_boolean
}
```

Дополнительные сведения о сопоставлении типов данных в ADX см. [здесь](https://docs.microsoft.com/azure/data-explorer/kusto/query/scalar-data-types/dynamic), а сведения о функций в ADX — на [этой странице](https://docs.microsoft.com/azure/data-explorer/kusto/query/schema-entities/stored-functions).
 
11. Примените функцию из предыдущего шага к проанализированной таблице с помощью политики обновления. [Политика](https://docs.microsoft.com/azure/data-explorer/kusto/management/updatepolicy) обновления указывает, что ADX автоматически добавляет данные в целевую таблицу при вставке новых данных в исходную таблицу на основе запроса преобразования, который выполняется с данными, вставленными в исходную таблицу. Мы можем использовать приведенный ниже запрос, чтобы назначить проанализированную таблицу в качестве целевой, а промежуточную таблицу — в качестве источника для политики обновления, определенной функцией, которая была создана на предыдущем шаге.

```
.alter table iiot_parsed policy update
@'[{"IsEnabled": true, "Source": "iiot_stage", "Query": "fn_InflightParseIIoTEvent()", "IsTransactional": true, "PropagateIngestionProperties": true}]'
```

После выполнения приведенного выше запроса данные начнут поступать и заполнять целевую таблицу iiot_parsed. Данные в iiot_parsed можно просмотреть следующим образом.

![Проанализированная таблица](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-4.png)

12. Давайте посмотрим, как можно повторить аналитику, выполненную на предыдущем шаге. Вычислим среднее значение телеметрии, определяемой параметром DisplayName:PositiveTrendData, в течение 10 минут во всех записях, полученных с определенного момента времени (определяемого переменной min_t). Так как теперь у нас есть значения тега PositveTrendData, которые хранятся в столбце с типом данных double, давайте улучшим производительность запросов.

![Повторение аналитики](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-5.png)

13. Сравним производительность запросов в обоих случаях. Мы можем найти время, затраченное на выполнение запроса с помощью элемента Stats в пользовательском интерфейсе ADX (который находится над результатами запроса).  

![Производительность запроса 1](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-6.png)

![Производительность запроса 2](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-7.png)

Мы видим, что запрос, использующий проанализированную таблицу, примерно вдвое быстрее, чем запрос для промежуточной таблицы. В этом примере у нас небольшой набор данных и параллельные запросы не выполняются, поэтому влияние на время выполнения запроса невелико, однако это сильно повлияет на производительность реалистичной рабочей нагрузки. Поэтому важно учитывать, что различные типы данных разделяются на разные столбцы.

> [!NOTE] 
> Политика обновления работает только с данными, которые поступают в промежуточную таблицу после настройки политики, и не применяется к имеющимся данным. Это нужно учитывать, когда, например, необходимо изменить политику обновления. Подробные сведения можно найти в документации по ADX.

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы узнали, как изменить в конфигурации значения по умолчанию, ознакомьтесь с указанными ниже документами. 

> [!div class="nextstepaction"]
> [Настройка компонентов промышленного Интернета вещей](tutorial-configure-industrial-iot-components.md)

> [!div class="nextstepaction"]
> [Визуализация и анализ данных с помощью Аналитики временных рядов](tutorial-visualize-data-time-series-insights.md)