---
title: Краткое руководство. анализ веб-приложений Java с помощью Azure Application Insights
description: 'Сведения о мониторинге производительности веб-приложений Java с помощью Application Insights. '
ms.topic: conceptual
ms.date: 11/22/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: cdad5a0ec158f216a63ae7e1e2abc10d2841b3bc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100593736"
---
# <a name="quickstart-get-started-with-application-insights-in-a-java-web-project"></a>Краткое руководство. Начало работы с Application Insights в веб-проекте Java


> [!CAUTION]
> По состоянию на ноябрь 2020 для мониторинга приложений Java рекомендуется автоматическое инструментирование с помощью Azure Monitor Application Insights агента Java 3,0. Дополнительные сведения о том, как приступить к работе, см. в разделе [Application Insights агент Java 3,0](./java-in-process-agent.md).

В этом кратком руководстве вы используете пакет SDK для Application Insights для инструментирования запросов, мониторинга зависимостей и сбора счетчиков производительности, диагностики проблем производительности и исключений, а также для написания кода, который позволяет отслеживать действия пользователей с приложением.

Application Insights — это расширяемая служба аналитики для разработчиков веб-ресурсов, позволяющая оценивать производительность и использование работающего приложения. Надстройка Application Insights поддерживает приложения Java, работающие под управлением Linux, Unix или Windows.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
* Работающее приложение Java.

## <a name="get-an-application-insights-instrumentation-key"></a>Получение ключа инструментирования Application Insights

