---
title: включить файл
description: включить файл
services: application-insights
author: lgayhardt
ms.service: application-insights
ms.topic: include
ms.date: 08/06/2019
ms.author: lagayhar
ms.custom: include file
ms.openlocfilehash: 1c4f6b876a4aa80c7e51f2bb3ca88234203d0daa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101726189"
---
Число метрик и событий, используемых в приложении (то есть на ключ инструментирования), ограничено. Ограничения зависят от выбранного [ценового плана](https://azure.microsoft.com/pricing/details/application-insights/).

| Ресурс | Ограничение по умолчанию | Примечание
| --- | --- | --- |
| Общий объем данных в день | 100 ГБ | Объем данных можно сократить, задав ограничение. Если требуется больше данных, на портале можно увеличить граничное значение до 1000 ГБ. Если требуется объем более 1000 ГБ, отправьте сообщение электронной почты на адрес AIDataCap@microsoft.com.
| Регулирование | 32 000 событий в секунду | Ограничение измеряется каждую минуту.
| Журналы хранения данных | [30–730 дней](../articles/azure-monitor/app/pricing.md#change-the-data-retention-period)  | Этот ресурс предназначен для [журналов](../articles/azure-monitor/logs/log-query-overview.md).
| Метрики хранения данных | 90 дней| Этот ресурс предназначен для [обозревателя метрик](../articles/azure-monitor/essentials/metrics-charts.md).
| Хранение подробных результатов [многошагового теста доступности](../articles/azure-monitor/app/availability-multistep.md) | 90 дней | Этот ресурс предоставляет подробные результаты каждого шага.
| Максимальный размер элемента телеметрии | 64 КБ |
| Максимальное количество элементов телеметрии на пакет | 64 000 |
| Длина имен свойств и метрик | 150 | См. [схемы типов](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/EndpointSpecs/Schemas/Bond).
| Длина строки значения свойства | 8192  | См. [схемы типов](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/EndpointSpecs/Schemas/Bond).
| Длина сообщения трассировки и исключения | 32,768  | См. [схемы типов](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/EndpointSpecs/Schemas/Bond).
| Количество [тестов доступности](../articles/azure-monitor/app/monitor-web-app-availability.md) для одного приложения | 100 |
| Хранение данных [профилировщика](../articles/azure-monitor/app/profiler.md) | 5 дней |
| Отправляемые данные [профилировщика](../articles/azure-monitor/app/profiler.md) в день | 10 ГБ |

Дополнительные сведения см. в статье [Управление ценами и квотами для Application Insights](../articles/azure-monitor/app/pricing.md).