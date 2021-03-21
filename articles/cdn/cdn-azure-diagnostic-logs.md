---
title: Журналы диагностики
titleSuffix: Azure Content Delivery Network
description: Узнайте, как использовать журналы диагностики Azure для хранения основных аналитиков, которые позволяют экспортировать метрики использования из конечной точки сети доставки содержимого Azure.
services: cdn
author: asudbring
manager: KumudD
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/15/2020
ms.author: allensu
ms.openlocfilehash: 96e80de5b8b5ab0a046913ce40ca2d7254dd0133
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100573226"
---
# <a name="diagnostic-logs---azure-content-delivery-network"></a>Журналы диагностики — сеть доставки содержимого Azure

Журналы диагностики Azure позволяют просматривать данные основной аналитики и сохранять их в одно или несколько из следующих целевых расположений:

* Учетная запись хранения Azure
* Рабочая область Log Analytics
* Центры событий Azure

Эта функция доступна на конечных точках CDN всех ценовых категорий. 

Журналы диагностики позволяют экспортировать базовые метрики использования из конечной точки CDN в различные источники, чтобы их можно было использовать в настраиваемом виде. Можно выполнить следующие типы экспорта данных:

* экспорт данных в хранилище BLOB-объектов, экспорт в CSV-файл и создание диаграмм в Excel;
* экспорт данных в Центры событий и их сопоставление с данными из других служб Azure;
* Экспорт данных в журналы Azure Monitor и просмотр данных в собственной Log Analytics рабочей области

Для выполнения следующих шагов требуется профиль Azure CDN. Прежде чем продолжить, ознакомьтесь с [разAzure CDN создать профиль и конечную точку](cdn-create-new-endpoint.md) .

## <a name="enable-logging-with-the-azure-portal"></a>Включение ведения журнала с помощью портала Azure

Выполните следующие действия, чтобы включить ведение журнала для конечной точки Azure CDN.

