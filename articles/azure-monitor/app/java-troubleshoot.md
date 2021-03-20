---
title: Устранение неполадок Application Insights в веб-проекте Java
description: Руководство по устранению неполадок — мониторинг динамических приложений Java с помощью Application Insights.
ms.topic: conceptual
ms.date: 03/14/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: df1f58418452ecafacddd013ab815ebca5cb8a35
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100097557"
---
# <a name="troubleshooting-and-q-and-a-for-application-insights-for-java-sdk"></a>Устранение неполадок и вопросы и ответы для Application Insights для пакета SDK для Java

> [!IMPORTANT]
> Для мониторинга приложений Java рекомендуется использовать автоматическое инструментирование без изменения кода. Следуйте указаниям для [Application Insights агента Java 3,0](./java-in-process-agent.md).

Возникли вопросы или проблемы при использовании [Azure Application Insights в Java][java]? Ниже приведен ряд советов.

## <a name="build-errors"></a>Ошибки сборки
**При добавлении пакета SDK для Application Insights посредством Maven или Gradle в Eclipse или Intellij Idea возникают ошибки проверки сборки или контрольной суммы.**

* Если в элементе зависимости `<version>` используется шаблон с подстановочными знаками (например, (Maven) `<version>[2.0,)</version>` или (Gradle) `version:'2.+'`), попробуйте указать конкретную версию, например `2.6.2`.

## <a name="no-data"></a>Нет данных
**Мне удалось успешно добавить Application Insights и запустить приложение, но данные не появились на портале.**

