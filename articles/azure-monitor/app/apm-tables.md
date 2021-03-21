---
title: Схема ресурсов на основе рабочей области Azure Monitor Application Insights
description: Сведения о новой структуре и схеме таблицы для Azure Monitor Application Insights ресурсов на основе рабочих областей.
ms.topic: conceptual
ms.date: 05/09/2020
ms.openlocfilehash: ef9d22cd2b45679928ee54778b2a521ea9ecab03
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100575611"
---
# <a name="workspace-based-resource-changes"></a>Изменения ресурсов на основе рабочей области

До появления [Application Insights ресурсов на основе рабочей области](create-workspace-resource.md)Application Insights данные хранились отдельно от других данных журнала в Azure Monitor. Они основаны на обозреватель данных Azure и используют один и тот же язык запросов Kusto (ККЛ). Это описано в разделе [журналы в Azure Monitor](../logs/data-platform-logs.md).

Данные ресурсов Application Insights на основе рабочих областей хранятся в Log Analytics рабочей области с другими данными мониторинга и данными приложений. Это упрощает настройку, позволяя более легко анализировать данные в нескольких решениях и использовать возможности рабочих областей.

## <a name="table-structure"></a>Структура таблицы

| Устаревшее имя таблицы | Новое имя таблицы | Описание |
|:---|:---|:---|
| availabilityResults | AppAvailabilityResults |  Сводные данные из тестов доступности.|
| бровсертимингс | AppBrowserTimings | Данные о производительности клиента, например время, затраченное на обработку входящих данных.|
| зависимости | AppDependencies | Вызовы из приложения в другие компоненты (включая внешние компоненты), записанные через TrackDependency (), например вызовы REST API, базы данных или файловой системы.  |
| customEvents | AppEvents | Пользовательские события, созданные приложением. |
| customMetrics | AppMetrics | Пользовательские метрики, созданные приложением. |
| pageViews | AppPageViews| Данные о каждом представлении веб-сайта с информацией браузера. |
| performanceCounters | AppPerformanceCounters | Измерения производительности от ресурсов вычислений, поддерживающих приложение, например счетчики производительности Windows. |
| requests | AppRequests | Запросы, полученные приложением. Например, отдельная запись запроса записывается в журнал для каждого HTTP-запроса, получаемого веб – приложением.  |
| исключения | AppSystemEvents | Исключения, вызываемые средой выполнения приложения, захватывают исключения как на стороне сервера, так и на стороне клиента (браузеров). |
| traces | AppTraces | Подробные журналы (трассировки), выдаваемые с помощью платформ кода приложения и ведения журналов, записанных с помощью TrackTrace (). |

## <a name="table-schemas"></a>Схемы таблиц

В следующих разделах показано сопоставление между классическими именами свойств и новыми именами свойств Application Insights на основе рабочей области.  Используйте эти сведения для преобразования любых запросов, использующих устаревшие таблицы.

Большинство столбцов имеют одинаковые имена с разной прописной буквой. Так как ККЛ учитывает регистр, необходимо изменить имя каждого столбца вместе с именами таблиц в существующих запросах. Столбцы с изменениями в дополнение к капитализации выделены. Вы по-прежнему можете использовать классические Application Insights запросы на панели **журналы** ресурса Application Insights, даже если это ресурс на основе рабочей области. Новые имена свойств необходимы для выполнения запросов в контексте Log Analytics рабочей области.

### <a name="appavailabilityresults"></a>AppAvailabilityResults

Устаревшая таблица: доступность

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|длительность|real|DurationMs|real|
|`id`|строка|`Id`|строка|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|Строка|
|location|строка|Расположение|строка|
|message|строка|Сообщение|строка|
|name|строка|Название|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|перформанцебуккет|строка|перформанцебуккет|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|size|real|Размер|real|
|Успешное завершение|строка|Успешно|Bool|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="appbrowsertimings"></a>AppBrowserTimings

Устаревшая таблица: Бровсертимингс

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|строка|
|name|строка|Имя|DATETIME|
|нетворкдуратион|real|нетворкдуратионмс|real|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|перформанцебуккет|строка|перформанцебуккет|строка|
|процессингдуратион|real|процессингдуратионмс|real|
|рецеиведуратион|real|рецеиведуратионмс|real|
|sdkVersion|строка|SdkVersion|строка|
|сенддуратион|real|сенддуратионмс|real|
|session_Id|строка|SessionId|строка|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|totalDuration|real|тоталдуратионмс|real|
|url|строка|Url|строка|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="appdependencies"></a>AppDependencies

Устаревшая таблица: зависимости

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|.|строка|Данные|строка|
|длительность|real|DurationMs|real|
|`id`|строка|`Id`|строка|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|Строка|
|name|строка|Название|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|перформанцебуккет|строка|перформанцебуккет|строка|
|resultCode|строка|ResultCode|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|Успешное завершение|строка|Успешно|Bool|
|target|строка|Целевой объект|строка|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|type|строка|DependencyType|строка|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="appevents"></a>AppEvents

