---
title: Запрос данных из среды 2-го поколения с помощью кода C# — Аналитика временных рядов Azure | Документация Майкрософт
description: Узнайте, как запрашивать данные из среды службы Аналитики временных рядов Azure 2-го поколения с помощью приложения, написанного на C#.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 10/02/2020
ms.custom: seodec18
ms.openlocfilehash: aecd18fd0d568904f9704b749525204ced05f3ef
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103463431"
---
# <a name="query-data-from-the-azure-time-series-insights-gen2-environment-using-c-sharp"></a>Запрос данных из среды Аналитики временных рядов Azure 2-го поколения с помощью C#

В этом примере C# показано, как запросить данные из [API доступа к данным 2-го поколения](/rest/api/time-series-insights/reference-data-access-overview) в средах службы Аналитики временных рядов Azure 2-го поколения.

> [!TIP]
> Ознакомьтесь с примерами общедоступного кода C# 2-го поколения по адресу [https://github.com/Azure-Samples/Azure-Time-Series-Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen2-sample/csharp-tsi-gen2-sample).

## <a name="summary"></a>Сводка

Приведенный ниже пример кода демонстрирует следующие возможности.

* Поддержка автоматического создания пакетов SDK из [Azure AutoRest](https://github.com/Azure/AutoRest).
* Получение маркера доступа с помощью Azure Active Directory с использованием [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).
* Передача полученного маркера доступа в заголовок `Authorization` последующих запросов API доступа к данным.
* В примере показан интерфейс консоли, демонстрирующий, как выполняются HTTP-запросы к следующим элементам:
  * [API сред 2-го поколения](/rest/api/time-series-insights/reference-environments-apis)
    * [API получения сведений о доступности сред](/rest/api/time-series-insights/dataaccessgen2/query/getavailability) и [API получения схемы событий](/rest/api/time-series-insights/dataaccessgen2/query/geteventschema)
  * [API запросов 2-го поколения](/rest/api/time-series-insights/reference-query-apis)
    * [API получения событий](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents), [API получения последовательности](/rest/api/time-series-insights/dataaccessgen2/query/execute#getseries) и [API получения последовательности статических выражений](/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)
  * [API модели временных рядов](/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)
    * [API получения иерархий](/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies) и [API пакета иерархий](/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/executebatch)
    * [API получения типов](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes) и [API пакета типов](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch)
    * [API получения экземпляров](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances) и [API пакета экземпляров](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/executebatch)

* Возможности расширенного [поиска](/rest/api/time-series-insights/reference-model-apis#search-features) и [TSX](/rest/api/time-series-insights/reference-time-series-expression-syntax).

## <a name="prerequisites-and-setup"></a>Предварительные требования и настройка

Перед компиляцией и запуском примера кода выполните следующие шаги.

1. [Подготовьте к работе среду Аналитики временных рядов 2-го поколения](./how-to-create-environment-using-portal.md).
1. Настройте среду службы Аналитики временных рядов Azure для Azure Active Directory, как описано в разделе [Проверка подлинности и авторизация](time-series-insights-authentication-and-authorization.md).
1. Запустите файл [GenerateCode.bat](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClient/GenerateCode.bat), как указано в [Readme.md](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClient/Readme.md), чтобы создать зависимости клиентов Аналитики временных рядов Azure 2-го поколения.
1. Откройте решение `TSIPreviewDataPlaneclient.sln` и задайте `DataPlaneClientSampleApp` в качестве проекта по умолчанию в Visual Studio.
1. Установите необходимые зависимости проекта, выполнив действия, описанные [ниже](#project-dependencies), и скомпилируйте пример в исполняемый файл `.exe`.
1. Запустите файл `.exe`, дважды щелкнув его.

## <a name="project-dependencies"></a>Зависимости проектов

Рекомендуется использовать последнюю версию Visual Studio:

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) — версия 16.4.2 или более поздняя

В примере кода имеется несколько обязательных зависимостей, которые можно просмотреть в файле [packages.config](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClientSampleApp/packages.config).

Скачайте пакеты в Visual Studio 2019, выбрав параметр **Сборка** > **Собрать решение**.

Кроме того, добавьте каждый пакет с помощью [NuGet 2.12 +](https://www.nuget.org/). Пример:

* `dotnet add package Microsoft.IdentityModel.Clients.ActiveDirectory --version 4.5.1`

## <a name="c-sample-code"></a>Пример кода C#

Чтобы получить доступ к примеру кода C#, обратитесь к репозиторию [Аналитика временных рядов Azure](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen2-sample/csharp-tsi-gen2-sample).

> [!NOTE]
>
> * Пример кода можно выполнить без изменения переменных среды по умолчанию.
> * Пример кода компилируется в исполняемое консольное приложение .NET.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о запросах см. в [справочнике по API запросов](/rest/api/time-series-insights/reference-query-apis).

* Узнайте, как [подключить приложение JavaScript с помощью клиентского пакета SDK](https://github.com/microsoft/tsiclient) к службе Аналитики временных рядов Azure.