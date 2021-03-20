---
title: Azure Application Insights | Документация Майкрософт
description: Сопоставьте данные из Application Insights с другими наборами данных, такими как таблицы обогащения или уточняющего запроса, источники данных, отличные от Application Insights, и пользовательские данные.
ms.topic: conceptual
author: eternovsky
ms.author: evternov
ms.date: 08/08/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 933280b5d3b81098f18f22a72bd2c7f942869e6a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100578329"
---
# <a name="correlating-application-insights-data-with-custom-data-sources"></a>Корреляция данных Application Insights с пользовательскими источниками данных

Application Insights собирает несколько разных типов данных: исключения, трассировки, просмотр страниц и другие. Хотя часто это достаточно для изучения производительности, надежности и использования приложения, бывают случаи, когда полезно сопоставлять данные, хранящиеся в Application Insights, с другими полностью настраиваемыми наборами данных.

Ситуации, когда можно использовать настраиваемые данные.

- Таблицы обогащения данных или поиска. Например, дополните имя сервера именем владельца сервера и местом расположения задания, в котором оно может быть найдено. 
- Корреляция с источниками данных, не относящихся к Application Insights. Например, сопоставьте данные о покупке в интернет-магазине с информацией из службы выполненных покупок, чтобы определить, насколько точны оценки времени доставки. 
- Полностью настраиваемые данные. Большинство клиентов любят язык запросов и производительность платформы журнала Azure Monitor, которая поддерживает Application Insights, и хотят использовать ее для запроса данных, которые совсем не связаны с Application Insights. Например, для отслеживания производительности солнечных батарей как части интеллектуальной домашней установки, описанной [здесь](https://www.catapultsystems.com/blogs/using-log-analytics-and-a-special-guest-to-forecast-electricity-generation/).

## <a name="how-to-correlate-custom-data-with-application-insights-data"></a>Корреляция пользовательских данных с данными Application Insights 

Поскольку Application Insights поддерживается мощной платформой журнала Azure Monitor, для приема данных можно использовать Azure Monitor на полную силу. Затем мы будем писать запросы с помощью оператора Join, который будет сопоставлять эти пользовательские данные с данными, доступными для нас в журналах Azure Monitor. 

## <a name="ingesting-data"></a>Прием данных

В этом разделе мы рассмотрим, как получить данные в журналах Azure Monitor.

Если у вас ее еще нет, подготавливает новую рабочую область Log Analytics, следуя [этим инструкциям](../vm/quick-collect-azurevm.md) , включая шаг "Создание рабочей области".

Начните отправку данных журнала Azure Monitor. Существует несколько вариантов.

- Для синхронного механизма можно либо напрямую вызвать [API сборщика данных](../logs/data-collector-api.md) , либо использовать наш соединитель приложения логики — просто найдите "Azure log Analytics" и выберите параметр "отправить данные":

  ![Снимок экрана "Выбор и действие"](./media/custom-data-correlation/01-logic-app-connector.png)  

- Для асинхронного варианта используйте API сборщика данных для сборки конвейера обработки. Дополнительные сведения см. в [этой статье](../logs/create-pipeline-datacollector-api.md) .

## <a name="correlating-data"></a>Корреляция данных

Служба Application Insights основана на платформе журнала Azure Monitor. Поэтому можете использовать [межресурсные соединения](../logs/cross-workspace-query.md), чтобы сопоставить любые данные, которые мы приняли в Azure Monitor, с вашими данными Application Insights.

Например, можно принять данные инвентаризации и расположения лаборатории в таблицу с именем "LabLocations_CL" в Log Analytics рабочей области с именем "Мила". Если затем нужно проверить наши запросы, которые отправляются в Application Insights приложении с именем "Мяи", и сопоставить имена компьютеров, обслуживающих запросы к расположениям этих компьютеров, хранящимся в упомянутой выше пользовательской таблице, можно выполнить следующий запрос из Application Insights или контекста Azure Monitor:

```
app('myAI').requests
| join kind= leftouter (
    workspace('myLA').LabLocations_CL
    | project Computer_S, Owner_S, Lab_S
) on $left.cloud_RoleInstance == $right.Computer
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения см. по ссылке [​​API сборщика данных](../logs/data-collector-api.md).
- Дополнительные сведения о [межресурсном соединении](../logs/cross-workspace-query.md).
