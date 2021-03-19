---
title: Как настроить метрики JMX — Azure Monitor Application Insights для Java
description: Настройка дополнительной коллекции метрик JMX для Azure Monitor агента Java Application Insights
ms.topic: conceptual
ms.date: 03/16/2021
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 020278bf6e5a823f6b3caa22d03f4b5dd003c0d9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609614"
---
# <a name="configuring-jmx-metrics"></a>Настройка метрик JMX

Агент Java 3,0 Application Insights по умолчанию собирает некоторые метрики JMX, но во многих случаях это недостаточно. В этом документе описывается параметр конфигурации JMX в разделе Details.

## <a name="how-do-i-collect-additional-jmx-metrics"></a>Разделы справки собирайте дополнительные метрики JMX?

Коллекцию метрик JMX можно настроить, добавив раздел в ```"jmxMetrics"``` applicationinsights.jsфайла. Можно указать имя метрики так, как она должна отображаться в портал Azure в ресурсе Application Insights. Необходимо определить имя и атрибут объекта для каждой из метрик, которые необходимо собрать.

## <a name="how-do-i-know-what-metrics-are-available-to-configure"></a>Разделы справки узнать, какие метрики доступны для настройки?

Вы зафиксированы. вам необходимо иметь представление об именах объектов и атрибутах, эти свойства отличаются для различных библиотек, платформ и серверов приложений, и часто неплохо документированы. Чтобы получить имена и атрибуты объектов, необходимо просмотреть дерево MBean. MBean-компонент — это управляемый объект Java, который может представлять устройство, приложение или ресурс, а также набор атрибутов. 

Чтобы просмотреть доступные метрики и просмотреть доступные метрики, мы рекомендуем использовать [элемент управления Java миссии](https://www.oracle.com/java/technologies/jdk-mission-control.html).

### <a name="how-to-navigate-the-java-mission-control-to-get-to-the-right-metrics"></a>Как перейти к нужным метрикам с помощью элемента управления "задача Java"?

При запуске средства управления Java миссии у вас появится возможность выбора виртуальных машин Java в левой части, щелкните соответствующий процесс на вкладке "браузер ВИРТУАЛЬНОЙ машины Java". Подождите, пока JMC загрузит панель мониторинга для процесса, выберите вкладку "браузер MBean-компонента" внизу (см. ниже). JMC должен находиться в той же папке, что и ВИРТУАЛЬНОЙ машины Java, а процесс или приложение должны быть настроены и запускаться.

![Снимок экрана обозревателя JMC MBean](media/java-ipa/jmx/jmc-mbean-browser.png)

### <a name="how-to-get-to-the-metrics-i-want-and-the-necessary-attributes"></a>Как получить нужные метрики и необходимые атрибуты?

Браузер MBean открывает дерево MBean и список категорий, которые можно расширить. При выборе категории слева открывается список атрибутов справа. Ниже приведен пример метрики, имя объекта и атрибуты. Атрибуты могут быть вложенными, как показано в примере ниже.

![Снимок экрана: дерево JMC MBean](media/java-ipa/jmx/jmc-metric-sample.png)

### <a name="configuration-example"></a>Пример конфигурации

В области выбора, как показано на рисунке выше, можно настроить несколько метрик. Первый из них является примером вложенной метрики `LastGcInfo` , имеющей несколько свойств и требующих захвата `GcThreadCount` .

```json
"jmxMetrics": [
      {
        "name": "Demo - GC Thread Count",
        "objectName": "java.lang:type=GarbageCollector,name=PS MarkSweep",
        "attribute": "LastGcInfo.GcThreadCount"
      },
      {
        "name": "Demo - GC Collection Count",
        "objectName": "java.lang:type=GarbageCollector,name=PS MarkSweep",
        "attribute": "CollectionCount"
      },
      {
        "name": "Demo - Thread Count",
        "objectName": "java.lang:type=Threading",
        "attribute": "ThreadCount"
      }
],
```

### <a name="types-of-collected-metrics-and-available-configuration-options"></a>Типы собранных метрик и доступные параметры конфигурации?

Мы поддерживаем числовые и логические метрики JMX, в то время как другие типы не поддерживаются и будут игнорироваться. 

В настоящее время подстановочные знаки и статистические атрибуты не поддерживаются, поэтому каждая пара атрибута "имя объекта"/"атрибут" должна быть настроена отдельно. 


## <a name="where-do-i-find-the-jmx-metrics-in-application-insights"></a>Где найти метрики JMX в Application Insights?

По мере выполнения приложения и сбора метрик JMX их можно просмотреть, перейдя в портал Azure и перейдя к ресурсу Application Insights. На вкладке метрики выберите раскрывающийся список, как показано ниже, чтобы просмотреть метрики.

![Снимок экрана метрик на портале](media/java-ipa/jmx/jmx-portal.png)