* Подождите минуту и нажмите «Обновить». Диаграммы автоматически обновляются через определенные интервалы, однако их можно обновлять и вручную. Интервал обновления зависит от временного диапазона, выбранного для диаграммы.
* Убедитесь, что ключ инструментирования указан в файле ApplicationInsights.xml (в папке ресурсов проекта) или настроен в переменной среды.
* Убедитесь в том, что в XML-файле нет узла `<DisableTelemetry>true</DisableTelemetry>`.
* В вашем брандмауэре вам может потребоваться открыть TCP-порты 80 и 443 для исходящего трафика, идущего на сайт dc.services.visualstudio.com. Ознакомьтесь с [полным списком исключений брандмауэра](./ip-addresses.md).
* На начальном экране Microsoft Azure посмотрите на карту состояния службы. Если есть какие-либо предупреждения, дождитесь возвращения всех модулей в состояние ОК, затем закройте и заново откройте модуль приложения Application Insights.
* [Включите ведение журнала](#debug-data-from-the-sdk) , добавив `<SDKLogger />` элемент в корневой узел в файле ApplicationInsights.xml (в папке ресурсов проекта) и проверьте наличие записей, начинающихся с AI: info/warn/Error для любых подозрительных журналов. 
* Убедитесь, что правильный файл ApplicationInsights.xml успешно загружен с помощью пакета SDK для Java. В случае успешной загрузки на консоли в разделе сообщений вывода отображается: «Файл конфигурации успешно найден».
* Если файл конфигурации не найден, просмотрите сообщения вывода, чтобы узнать, где происходил поиск файла конфигурации, и убедитесь, что файл ApplicationInsights.xml находится в одном из этих расположений. Как правило, файл конфигурации следует размещать вместе с JAR-файлами пакета SDK для Application Insights. Например, в Tomcat это папка WEB-INF/classes. На период разработки можно поместить ApplicationInsights.xml в папку resources веб-проекта.
* Также проверьте, нет ли на [странице проблем на GitHub](https://github.com/microsoft/ApplicationInsights-Java/issues) известных проблем с пакетом SDK.
* Убедитесь, что используются одинаковые версии ядра Application Insights и аппендеров web, агента и ведения журнала, чтобы исключить проблемы с конфликтом версий.

#### <a name="i-used-to-see-data-but-it-has-stopped"></a>Ранее видимые данные перестали отображаться
* Вы достигли месячной квоты точек данных? Чтобы узнать, откройте параметры, квоту и цены. Если это так, вы можете обновить план или заплатить за дополнительную емкость. См. [таблицу цен](https://azure.microsoft.com/pricing/details/application-insights/).
* Возможно, вы недавно обновляли SDK? Убедитесь, что в каталоге проекта присутствуют только уникальные JAR-файлы пакета SDK. Не должно существовать двух разных версий одного пакета SDK.
* Правильный ли ресурс AI вы изучаете? Проверьте параметр iKey в приложении и в ресурсе, в который вы передаете телеметрию. Эти значения должны совпадать.

#### <a name="i-dont-see-all-the-data-im-expecting"></a>Не отображаются все данные, которые ожидалось увидеть
* Откройте страницу данных об использовании и предполагаемых расходов и проверьте, не выполняется ли операция [выборки](./sampling.md). (100% передача означает, что выборка не выполняется.) Служба Application Insights может быть настроена на принятие только части телеметрии, поступающих из приложения. Это помогает не выходить за пределы месячной квоты.
* Включена ли выборка SDK? Если включена,то для выборки данных будет применяться частота, настроенная для всех применимых типов.
* Возможно, выполняется старая версия пакета SDK для Java? Начиная с версии 2.0.1 мы внедрили механизм отказоустойчивости, который обрабатывает временные ошибки в сети и на серверной части, а также сохраняет данные на локальных дисках.
* Возможно, сработало регулирование из-за слишком высокой нагрузки на телеметрию? Включив ведение журнала INFO, вы увидите сообщение "Приложение регулируется". В настоящее время действует ограничение 32 000 элементов телеметрии в секунду.

### <a name="java-agent-cannot-capture-dependency-data"></a>Агент Java не может собрать данные зависимостей
* При настройке агента Java вы соблюдали все инструкции [из этой статьи](java-agent.md)?
* Убедитесь, что JAR-файл агента и файл AI-Agent.xml размещаются в одной папке.
* Убедитесь, что поддерживается автоматический сбор для зависимости, которую вы намерены собирать. Сейчас поддерживаются только такие коллекции зависимостей: MySQL, MsSQL, Oracle DB и кэш Redis для Azure.

## <a name="no-usage-data"></a>Нет данных по использованию
**Приводятся данные по запросам и времени ответа, но нет данных по просмотру страниц, использованию браузеров и пользователям.**

Вы успешно настроили приложение для отправки данных телеметрии с сервера. Теперь вам нужно [настроить веб-страницы для отправки данных телеметрии из веб-браузера][usage].

Если клиент является приложением на [телефоне или другом устройстве][platforms], из него также можно отправлять данные телеметрии.

Для настройки телеметрии в клиенте и на сервере используйте один и тот же ключ инструментирования. Данные будут приводиться в одном и том же ресурсе Application Insights, и вы сможете сопоставлять события, полученные от клиента и с сервера.

## <a name="disabling-telemetry"></a>Отключение телеметрии
**Как отключить сбор данных телеметрии?**

В коде:

```Java

    TelemetryConfiguration config = TelemetryConfiguration.getActive();
    config.setTrackingIsDisabled(true);
```

**Or**

Измените файл ApplicationInsights.xml (в папке ресурсов проекта). Добавьте следующий элемент в корневой узел:

```XML

    <DisableTelemetry>true</DisableTelemetry>
```

При изменении значения в файле XML необходимо перезапустить приложение.

## <a name="changing-the-target"></a>Изменение целевого ресурса
**Как можно изменить ресурс Azure, в который мой проект отправляет данные?**

* [Получите ключ инструментирования нового ресурса.][java]
* Если вы добавили Application Insights в проект с помощью набора средств Azure для Eclipse, щелкните веб-проект правой кнопкой мыши, выберите **Azure**, а затем выберите **Настроить Application Insights** и измените ключ.
* Если ключ инструментирования настроен в переменной среды, внесите в эту переменную новое значение iKey.
* В противном случае измените ключ в файле ApplicationInsights.xml в папке ресурсов проекта.

## <a name="debug-data-from-the-sdk"></a>Отладка данных из пакета SDK

**Как определить, что делает пакет SDK?**

Чтобы получить дополнительные сведения о работе API, добавьте `<SDKLogger/>` в корневой узел файла конфигурации ApplicationInsights.xml.

### <a name="applicationinsightsxml"></a>ApplicationInsights.xml

Можно также указать средству ведения журнала выводить данные в файл.

```XML
  <SDKLogger type="FILE"><!-- or "CONSOLE" to print to stderr -->
    <Level>TRACE</Level>
    <UniquePrefix>AI</UniquePrefix>
    <BaseFolderPath>C:/agent/AISDK</BaseFolderPath>
</SDKLogger>
```

### <a name="spring-boot-starter"></a>Начальная загрузка весны

Чтобы включить ведение журнала SDK в приложениях с пружинной загрузкой с помощью Application Insights пружинной загрузки, добавьте в `application.properties` файл следующее:

```yaml
azure.application-insights.logger.type=file
azure.application-insights.logger.base-folder-path=C:/agent/AISDK
azure.application-insights.logger.level=trace
```

или для печати до стандартной ошибки:

```yaml
azure.application-insights.logger.type=console
azure.application-insights.logger.level=trace
```

### <a name="java-agent"></a>Агент Java

Чтобы включить ведение журнала агента ВИРТУАЛЬНОЙ машины Java, обновите [ файлAI-Agent.xml](java-agent.md):

```xml
<AgentLogger type="FILE"><!-- or "CONSOLE" to print to stderr -->
    <Level>TRACE</Level>
    <UniquePrefix>AI</UniquePrefix>
    <BaseFolderPath>C:/agent/AIAGENT</BaseFolderPath>
</AgentLogger>
```

### <a name="java-command-line-properties"></a>Свойства командной строки Java
_Начиная с версии 2.4.0_

Чтобы включить ведение журнала с помощью параметров командной строки, не изменяя файлы конфигурации, выполните следующие действия.

```
java -Dapplicationinsights.logger.file.level=trace -Dapplicationinsights.logger.file.uniquePrefix=AI -Dapplicationinsights.logger.baseFolderPath="C:/my/log/dir" -jar MyApp.jar
```

или для печати до стандартной ошибки:

```
java -Dapplicationinsights.logger.console.level=trace -jar MyApp.jar
```

## <a name="the-azure-start-screen"></a>Начальный экран Azure
**Я ищу [портал Azure](https://portal.azure.com). Есть ли на карте информация о моем приложении?**

Нет, она показывает работоспособность серверов Azure по всему миру.

*Как найти данные о приложении на начальной доске (начальном экране) Azure?*

Если вы [настроили приложение для использования Application Insights][java], нажмите кнопку "Обзор", выберите Application Insights, а затем выберите ресурс приложения, который вы создали для приложения. Чтобы упростить эту операцию в будущем, можно закрепить приложение на начальном экране.

## <a name="intranet-servers"></a>Серверы в интрасети
**Можно ли наблюдать за сервером в интрасети?**

Да, если ваш сервер может отправлять данные телеметрии на портал Application Insights через общедоступную сеть Интернет.

Возможно, потребуется [открыть некоторые исходящие порты в брандмауэре сервера](./ip-addresses.md#outgoing-ports) , чтобы пакет SDK мог отправить данные на портал.

## <a name="data-retention"></a>Хранение данных
**Как долго данные хранятся на портале? Защищен ли он?**

Ознакомьтесь с разделом [Хранение данных и конфиденциальность][data].

## <a name="debug-logging"></a>Ведение журнала отладки
Application Insights использует модуль `org.apache.http`. Он перемещен в JAR-файлы ядра Application Insights в пространство имен `com.microsoft.applicationinsights.core.dependencies.http`. Это позволяет Application Insights обрабатывать ситуации, в которых разные версии `org.apache.http` существуют в одной базе кода.

>[!NOTE]
>Если включить уровень ведения журнала "Отладка" для всех пространств имен в приложении, он будет действовать для всех выполняемых модулей, включая `org.apache.http`, который был переименован в `com.microsoft.applicationinsights.core.dependencies.http`. Application Insights не сможет применять фильтрацию этих вызовов, так как вызов журнала выполняется библиотекой Apache. При уровне ведения журнала "Отладка" создается значительный объем данных журнала, и он не рекомендуется для экземпляров, выполняемых в рабочей среде.

## <a name="next-steps"></a>Дальнейшие действия
**Служба Application Insights настроена для серверного приложения Java. Что еще можно сделать?**

* [Отслеживать доступность веб-страниц][availability]
* [Отслеживать использование веб-страниц][usage]
* [Отслеживать использование и диагностировать проблемы в приложениях для устройств][platforms]
* [Написать код для отслеживания использования приложения][track]
* [Записать журналы диагностики][javalogs]

## <a name="get-help"></a>Получить помощь
* [Stack Overflow](https://stackoverflow.com/questions/tagged/ms-application-insights)
* [Сообщите о проблеме на GitHub](https://github.com/microsoft/ApplicationInsights-Java/issues)

<!--Link references-->

[availability]: ./monitor-web-app-availability.md
[data]: ./data-retention-privacy.md
[java]: java-get-started.md
[javalogs]: java-trace-logs.md
[platforms]: ./platforms.md
[track]: ./api-custom-events-metrics.md
[usage]: javascript.md