Устаревшая таблица: customEvents

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|строка|
|name|строка|Название|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="appmetrics"></a>AppMetrics

Устаревшая таблица: customMetrics

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|iKey|строка|IKey|строка|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|строка|
|name|строка|Название|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|
|value|real|удален||
|валуекаунт|INT|валуекаунт|INT|
|валуемакс|real|валуемакс|real|
|валуемин|real|валуемин|real|
|валуестддев|real|валуестддев|real|
|валуесум|real|валуесум|real|

### <a name="apppageviews"></a>AppPageViews

Устаревшая таблица: pageViews

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|длительность|real|DurationMs|real|
|`id`|строка|`Id`|строка|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|Строка|
|name|строка|Название|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|перформанцебуккет|строка|перформанцебуккет|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|url|строка|Url|строка|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="appperformancecounters"></a>AppPerformanceCounters

Устаревшая таблица: performanceCounters

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|категория|строка|Category|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|Счетчик|строка|удален||
|customDimensions|Динамический|Свойства|Динамический|
|iKey|строка|IKey|строка|
|экземпляр|строка|Экземпляр|строка|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|строка|
|name|строка|Название|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|
|value|real|Значение|real|

### <a name="apprequests"></a>AppRequests

Устаревшая таблица: запросы

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|длительность|real|DurationMs|Real|
|`id`|строка|`Id`|Строка|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|Строка|
|name|строка|name|Строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|перформанцебуккет|строка|перформанцебуккет|Строка|
|resultCode|строка|ResultCode|Строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|source|строка|Источник|Строка|
|Успешное завершение|строка|Успешно|Bool|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|url|строка|Url|Строка|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="appsystemevents"></a>AppSystemEvents

Устаревшая таблица: исключения

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|сборка|строка|Сборка|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|подробности|Динамический|Сведения|Динамический|
|handledAt|строка|хандледат|строка|
|iKey|строка|IKey|строка|
|иннермостассембли|строка|иннермостассембли|строка|
|иннермостмессаже|строка|иннермостмессаже|строка|
|иннермостмесод|строка|иннермостмесод|строка|
|иннермосттипе|строка|иннермосттипе|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|строка|
|message|строка|Сообщение|строка|
|method|строка|Метод|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|аутерассембли|строка|аутерассембли|строка|
|аутермессаже|строка|аутермессаже|строка|
|аутермесод|строка|аутермесод|строка|
|аутертипе|строка|аутертипе|строка|
|проблемид|строка|проблемид|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|северитилевел|INT|SeverityLevel|INT|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|type|строка|ExceptionType|строка|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

### <a name="apptraces"></a>AppTraces

Устаревшая таблица: трассировки

|ApplicationInsights|Type|LogAnalytics|Type|
|:---|:---|:---|:---|
|appId|строка|\_ResourceGuid|строка|
|application_Version|строка|AppVersion|строка|
|appName|строка|\_ResourceId|строка|
|client_Browser|строка|ClientBrowser|строка|
|client_City|строка|клиентЦити|строка|
|client_CountryOrRegion|строка|клиенткаунтрйоррегион|строка|
|client_IP|строка|ClientIP|строка|
|client_Model|строка|клиентмодел|строка|
|client_OS|строка|клиентос|строка|
|client_StateOrProvince|строка|клиентстатеорпровинце|строка|
|client_Type|строка|клиенттипе|строка|
|cloud_RoleInstance|строка|аппролеинстанце|строка|
|cloud_RoleName|строка|аппроленаме|строка|
|customDimensions|Динамический|Свойства|Динамический|
|кустоммеасурементс|Динамический|Измерения|Динамический|
|iKey|строка|IKey|строка|
|itemCount|INT|ItemCount|INT|
|itemId|строка|\_ItemId|строка|
|itemType|строка|Тип|строка|
|message|строка|Сообщение|строка|
|operation_Id|строка|Операции|строка|
|operation_Name|строка|OperationName|строка|
|operation_ParentId|строка|оператионпарентид|строка|
|operation_SyntheticSource|строка|оператионсинсетиксаурце|строка|
|sdkVersion|строка|SdkVersion|строка|
|session_Id|строка|SessionId|строка|
|северитилевел|INT|SeverityLevel|INT|
|TIMESTAMP|DATETIME|TimeGenerated|DATETIME|
|user_AccountId|строка|UserAccountId|строка|
|user_AuthenticatedId|строка|усераусентикатедид|строка|
|user_Id|строка|UserId|строка|

## <a name="next-steps"></a>Дальнейшие действия

* [Изучение метрик](../essentials/metrics-charts.md)
* [Написание запросов аналитики](../logs/log-query-overview.md)

