---
title: Партнерские решения по мониторингу Azure Service Fabric
description: Узнайте, как отслеживать приложения Azure Service Fabric, кластеры и инфраструктуру с помощью решений для мониторинга партнеров.
author: srrengar
ms.topic: article
ms.date: 10/16/2018
ms.author: srrengar
ms.openlocfilehash: fd24d65ebdf4e458870819286024d1ea2e13d83e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75645724"
---
# <a name="azure-service-fabric-monitoring-partners"></a>Партнерские решения по мониторингу Azure Service Fabric

В этой статье рассказывается о возможностях мониторинга приложений, кластеров и инфраструктуры Service Fabric с помощью удобных партнерских решений. С каждым из указанных ниже партнеров мы сотрудничали для создания интегрированных предложений для Service Fabric.

## <a name="dynatrace"></a>Dynatrace

Интеграция с Dynatrace обеспечивает множество готовых возможностей для мониторинга кластеров Service Fabric. Установка агента Dynatrace OneAgent на экземпляры VMSS позволяет использовать счетчики производительности и топологию развертывания Service Fabric на уровне приложения. Dynatrace также является отличным решением для локального мониторинга. Дополнительные сведения о функциях, перечисленных в [объявлении](https://www.dynatrace.com/news/blog/automatic-end-to-end-service-fabric-monitoring-with-dynatrace/) и [инструкциях](https://www.dynatrace.com/news/blog/automatic-end-to-end-service-fabric-monitoring-with-dynatrace/) по включению dynaTrace в кластере, см. в этой статье. 

## <a name="datadog"></a>Datadog

Datadog предлагает расширение для VMSS для экземпляров Windows и Linux. С помощью Datadog можно собирать данные журналов событий Windows, включая события платформы Service Fabric в Windows. Ознакомьтесь с инструкциями по отправке данных диагностики в Datadog [здесь](https://www.datadoghq.com/blog/azure-monitoring-enhancements/#integrate-with-azure-service-fabric).

## <a name="appdynamics"></a>AppDynamics

Интеграция Service Fabric с AppDynamics осуществляется на уровне приложения. Изменяя переменные среды и используя пакеты NuGet AppDynamics, вы можете отправлять данные телеметрии приложений в AppDynamics. См. [инструкции](https://docs.appdynamics.com/display/AZURE/Install+AppDynamics+for+Azure+Service+Fabric) по интеграции приложений .NET Service Fabric с AppDynamics.

## <a name="new-relic"></a>New Relic

New Relic — это средство управления производительностью приложений, которое хорошо интегрируется с приложениями Service Fabric. Вы можете установить пакеты NuGet New Relic и добавить определенные переменные среды в файлы манифестов для отправки данных телеметрии приложений в New Relic. Ознакомьтесь с этими [инструкциями](https://docs.newrelic.com/docs/agents/net-agent/azure-installation/install-net-agent-azure-service-fabric) по включению телеметрии New Relic для приложений .NET Service Fabric.

## <a name="elk"></a>ELK 

Стек ELK — это коллекция технологий с открытым исходным кодом: Elasticsearch, Logstash и Kibana. Используя эти технологии в сочетании, можно получать, хранить и анализировать Service Fabric данных мониторинга и диагностики. [Здесь](service-fabric-tutorial-java-elk.md) есть учебник для собственных приложений Java Service Fabric. 

## <a name="humio"></a>хумио

Хумио — это служба сбора журналов, которая может собирать журналы из приложений и событий из Service Fabric в облаке или локально в режиме реального времени. В дополнение к интерактивному наблюдению Хумио предлагает современные средства анализа и визуализации для просмотра и сбора информации из вашей системы диагностики. Хумио имеет Экономичные тарифные планы и создается для масштабирования, сохраняя при этом более высокую скорость. Она напрямую интегрируется с Service Fabric событиями платформы и данными телеметрии приложений. Дополнительные сведения об интеграции Хумио и Service Fabric можно получить [здесь](https://github.com/humio/service-fabric-humio).

## <a name="next-steps"></a>Дальнейшие действия

* Получите [представление о мониторинге и диагностике](service-fabric-diagnostics-overview.md) в Service Fabric.
* Узнайте, как [диагностировать типичные сценарии](service-fabric-diagnostics-common-scenarios.md) с помощью наших фирменных инструментов.
