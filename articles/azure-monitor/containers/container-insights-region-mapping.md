---
title: Сопоставления регионов контейнера аналитики
description: Описывает сопоставления регионов, поддерживаемые в контейнере аналитика, Log Analytics рабочей области и пользовательских метриках.
ms.topic: conceptual
ms.date: 09/22/2020
ms.custom: references_regions
ms.openlocfilehash: f9e910b1352109608becb82609e85e26d27d2cd1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101728881"
---
# <a name="region-mappings-supported-by-container-insights"></a>Сопоставления регионов, поддерживаемые контейнером аналитики

 При включении аналитики контейнеров поддерживаются только определенные регионы для связывания рабочей области Log Analytics и кластера AKS и сбора пользовательских метрик, отправленных в Azure Monitor.

## <a name="log-analytics-workspace-supported-mappings"></a>Поддерживаемые сопоставления рабочей области Log Analytics

Поддерживаемые регионы AKS перечислены в списке [продукты, доступные по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service). Рабочая область Log Analytics должна находиться в том же регионе, за исключением регионов, перечисленных в следующей таблице. Просмотрите [заметки о выпуске AKS](https://github.com/Azure/AKS/releases) для обновлений.


|**Регион кластера AKS** | **Регион Log Analytics рабочей области** |
|-----------------------|------------------------------------|
|**Африка** | |
|саусафриканорс |WestEurope |
|саусафрикавест |WestEurope |
|**Австралия** | |
|AustraliaCentral2 |аустралиацентрал |
|**Бразилия** | |
|бразилсаус | SouthCentralUS |
|**Канада** ||
|CanadaEast |CanadaCentral |
|**Европа** | |
|францесаус |францецентрал |
|уквест |UKSouth |
|**Индия** | |
|SouthIndia |CentralIndia |
|WestIndia |CentralIndia |
|**Япония** | |
|жапанвест |JapanEast |
|**Корея** | |
|KoreaSouth |KoreaCentral |
|**США** | |
|WestCentralUS<sup>1</sup>|EastUS <sup>1</sup>|


<sup>1</sup> из-за емкости ограничений регион недоступен при создании новых ресурсов. Это относится к рабочей области Log Analytics. Однако уже существующие связанные ресурсы в регионе должны продолжать работать.

## <a name="custom-metrics-supported-regions"></a>Регионы, поддерживаемые пользовательскими метриками

Сбор метрик из узлов кластеров Azure Kubernetes Services (AKS) и модулей Pod поддерживается для публикации в качестве пользовательских метрик только в следующих [регионах Azure](../essentials/metrics-custom-overview.md#supported-regions).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы начать мониторинг кластера AKS, ознакомьтесь со [сведениями о том, как включить аналитику контейнеров](container-insights-onboard.md) , чтобы понять требования и доступные методы для включения мониторинга.  
