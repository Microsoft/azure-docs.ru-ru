---
title: Мониторинг производительности веб-приложения Java в Linux в среде Azure | Документация Майкрософт
description: Расширенный мониторинг производительности приложений на веб-сайте Java с подключаемым модулем CollectD для Application Insights.
ms.topic: conceptual
ms.date: 03/14/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 08d8deb4c7769c4f8a202050d7b5515439d691f4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100573859"
---
# <a name="collectd-linux-performance-metrics-in-application-insights-deprecated"></a>собранные: метрики производительности Linux в Application Insights [устарело]

> [!IMPORTANT]
> Для мониторинга приложений Java **рекомендуется** использовать автоматическое инструментирование без изменения кода. Следуйте указаниям для **[Application Insights агента Java 3,0](./java-in-process-agent.md)**.

Для работы с метриками производительности Linux в [Application Insights](./app-insights-overview.md) установите инструмент [collectd](https://collectd.org/) вместе с его подключаемым модулем Application Insights. Это решение с открытым исходным кодом собирает разнообразные данные системной и сетевой статистики.

Обычно collectd используется, если вы уже [инструментировали веб-службу Java с помощью Application Insights][java]. Это средство предоставляет больше данных, помогая вам повысить производительность приложения или диагностировать неполадки. 

## <a name="get-your-instrumentation-key"></a>Получение ключа инструментирования
На [портале Microsoft Azure](https://portal.azure.com) откройте ресурс [Application Insights](./app-insights-overview.md), в котором требуется отобразить данные. (Либо [создайте новый ресурс](./create-new-resource.md).)

Скопируйте ключ инструментирования, идентифицирующий этот ресурс.

![Просмотрите все, откройте свой ресурс и затем в раскрывающемся списке основных компонентов выберите и скопируйте ключ инструментирования](./media/java-collectd/instrumentation-key-001.png)

## <a name="install-collectd-and-the-plug-in"></a>Установка collectd и подключаемого модуля
На компьютерах с сервером Unix выполните следующие действия.

1. Установите [collectd](https://collectd.org/) 5.4.0 или более поздней версии.
2. Загрузите [подключаемый модуль записи Application Insights collectd](https://github.com/microsoft/ApplicationInsights-Java/tree/master/core/src/main/java/com/microsoft/applicationinsights/internal). Запишите номер версии.
3. Скопируйте подключаемый модуль JAR в `/usr/share/collectd/java`.
4. Отредактируйте файл `/etc/collectd/collectd.conf`:
   * Убедитесь, что [подключаемый модуль Java](https://collectd.org/wiki/index.php/Plugin:Java) включен.
   * Обновите JVMArg для java.class.path, включив в него указанный ниже файл JAR. Номер версии должен совпадать с номером версии, которую вы загрузили:
   * `/usr/share/collectd/java/applicationinsights-collectd-1.0.5.jar`
   * Добавьте следующий фрагмент кода, используя ключ инструментирования из ресурса:

```XML

     LoadPlugin "com.microsoft.applicationinsights.collectd.ApplicationInsightsWriter"
     <Plugin ApplicationInsightsWriter>
        InstrumentationKey "Your key"
     </Plugin>
```

Пример файла конфигурации (фрагмент):

```XML

    ...
    # collectd plugins
    LoadPlugin cpu
    LoadPlugin disk
    LoadPlugin load
    ...

    # Enable Java Plugin
    LoadPlugin "java"

    # Configure Java Plugin
    <Plugin "java">
      JVMArg "-verbose:jni"
      JVMArg "-Djava.class.path=/usr/share/collectd/java/applicationinsights-collectd-1.0.5.jar:/usr/share/collectd/java/collectd-api.jar"

      # Enabling Application Insights plugin
      LoadPlugin "com.microsoft.applicationinsights.collectd.ApplicationInsightsWriter"

      # Configuring Application Insights plugin
      <Plugin ApplicationInsightsWriter>
        InstrumentationKey "12345678-1234-1234-1234-123456781234"
      </Plugin>

      # Other plugin configurations ...
      ...
    </Plugin>
    ...
```

Настройте другие [подключаемые модули collectd](https://collectd.org/wiki/index.php/Table_of_Plugins), которые могут собирать разные данные из разных источников.

Перезапустите collectd в соответствии с его [документацией](https://collectd.org/wiki/index.php/First_steps).

## <a name="view-the-data-in-application-insights"></a>Просмотр данных в Application Insights
В ресурсе Application Insights откройте раздел [метрики и добавьте диаграммы][metrics], выбрав метрики, которые нужно просмотреть из пользовательской категории.

По умолчанию метрики со всех хост-компьютеров, которые их поставляют, объединяются. Чтобы просмотреть метрики по хостам, в колонке сведений о диаграмме включите группировку и выберите группировку по параметру CollectD-Host.

## <a name="to-exclude-upload-of-specific-statistics"></a>Исключение загрузки определенных статистических данных
По умолчанию подключаемый модуль Application Insights отправляет все данные, собранные всеми подключаемыми модулями read collectd. 

Чтобы исключить данные из определенных подключаемых модулей или источников данных, выполните указанные ниже действия.

* Измените файл конфигурации. 
* В `<Plugin ApplicationInsightsWriter>`добавьте строки директив следующего вида:

| Директива | Действие |
| --- | --- |
| `Exclude disk` |Исключаются все данные, собранные подключаемым модулем `disk` |
| `Exclude disk:read,write` |Исключаются источники `read` и `write` из подключаемого модуля `disk` |

Каждая директива должна начинаться с новой строки.

## <a name="problems"></a>Проблемы?
*Данные не отображаются в портале*

* Откройте [Поиск][diagnostic] и проверьте поступление необработанных событий. Иногда они появляются в обозревателе метрик не сразу.
* Вам может понадобиться [указать исключения брандмауэра для исходящих данных](./ip-addresses.md)
* Включите трассировку в подключаемом модуле Application Insights. Добавьте в `<Plugin ApplicationInsightsWriter>`следующую строку:
  * `SDKLogger true`
* Откройте терминал и запустите collectd в режиме подробного протоколирования, чтобы проверить, не сообщает ли он о каких-либо неполадках:
  * `sudo collectd -f`

## <a name="known-issue"></a>Известная проблема

Подключаемый модуль записи для Application Insights несовместим с некоторыми подключаемыми модулями чтения. Некоторые подключаемые модули иногда отправляют сообщение "не число", когда подключаемый модуль Application Insights ожидает число с плавающей запятой.

Симптом. в собранном журнале отображаются ошибки, включающие "AI:... SyntaxError: непредвиденная лексема N ".

Обходное решение. Исключите данные, собранные подключаемым модулем записи, с которым связана проблема. 

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#track-exception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[eclipse]: app-insights-java-eclipse.md
[java]: java-get-started.md
[javalogs]: java-trace-logs.md
[metrics]: ../essentials/metrics-charts.md