> [!IMPORTANT]
> Новые регионы Azure **должны** использовать строки подключения вместо ключей инструментирования. [Строка подключения](./sdk-connection-string.md?tabs=java) определяет ресурс, с которым необходимо связать данные телеметрии. Она также позволяет изменить конечные точки, которые ресурс будет использовать в качестве назначения для данных телеметрии. Необходимо скопировать строку подключения и добавить ее в код приложения или переменную среды.
1. Войдите на [портал Azure](https://portal.azure.com/).
2. На портале Azureсоздайте ресурс Application Insights. Задайте тип приложения: веб-приложение Java.

3. Найдите ключ инструментирования нового ресурса. Далее будет необходимо вставить его в проект кода.

    ![В обзоре нового ресурса щелкните "Свойства" и скопируйте ключ инструментирования](./media/java-get-started/instrumentation-key-001.png)

## <a name="add-the-application-insights-sdk-for-java-to-your-project"></a>Добавление в проект пакета SDK Application Insights для Java

*Выберите тип проекта.*

# <a name="maven"></a>[Maven](#tab/maven)

Если проект уже настроен на использование Maven для сборки, объедините приведенный ниже код в файл *pom.xml* .

Затем обновите зависимости проекта, чтобы скачать двоичные файлы.

```XML
    <dependencies>
      <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-web-auto</artifactId>
        <!-- or applicationinsights-web for manual web filter registration -->
        <!-- or applicationinsights-core for bare API -->
        <version>2.6.2</version>
      </dependency>
    </dependencies>
```

# <a name="gradle"></a>[Gradle](#tab/gradle)

Если проект уже настроен на использование Gradle для сборки, объедините приведенный ниже код в файл *Build. Gradle* .

Затем обновите зависимости проекта, чтобы скачать двоичные файлы.

```gradle
    dependencies {
      compile group: 'com.microsoft.azure', name: 'applicationinsights-web-auto', version: '2.6.2'
      // or applicationinsights-web for manual web filter registration
      // or applicationinsights-core for bare API
    }
```

---

### <a name="questions"></a>Вопросы
* *Какова связь между `-web-auto` `-web` `-core` компонентами и?*
  * `applicationinsights-web-auto` предоставляет метрики, которые отсчитываются количество запросов HTTP сервлета и время отклика путем автоматической регистрации фильтра Application Insights сервлета в среде выполнения.
  * `applicationinsights-web` также предоставляет метрики, которые отсчитываются количество запросов HTTP сервлета и время ответа, но требуют ручной регистрации фильтра Application Insights сервлета в приложении.
  * `applicationinsights-core` предоставляет только простой API, например, если приложение не основано на сервлета.
  
* *Как обновить пакет SDK до последней версии?*
  * По состоянию на ноябрь 2020 для мониторинга приложений Java рекомендуется автоматическое инструментирование с помощью Azure Monitor Application Insights агента Java 3,0. Дополнительные сведения о том, как приступить к работе, см. в разделе [Application Insights агент Java 3,0](./java-in-process-agent.md).

## <a name="add-an-applicationinsightsxml-file"></a>Добавление файла *ApplicationInsights.xml*
Добавьте *ApplicationInsights.xml* в папку ресурсов проекта или убедитесь, что она добавлена в путь к классу развертывания проекта. Скопируйте в него следующий код XML.

Замените ключ инструментирования на тот, который был получен из портал Azure.

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">

   <!-- The key from the portal: -->
   <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>

   <!-- HTTP request component (not required for bare API) -->
   <TelemetryModules>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
   </TelemetryModules>

   <!-- Events correlation (not required for bare API) -->
   <!-- These initializers add context data to each event -->
   <TelemetryInitializers>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
   </TelemetryInitializers>

</ApplicationInsights>
```

При необходимости файл конфигурации может находиться в любом расположении, доступном для приложения.  Свойство System `-Dapplicationinsights.configurationDirectory` указывает каталог, содержащий *ApplicationInsights.xml*. Например, файл конфигурации, расположенный в каталоге `E:\myconfigs\appinsights\ApplicationInsights.xml`, будет настроен со свойством `-Dapplicationinsights.configurationDirectory="E:\myconfigs\appinsights"`.

* Ключ инструментирования пересылается вместе с каждым элементом телеметрии; служба Application Insights отобразит его в ресурсе.
* Компонент HTTP-запросов является необязательным. Он автоматически передает на портал телеметрию о запросах и значения времени ответа.
* Корреляция события является дополнением к компоненту HTTP-запросов. Он назначает идентификатор каждому запросу, полученному сервером. Затем этот идентификатор добавляется в качестве свойства для каждого элемента телеметрии в качестве свойства "Operation.Id". Благодаря этому можно выделить данные телеметрии, связанные с каждым из запросов, путем установки фильтра [Поиск по журналу диагностики][diagnostic].

### <a name="alternative-ways-to-set-the-instrumentation-key"></a>Другие способы задать ключ инструментирования
Пакет SDK Application Insights ищет ключ в следующем порядке:

1. Системное свойство:-DAPPINSIGHTS_INSTRUMENTATIONKEY = your_ikey
2. Переменная среды: APPINSIGHTS_INSTRUMENTATIONKEY
3. Файл конфигурации: *ApplicationInsights.xml*

Вы также можете [задать его в коде](./api-custom-events-metrics.md#ikey):

```java
    String instrumentationKey = "00000000-0000-0000-0000-000000000000";

    if (instrumentationKey != null)
    {
        TelemetryConfiguration.getActive().setInstrumentationKey(instrumentationKey);
    }
```

## <a name="add-agent"></a>Добавить агент

[Установите агент Java](java-agent.md) для отслеживания исходящих вызовов HTTP, запросов JDBC, ведения журнала приложений и лучшего именования операций.

## <a name="run-your-application"></a>Запуск приложения
Запустите приложение в режиме отладки на компьютере разработки или опубликуйте его на своем сервере.

## <a name="view-your-telemetry-in-application-insights"></a>Просмотр данных телеметрии в Application Insights
Вернитесь к ресурсу Application Insights на [портале Microsoft Azure](https://portal.azure.com).

В колонке обзора появятся данные HTTP-запросов. (Если данные отсутствуют, подождите несколько секунд и нажмите кнопку обновления).

![Снимок экрана: обзор с примером данных](./media/java-get-started/overview-graphs.png)

[Дополнительные сведения о метриках.][metrics]

Щелкните любую диаграмму, чтобы увидеть более подробные агрегированные метрики.

![Панель Application Insights сбоев с диаграммами](./media/java-get-started/006-barcharts.png)

### <a name="instance-data"></a>Данные экземпляров
Щелкните тип запроса, чтобы просмотреть отдельные экземпляры.

![Детализация конкретного примера представления](./media/java-get-started/007-instance.png)

### <a name="analytics-powerful-query-language"></a>Аналитика: мощный язык запросов
По мере увеличения объема накопленных данных вы сможете использовать запросы для объедения данных и поиска отдельных экземпляров.  [Аналитика](../logs/log-query-overview.md) — это мощный инструмент, который не только позволяет изучать сведения о производительности и использовании, но и диагностировать возможные неполадки.

![Пример аналитики](./media/java-get-started/0025.png)

## <a name="install-your-app-on-the-server"></a>Установка приложения на сервере
Теперь опубликуйте приложение на сервере, откройте доступ для пользователей и изучайте телеметрию на портале.

* Убедитесь, что брандмауэр позволяет приложению отправлять телеметрию на следующие порты:

  * dc.services.visualstudio.com:443
  * f5.services.visualstudio.com:443

* Если исходящий трафик должен проходить через брандмауэр, определите системные свойства `http.proxyHost` и `http.proxyPort`.

* На серверах Windows необходимо установить следующее:

  * [распространяемые компоненты Microsoft Visual C++.](https://www.microsoft.com/download/details.aspx?id=40784)

    (Сюда входят счетчики производительности).

## <a name="azure-app-service-aks-vms-config"></a>Служба приложений Azure, AKS, Конфигурация виртуальных машин

Лучший и самый простой подход к мониторингу приложений, работающих на любом из поставщиков ресурсов Azure, — использовать Application Insights автоматическое инструментирование с помощью [агента Java 3,0](./java-in-process-agent.md).


## <a name="exceptions-and-request-failures"></a>Исключения и ошибки запросов
Необработанные исключения и ошибки запросов автоматически собираются Application Insights веб-фильтром.

Чтобы получить данные о других исключениях, можно [Вставить в код вызовы trackException ()][apiexceptions].

## <a name="monitor-method-calls-and-external-dependencies"></a>Мониторинг вызовов методов и внешних зависимостей.
[установить агент для Java](java-agent.md) .

И для автоматического именования операций.

## <a name="w3c-distributed-tracing"></a>Распределенная трассировка W3C

Пакет средств разработки для Java Application Insights теперь поддерживает [распределенную трассировку W3C](https://w3c.github.io/trace-context/).

Конфигурация входящих параметров пакета средств разработки описана далее в статье о [корреляции](correlation.md).

Конфигурация входящих параметров пакета средств разработки задана в файле [AI-Agent.xml](java-agent.md).

## <a name="performance-counters"></a>Счетчики производительности
Откройте раздел **исследование**, **метрики**, чтобы просмотреть диапазон счетчиков производительности.

![Снимок экрана: панель метрик с выбранным байтовым частным байтом процесса](./media/java-get-started/011-perf-counters.png)

### <a name="customize-performance-counter-collection"></a>Настройка сбора данных счетчиками производительности
Чтобы отключить сбор данных о стандартном наборе счетчиков производительности, добавьте следующий код в корневой узел файла *ApplicationInsights.xml* :

```XML
    <PerformanceCounters>
       <UseBuiltIn>False</UseBuiltIn>
    </PerformanceCounters>
```

### <a name="collect-additional-performance-counters"></a>Сбор данных дополнительными счетчиками производительности
Можно задать необходимость сбора данных дополнительными счетчиками производительности.

#### <a name="jmx-counters-exposed-by-the-java-virtual-machine"></a>Счетчики JMX (предоставляются виртуальной машиной Java)

```XML
    <PerformanceCounters>
      <Jmx>
        <Add objectName="java.lang:type=ClassLoading" attribute="TotalLoadedClassCount" displayName="Loaded Class Count"/>
        <Add objectName="java.lang:type=Memory" attribute="HeapMemoryUsage.used" displayName="Heap Memory Usage-used" type="composite"/>
      </Jmx>
    </PerformanceCounters>
```

* `displayName` — имя, отображаемое в портале Application Insights.
* `objectName` — имя объекта JMX.
* `attribute` — атрибут имени объекта JMX для выборки
* `type` (необязательно) — тип атрибута объекта JMX:
  * по умолчанию: простой тип, такой как int или long;
  * `composite`— данные счетчика производительности имеют формат "Атрибут.Данные";
  * `tabular`— данные счетчика производительности имеют формат строки таблицы.

#### <a name="windows-performance-counters"></a>Счетчики производительности Windows
Каждый [счетчик производительности Windows](/windows/win32/perfctrs/performance-counters-portal) входит в состав категории (аналогично поле является членом класса). Категория может быть глобальной либо иметь пронумерованные или именованные экземпляры.

```XML
    <PerformanceCounters>
      <Windows>
        <Add displayName="Process User Time" categoryName="Process" counterName="%User Time" instanceName="__SELF__" />
        <Add displayName="Bytes Printed per Second" categoryName="Print Queue" counterName="Bytes Printed/sec" instanceName="Fax" />
      </Windows>
    </PerformanceCounters>
```

* displayName — имя, отображаемое в портале Application Insights.
* categoryName — категория счетчика производительности (объект производительности), с которой связан этот счетчик производительности.
* counterName — имя счетчика производительности.
* instanceName — имя экземпляра категории счетчика производительности или пустая строка (""), если категория содержит единственный экземпляр. Если categoryName имеет значение "Process", и источником данных для счетчиков производительности является текущий процесс виртуальной машины Java, на которой выполняется ваше приложение, укажите `"__SELF__"`.

### <a name="unix-performance-counters"></a>Счетчики производительности Unix
* [установите collectd с подключаемым модулем Application Insights](java-collectd.md) .

## <a name="get-user-and-session-data"></a>Получение данных о пользователях и сеансах
Итак, вы отправляете телеметрию с веб-сервера. Теперь для получения полного представления о приложении можно настроить дополнительные функции мониторинга:

* [Добавьте телеметрии на веб-страницы][usage] для мониторинга просмотров страниц и метрик пользователя.
* [Настройте веб-тесты][availability], которые помогут быть уверенными в том, что приложение остается работоспособным и правильно отвечает на запросы.

## <a name="send-your-own-telemetry"></a>Отправка собственных данных телеметрии
После установки пакета SDK можно использовать интерфейс API для отправки собственных данных телеметрии.

* [Отслеживайте пользовательские события и метрики][api], чтобы знать, какие операции выполняют пользователи в приложении.
* [Выполняйте поиск событий и журналов][diagnostic] для диагностики неполадок.

## <a name="availability-web-tests"></a>Доступность веб-тестов
Application Insights может тестировать ваш веб-сайт через равные промежутки времени для проверки, работает ли он и правильно ли отвечает на запросы.

[Дополнительные сведения о настройке веб-тестов доступности см. здесь.][availability]

## <a name="questions-problems"></a>У вас появились вопросы? Проблемы?
[Устранение неполадок Java](java-troubleshoot.md)

## <a name="next-steps"></a>Дальнейшие действия
* [Отслеживайте вызовы зависимостей.](java-agent.md)
* [Отслеживайте счетчики производительности Unix.](java-collectd.md)
* Добавляйте [мониторинг на веб-страницы](javascript.md), чтобы отслеживать время загрузки страниц, вызовы AJAX и исключения браузера.
* Пишите [пользовательскую телеметрию](./api-custom-events-metrics.md) для отслеживания использования в браузере или на сервере.
* Использование  [аналитики](../logs/log-query-overview.md) для мощных запросов телеметрии из приложения
* Дополнительные сведения см. в разделе [Azure for Java developers](/java/azure) (Azure для разработчиков Java).

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#trackexception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[javalogs]: java-trace-logs.md
[metrics]: ../essentials/metrics-charts.md
[usage]: javascript.md