1. Войдите на [портал Azure](https://portal.azure.com). 

2. В портал Azure перейдите ко **всем ресурсам**, которые  ->  **ваш-CDN-Profile**

2. Выберите конечную точку CDN, для которой необходимо включить журналы диагностики:

    :::image type="content" source="./media/cdn-diagnostics-log/02_browse-to-diagnostics-logs.png" alt-text="Выберите конечную точку CDN." border="true":::

3. В разделе **мониторинг** выберите **журналы диагностики** :

    :::image type="content" source="./media/cdn-diagnostics-log/03_diagnostics-logs-options.png" alt-text="Выберите журналы диагностики." border="true":::

### <a name="enable-logging-with-azure-storage"></a>Включение ведения журнала с помощью службы хранилища Azure

Чтобы использовать учетную запись хранения для хранения журналов, выполните следующие действия.

 >[!NOTE] 
 >Для выполнения этих действий требуется учетная запись хранения. Дополнительные сведения см. в статье **[Создание учетной записи хранения Azure](../storage/common/storage-account-create.md?tabs=azure-portal&toc=%2fazure%2fstorage%2fblobs%2ftoc.json)** .
    
1. В качестве **имени параметра диагностики** введите имя для параметров журнала диагностики.
 
2. Выберите **Архивировать в учетной записи хранения**, а затем выберите **CoreAnalytics**. 

3. Для параметра **Хранение (в днях)** выберите число дней хранения. Нулевое значение означает, что журналы будут храниться неограниченно долго. 

4. Выберите подписку и учетную запись хранения для журналов.

    :::image type="content" source="./media/cdn-diagnostics-log/04_diagnostics-logs-storage.png" alt-text="Журналы диагностики — хранилище." border="true":::

3. Щелкните **Сохранить**.

### <a name="send-to-log-analytics"></a>Отправка в Log Analytics

Чтобы использовать Log Analytics для журналов, выполните следующие действия.

>[!NOTE] 
>Для выполнения этих действий требуется рабочая область log Analytics. Дополнительные сведения см. в разделе **[создание log Analytics рабочей области на портал Azure](../azure-monitor/logs/quick-create-workspace.md)** .
    
1. В качестве **имени параметра диагностики** введите имя для параметров журнала диагностики.

2. Выберите **отправить log Analytics**, а затем выберите **CoreAnalytics**. 

3. Выберите подписку и Log Analytics рабочую область для журналов.

   :::image type="content" source="./media/cdn-diagnostics-log/05-la-workspace.png" alt-text="Журналы диагностики — Log Analytics." border="true":::

4. Щелкните **Сохранить**.

### <a name="stream-to-an-event-hub"></a>"Передать в концентратор событий";

Чтобы использовать концентратор событий для журналов, выполните следующие действия.

>[!NOTE] 
>Для выполнения этих действий требуется концентратор событий. Дополнительные сведения см. в разделе **[Краткое руководство. Создание концентратора событий с помощью портал Azure](../event-hubs/event-hubs-create.md)** .
    
1. В качестве **имени параметра диагностики** введите имя для параметров журнала диагностики.

2. Выберите **поток в концентраторе событий**, а затем выберите **CoreAnalytics**. 

3. Выберите подписку и пространство имен концентратора событий для журналов.

   :::image type="content" source="./media/cdn-diagnostics-log/06-eventhub-namespace.png" alt-text="Журналы диагностики — концентратор событий." border="true":::

4. Щелкните **Сохранить**.


## <a name="enable-logging-with-powershell"></a>Включение ведения журнала с помощью PowerShell

Используйте командлеты Azure PowerShell из примера ниже, чтобы включить журналы диагностики.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="enable-diagnostic-logs-in-a-storage-account"></a>Включение журналов диагностики в учетной записи хранения

1. Войдите в Azure PowerShell.

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```

2. Чтобы включить журналы диагностики в учетной записи хранения, введите следующие команды. Замените переменные значениями:

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $storageacct = <your-storage-account-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $storage = Get-AzStorageAccount -ResourceGroupName $rsg -Name $storageacct

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -StorageAccountId $storage.id -Enabled $true -Categories CoreAnalytics
    ```

### <a name="enable-diagnostics-logs-for-log-analytics-workspace"></a>Включение журналов диагностики для рабочей области Log Analytics

1. Войдите в Azure PowerShell.

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Чтобы включить журналы диагностики для рабочей области Log Analytics, введите следующие команды. Замените переменные значениями:

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $workspacename = <your-log-analytics-workspace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $rsg -Name $workspacename

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -WorkspaceId $workspace.ResourceId -Enabled $true -Categories CoreAnalytics
    ```
### <a name="enable-diagnostics-logs-for-event-hub-namespace"></a>Включение журналов диагностики для пространства имен концентратора событий

1. Войдите в Azure PowerShell.

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Чтобы включить журналы диагностики для рабочей области Log Analytics, введите следующие команды. Замените переменные значениями:

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $eventhubname = <your-event-hub-namespace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -EventHubName $eventhubname -Enabled $true -Categories CoreAnalytics
    ```

## <a name="consuming-diagnostics-logs-from-azure-storage"></a>Использование журналов диагностики из службы хранилища Azure
В этом разделе описывается схема основной аналитики CDN, Организация в учетной записи хранения Azure, а также приведен пример кода для загрузки журналов в CSV-файл.

### <a name="using-microsoft-azure-storage-explorer"></a>Использование обозревателя хранилищ Microsoft Azure
Вы можете скачать это средство [на странице обозревателя хранилищ Azure](https://storageexplorer.com/). После скачивания и установки программы настройте в ней использование той же учетной записи хранения Azure, которая была настроена в качестве назначения для журналов диагностики CDN.

1.  Открыть **Обозреватель службы хранилища Microsoft Azure**
2.  Найдите учетную запись хранения.
3.  Разверните узел **Контейнеры больших двоичных объектов** в учетной записи хранения.
4.  Выберите контейнер с именем *insights-logs-coreanalytics*.
5.  Результаты отобразятся на панели справа, начиная с первого уровня, который обозначен как *resourceId=*. Продолжайте выбирать уровни, пока не найдете файл *PT1H.json*. Описание пути см. в следующих данных о [формате пути большого двоичного объекта](cdn-azure-diagnostic-logs.md#blob-path-format).
6.  Каждый файл *PT1H.json* в большом двоичном объекте представляет журналы аналитики за один час для конкретной конечной точки CDN или соответствующего личного домена.
7.  Схема содержимого этого JSON-файла описана в разделе "Схемы журналов основной аналитики".


#### <a name="blob-path-format"></a>Формат пути BLOB-объекта

Журналы основной аналитики создаются каждый час, собираются в один BLOB-объект и хранятся в нем в формате полезных данных JSON. Инструмент "Обозреватель хранилищ" интерпретирует "/" как разделитель каталогов и отображает иерархию. Путь к большому двоичному объекту Azure выглядит так, как если бы он был иерархической структурой и представляет имя большого двоичного объекта. Имя BLOB-объекта соответствует описанному ниже соглашению об именовании.    

```resourceId=/SUBSCRIPTIONS/{Subscription Id}/RESOURCEGROUPS/{Resource Group Name}/PROVIDERS/MICROSOFT.CDN/PROFILES/{Profile Name}/ENDPOINTS/{Endpoint Name}/ y={Year}/m={Month}/d={Day}/h={Hour}/m={Minutes}/PT1H.json```

**Описание полей**

|Значение|Описание|
|-------|---------|
|Идентификатор подписки    |Идентификатор подписки Azure в формате GUID.|
|Имя группы ресурсов |Имя группы ресурсов, к которой относятся ресурсы CDN.|
|Имя профиля |Имя профиля CDN.|
|Имя конечной точки |Имя конечной точки CDN.|
|Year;|  4-значное обозначение года, например 2017|
|Месяц| 2-значное обозначение номера месяца 01 = Январь... 12 = Декабрь|
|День|   2-значное обозначение дня месяца|
|PT1H.json| Фактический JSON-файл, в котором хранятся данные аналитики.|

### <a name="exporting-the-core-analytics-data-to-a-csv-file"></a>Экспорт данных основной аналитики в CSV-файл

Для доступа к базовой аналитике предоставляется пример кода для средства. Это средство выполняет скачивание JSON-файлов с преобразованием в неструктурированные файлы с разделителями-запятыми, на основе которых можно создавать диаграммы или вычислять статистику.

Вот как использовать это средство:

1.  Посетите ссылку GitHub: [https://github.com/Azure-Samples/azure-cdn-samples/tree/master/CoreAnalytics-ExportToCsv](https://github.com/Azure-Samples/azure-cdn-samples/tree/master/CoreAnalytics-ExportToCsv)
2.  Скачайте код.
3.  Следуйте инструкциям по компиляции и настройке.
4.  Запустите средство.
5.  Полученный CSV-файл позволяет просмотреть данные аналитики в простой одноуровневой структуре.

## <a name="log-data-delays"></a>Задержка данных журнала

В следующей таблице представлены задержки данных журналов для профилей **Azure CDN уровня "Стандартный" от Майкрософт**, **Azure CDN уровня "Стандартный" от Akamai** и **Azure CDN уровня "Стандартный" или "Премиум" от Verizon**.

Задержка данных журнала Майкрософт | Задержка данных журнала Verizon | Задержка данных журнала Akamai
--- | --- | ---
Задержка на 1 час. | Задержка составляет 1 час, а от момента распространения конечной точки до начала отображения данных может пройти до 2 часов. | Задержка составляет 24 часа, а данные, созданные более 24 часов назад, начнут отображаться с задержкой до 2 часов. Если данные созданы недавно, задержка отображения журналов может составить до 25 часов.

## <a name="diagnostic-log-types-for-cdn-core-analytics"></a>Типы журналов диагностики для основной аналитики CDN

В настоящее время все предлагаемые журналы основной аналитики корпорации Майкрософт содержат метрики для статистики HTTP-ответов и исходящих данных, зафиксированные в точках присутствия и (или) границах CDN.

### <a name="core-analytics-metrics-details"></a>Сведения о метриках основной аналитики
В следующей таблице приведен список метрик, доступных в основных журналах аналитики для.

* **Azure CDN уровня "Стандартный" от Майкрософт**;
* **Azure CDN уровня "Стандартный" от Akamai**;
* **Azure CDN "Стандартный" или "Премиум" от Verizon**

Не все метрики доступны у всех поставщиков, хотя различия минимальны. В этой таблице также есть сведения о доступности метрик у определенных поставщиков. Метрики доступны только для конечных точек CDN, на которых зарегистрирован трафик.


|Метрика                     | Описание | Microsoft | Verizon | Akamai |
|---------------------------|-------------|-----------|---------|--------|
| RequestCountTotal         | Общее количество запросов за этот период. | Да | Да |Да |
| RequestCountHttpStatus2xx | Количество запросов, на которые возвращены ответы с кодами HTTP 2xx (200, 202 и т. д.). | Да | Да |Да |
| RequestCountHttpStatus3xx | Количество запросов, на которые возвращены ответы с кодами HTTP 3xx (300, 302 и т. д.). | Да | Да |Да |
| RequestCountHttpStatus4xx | Количество запросов, на которые возвращены ответы с кодами HTTP 4xx (400, 404 и т. д.). | Да | Да |Да |
| RequestCountHttpStatus5xx | Количество запросов, на которые возвращены ответы с кодами HTTP 5xx (500, 504 и т. д.). | Да | Да |Да |
| RequestCountHttpStatusOthers | Количество всех остальных кодов HTTP (кроме 2xx–5xx). | Да | Да |Да |
| RequestCountHttpStatus200 | Количество всех запросов, на которые возвращался ответ с кодом HTTP 200. | Да | Нет  |Да |
| RequestCountHttpStatus206 | Количество всех запросов, на которые возвращался ответ с кодом HTTP 206. | Да | Нет  |Да |
| RequestCountHttpStatus302 | Количество всех запросов, на которые возвращался ответ с кодом HTTP 302. | Да | Нет  |Да |
| RequestCountHttpStatus304 | Количество всех запросов, на которые возвращался ответ с кодом HTTP 304. | Да | Нет  |Да |
| RequestCountHttpStatus404 | Количество всех запросов, на которые возвращался ответ с кодом HTTP 404. | Да | Нет  |Да |
| RequestCountCacheHit | Количество всех запросов, в результате которых произошло попадание в кэш. Такие ресурсы возвращаются клиенту непосредственно из точки присутствия. | Да | Да | Нет  |
| RequestCountCacheMiss | Количество всех запросов, в результате которых произошел промах кэша. Промах кэша означает, что ресурс не найден на ближайшей точке подключения к клиенту и был получен из источника. | Да | Да | Нет |
| RequestCountCacheNoCache | Количество всех запросов к ресурсу, кэширование которых запрещено из-за конфигурации пользователя на границе. | Да | Да | Нет |
| RequestCountCacheUncacheable | Число всех запросов к ресурсам, которые не могут быть кэшированы заголовками Cache-Control и срока действия ресурса. Это число означает, что он не должен кэшироваться в POP или HTTP-клиенте. | Да | Да | Нет |
| RequestCountCacheOthers | Количество всех запросов с состоянием кэша, не указанным выше. | Нет | Да | Нет  |
| EgressTotal | Объем передачи исходящих данных в ГБ. | Да |Да |Да |
| EgressHttpStatus2xx | Объем передачи исходящих данных* для ответов с кодами состояния HTTP 2xx (ГБ). | Да | Да | Нет  |
| EgressHttpStatus3xx | Объем передачи исходящих данных для ответов с кодами состояния HTTP 3xx (ГБ). | Да | Да | Нет  |
| EgressHttpStatus4xx | Объем передачи исходящих данных для ответов с кодами состояния HTTP 4xx (ГБ). | Да | Да | Нет  |
| EgressHttpStatus5xx | Объем передачи исходящих данных для ответов с кодами состояния HTTP 5xx (ГБ). | Да | Да | Нет |
| EgressHttpStatusOthers | Объем передачи исходящих данных для ответов с другими кодами состояния HTTP (ГБ). | Да | Да | Нет  |
| EgressCacheHit | Объем передачи исходящих данных для ответов, которые были доставлены непосредственно из кэша CDN на точки присутствия и границы CDN. | Да | Да | Нет |
| EgressCacheMiss. | Передача исходящих данных для ответов, которые не были найдены на ближайшем сервере POP, и получены с сервера источника. | Да | Да | Нет |
| EgressCacheNoCache | Передача исходящих данных для ресурсов, которые запрещено кэшировать из-за пользовательской конфигурации на границе. | Да | Да | Нет |
| EgressCacheUncacheable | Передача исходящих данных для ресурсов, которые не могут быть кэшированы с помощью заголовков Cache-Control и или срока действия ресурса. Указывает, что он не должен кэшироваться в POP или HTTP-клиенте. | Да | Да | Нет |
| EgressCacheOthers | Объем передачи исходящих данных для других сценариев с использованием кэша. | Нет | Да | Нет |

*Передача исходящих данных — это трафик, который доставляется с POP-серверов CDN клиенту.


### <a name="schema-of-the-core-analytics-logs"></a>Схема журналов основной аналитики 

Все журналы хранятся в формате JSON, и каждая запись содержит строковые поля, как показано на схеме ниже.

```json
    "records": [
        {
            "time": "2017-04-27T01:00:00",
            "resourceId": "<ARM Resource Id of the CDN Endpoint>",
            "operationName": "Microsoft.Cdn/profiles/endpoints/contentDelivery",
            "category": "CoreAnalytics",
            "properties": {
                "DomainName": "<Name of the domain for which the statistics is reported>",
                "RequestCountTotal": integer value,
                "RequestCountHttpStatus2xx": integer value,
                "RequestCountHttpStatus3xx": integer value,
                "RequestCountHttpStatus4xx": integer value,
                "RequestCountHttpStatus5xx": integer value,
                "RequestCountHttpStatusOthers": integer value,
                "RequestCountHttpStatus200": integer value,
                "RequestCountHttpStatus206": integer value,
                "RequestCountHttpStatus302": integer value,
                "RequestCountHttpStatus304": integer value,
                "RequestCountHttpStatus404": integer value,
                "RequestCountCacheHit": integer value,
                "RequestCountCacheMiss": integer value,
                "RequestCountCacheNoCache": integer value,
                "RequestCountCacheUncacheable": integer value,
                "RequestCountCacheOthers": integer value,
                "EgressTotal": double value,
                "EgressHttpStatus2xx": double value,
                "EgressHttpStatus3xx": double value,
                "EgressHttpStatus4xx": double value,
                "EgressHttpStatus5xx": double value,
                "EgressHttpStatusOthers": double value,
                "EgressCacheHit": double value,
                "EgressCacheMiss": double value,
                "EgressCacheNoCache": double value,
                "EgressCacheUncacheable": double value,
                "EgressCacheOthers": double value,
            }
        }

    ]
}
```

Здесь *time* представляет начало периода, для которого выводятся статистические данные. Метрика, не поддерживаемая поставщиком CDN, а не двойное или целочисленное значение, приводит к значению NULL. Это значение указывает на отсутствие метрики, то есть по смыслу отличается от значения 0. Один набор этих метрик для каждого домена настраивается на конечной точке.

Пример набора свойств:

```json
{
     "DomainName": "manlingakamaitest2.azureedge.net",
     "RequestCountTotal": 480,
     "RequestCountHttpStatus2xx": 480,
     "RequestCountHttpStatus3xx": 0,
     "RequestCountHttpStatus4xx": 0,
     "RequestCountHttpStatus5xx": 0,
     "RequestCountHttpStatusOthers": 0,
     "RequestCountHttpStatus200": 480,
     "RequestCountHttpStatus206": 0,
     "RequestCountHttpStatus302": 0,
     "RequestCountHttpStatus304": 0,
     "RequestCountHttpStatus404": 0,
     "RequestCountCacheHit": null,
     "RequestCountCacheMiss": null,
     "RequestCountCacheNoCache": null,
     "RequestCountCacheUncacheable": null,
     "RequestCountCacheOthers": null,
     "EgressTotal": 0.09,
     "EgressHttpStatus2xx": null,
     "EgressHttpStatus3xx": null,
     "EgressHttpStatus4xx": null,
     "EgressHttpStatus5xx": null,
     "EgressHttpStatusOthers": null,
     "EgressCacheHit": null,
     "EgressCacheMiss": null,
     "EgressCacheNoCache": null,
     "EgressCacheUncacheable": null,
     "EgressCacheOthers": null
}

```

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Журналы диагностики Azure](../azure-monitor/essentials/platform-logs-overview.md)
* [Анализ вариантов использования CDN Azure](./cdn-analyze-usage-patterns.md)
* [Журналы Azure Monitor](../azure-monitor/logs/log-query-overview.md)
* [Log Analytics REST API Reference](/rest/api/loganalytics) (Справочник по REST API Log Analytics)