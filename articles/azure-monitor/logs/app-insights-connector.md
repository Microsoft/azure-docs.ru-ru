---
title: Просмотр данных приложений Azure Application Insights | Документация Майкрософт
description: Решение "Соединитель Application Insights" можно использовать для диагностики проблем с производительностью и для того, чтобы узнать, какие действия пользователи выполняют в вашем приложении, которое отслеживается с помощью Application Insights.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/13/2019
ms.openlocfilehash: c21ed461280dfa617c852fe6c18c30e5d697b704
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031282"
---
# <a name="application-insights-connector-management-solution-deprecated"></a>Решение по управлению Соединителем Application Insights (устарело)

![Символ Application Insights](./media/app-insights-connector/app-insights-connector-symbol.png)

>[!NOTE]
> Благодаря поддержке [запросов между ресурсами](../logs/cross-workspace-query.md) решение по управлению Соединителем Application Insights больше не требуется. Его было объявлено устаревшим и удалено из Azure Marketplace, а портал OMS официально объявлен устаревшим 15 января 2019 года для коммерческого облака Azure. Этот компонент будет выведен из эксплуатации 30 марта 2019 года для облака Azure для US Gov организаций.
>
>Существующие соединения будут работать до 30 июня 2019 года.  В связи с устареванием портала OMS на нем невозможно настроить или удалить существующие подключения. Чтобы получить сведения о сценарии для удаления существующих соединителей с помощью PowerShell см. раздел [Удаление соединителя с помощью PowerShell](#removing-the-connector-with-powershell) указанный ниже.
>
>Рекомендации по запросах журнала Application Insights для нескольких приложений см. в статье [Объединение нескольких ресурсов Azure Monitor Application Insights](./unify-app-resource-data.md). Дополнительные сведения об устаревании портала OMS см. в статье [Перемещение портала OMS в Azure](./oms-portal-transition.md).
>
> 

С помощью решения "Соединитель Application Insights" можно диагностировать проблемы с производительностью, а также узнать, какие действия пользователи выполняют в вашем приложении, которое отслеживается с помощью [Application Insights](../app/app-insights-overview.md). В Log Analytics доступны представления данных телеметрии приложения, которые отображаются для разработчиков в Application Insights. Тем не менее при интеграции приложений Application Insights с Log Analytics видимость приложений увеличивается, так как данные операций и приложений находятся в одном месте. Наличие тех же представлений, что и у разработчиков, поможет вам сотрудничать с ними. Благодаря общим представлениям вы сможете быстрее обнаруживать проблемы с платформой и приложением, а также решать их.

Используя решение, вы можете:

- просматривать все приложения Application Insights в одном месте, даже если они находятся в разных подписках Azure;
- сопоставлять данные инфраструктуры с данными приложений;
- визуализировать данные приложения с перспективами в поиске по журналам;
- сводить данные из Log Analytics в приложении Application Insights на портале Azure.


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="connected-sources"></a>Подключенные источники

В отличие от большинства других решений Log Analytics, данные для соединителя Application Insights не собираются агентами. Все данные, используемые решением, поступают непосредственно из Azure.

| Подключенный источник | Поддерживается | Описание |
| --- | --- | --- |
| [Агенты Windows](./../agents/agent-windows.md) | Нет | Решение не собирает сведения из агентов Windows. |
| [Агенты Linux](../vm/quick-collect-linux-computer.md) | Нет | Решение не собирает сведения из агентов Linux. |
| [Группы управления SCOM](../agents/om-agents.md) | Нет | Решение не собирает сведения из агентов в подключенной группе управления SCOM. |
| [Учетная запись хранения Azure](../essentials/resource-logs.md#send-to-log-analytics-workspace) | Нет | Решение не собирает сведения из службы хранилища Azure. |

## <a name="prerequisites"></a>Предварительные условия

- Для доступа к сведениям о соединителе Application Insights необходима подписка Azure.
- Необходимо иметь хотя бы один настроенный ресурс Application Insights.
- Необходимо быть владельцем или участником ресурса Application Insights.

## <a name="configuration"></a>Конфигурация

1. Включите решение "Аналитика веб-приложений Azure" из [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps) или выполните инструкции по [добавлению решений Log Analytics из коллекции решений](../insights/solutions.md).
2. Перейдите на [портал Azure](https://portal.azure.com). Выберите **Все службы**, чтобы открыть Application Insights. Затем выполните поиск по запросу "Application Insights". 
3. В разделе **Подписки** выберите подписку с ресурсами Application Insights, а затем в разделе **Имя** выберите одно или несколько приложений.
4. Выберите команду **Сохранить**.

Приблизительно через 30 минут данные станут доступными, а на плитке Application Insights появятся данные, как показано на следующем изображении:

![Плитка Application Insights](./media/app-insights-connector/app-insights-tile.png)

Моменты, которые следует учитывать:

- Приложения Application Insights можно связать только с одной рабочей областью Log Analytics.
- С Log Analytics можно связать [ресурсы Application Insights только уровня "Базовый" или "Корпоративный"](https://azure.microsoft.com/pricing/details/application-insights). Тем не менее можно использовать уровень "Бесплатный" Log Analytics.

## <a name="management-packs"></a>Пакеты управления

В этом решении не предусматривается установка пакетов управления в подключенных группах управления.

## <a name="use-the-solution"></a>Использование решения

В следующих разделах описываются способы использования колонок, отображаемых на панели мониторинга Application Insights, для просмотра данных из приложений и работы с ними.

### <a name="view-application-insights-connector-information"></a>Просмотр сведений о соединителе Application Insights

Щелкните плитку **Application Insights**, чтобы открыть панель мониторинга **Application Insights** для просмотра следующих колонок.

![Снимок экрана панели мониторинга Application Insights, в которой показаны колонки для приложений, объем данных и доступность.](./media/app-insights-connector/app-insights-dash01.png)

![Снимок экрана панели мониторинга Application Insights, в которой показаны колонки для запросов сервера, сбоев и исключений.](./media/app-insights-connector/app-insights-dash02.png)

Панель мониторинга содержит колонки, приведенные в таблице. В каждой колонке содержится максимум 10 элементов, соответствующих таким указанным критериям, как область действия и диапазон времени. Вы можете выполнить поиск по журналам, в результате которого возвращаются все записи, если щелкнуть заголовок колонки или **Показать все** в ее нижней части.


| **Столбец** | **Описание** |
| --- | --- |
| Applications — Number of applications | Указывает число приложений в ресурсах приложения. Кроме того, приводятся имена приложений и количество записей приложений в каждом из них. Щелкните количество, чтобы выполнить поиск по журналам: <code>ApplicationInsights &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName</code>. <br><br>  Щелкните имя приложения, чтобы выполнить поиск по журналам для приложения, в котором показаны записи приложения для каждого узла, записи по типам телеметрии и все данные по типам (в зависимости от последнего дня). |
| Data Volume — Hosts sending data | Показывает число узлов компьютера, которые отправляют данные. Здесь также приводятся узлы компьютера и количество записей для каждого узла. Щелкните количество, чтобы выполнить поиск по журналам: <code>ApplicationInsights &#124; summarize AggregatedValue = sum(SampledCount) by Host</code>. <br><br> Щелкните имя компьютера, чтобы выполнить поиск по журналам для узла, в котором показаны записи приложения для каждого узла, записи по типам телеметрии и все данные по типам (в зависимости от последнего дня). |
| Availability — Webtest results | Показывает кольцевую диаграмму с результатами веб-тестов, в которой указано, положительные они или отрицательные. Щелкните диаграмму, чтобы выполнить поиск по журналам: <code>ApplicationInsights &#124; where TelemetryType == "Availability" &#124; summarize AggregatedValue = sum(SampledCount) by AvailabilityResult</code>. <br><br> Результаты показывают количество пройденных и непройденных тестов. Показывает все веб-приложения с трафиком за последнюю минуту. Щелкните имя приложения, чтобы просмотреть поиск по журналам с подробными сведениями о непройденных веб-тестах. |
| Server Requests — Requests per hour | Показывает график запросов сервера в час для различных приложений. Наведите указатель мыши на линию на диаграмме, чтобы просмотреть 3 приложения, получающие наибольшее количество запросов на определенный момент времени. Кроме того, отображает список приложений, получающих запросы, и число запросов за выбранный период. <br><br>Щелкните диаграмму, чтобы выполнить поиск по журналам для <code>ApplicationInsights &#124; where TelemetryType == "Request" &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName, bin(TimeGenerated, 1h)</code>, после чего отобразится график с подробными сведениями о количестве запросов сервера в час для различных приложений. <br><br> Щелкните приложение в списке, чтобы выполнить поиск по журналам для <code>ApplicationInsights &#124; where ApplicationName == "yourapplicationname" and TelemetryType == "Request" and iff(isnotnull(toint(RequestSuccess)), RequestSuccess == false, RequestSuccess == "false") == true</code>, после чего отобразится список запросов, диаграммы для запросов за период времени и длительность запросов, а также список кодов ответов на запросы.   |
| Failures — Failed requests per hour | Показывает график со сведениями о количестве невыполненных запросов приложения в час. Наведите указатель мыши на диаграмму, чтобы просмотреть 3 приложения с наибольшим количеством невыполненных запросов на определенный момент времени. Кроме того, отображает список приложений и сведения о количестве невыполненных запросов для каждого из них. Щелкните диаграмму, чтобы выполнить поиск по журналам для <code>ApplicationInsights &#124; where TelemetryType == "Request" and iff(isnotnull(toint(RequestSuccess)), RequestSuccess == false, RequestSuccess == "false") == true &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName, bin(TimeGenerated, 1h)</code>, после чего отобразится график с подробными сведениями о невыполненных запросах приложения. <br><br>Щелкните элемент в списке, чтобы выполнить поиск по журналам для <code>ApplicationInsights &#124; where ApplicationName == "yourapplicationname" and TelemetryType == "Request" and iff(isnotnull(toint(RequestSuccess)), RequestSuccess == false, RequestSuccess == "false") == true</code>, после чего отобразятся невыполненные запросы, диаграммы с невыполненными запросами за период времени и длительность запросов, а также список кодов ответов на невыполненные запросы. |
| Exceptions — Exceptions per hour | Показывает график со сведениями о количестве исключений в час. Наведите указатель мыши на диаграмму, чтобы просмотреть 3 приложения с наибольшим количеством исключений на определенный момент времени. Кроме того, отображает список приложений и сведения о количестве исключений для каждого из них. Щелкните диаграмму, чтобы выполнить поиск по журналам для <code>ApplicationInsights &#124; where TelemetryType == "Exception" &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName, bin(TimeGenerated, 1h)</code>, после чего отобразится график с подробными сведениями об исключениях. <br><br>Щелкните элемент в списке, чтобы выполнить поиск по журналам для <code>ApplicationInsights &#124; where ApplicationName == "yourapplicationname" and TelemetryType == "Exception"</code>, после чего отобразится список исключений, диаграммы с исключениями на определенный момент времени и невыполненные запросы, а также список типов исключений.  |

### <a name="view-the-application-insights-perspective-with-log-search"></a>Просмотр перспективы Application Insights с поиском по журналам

Если щелкнуть какой-либо элемент на панели мониторинга, отобразится перспектива Application Insights, показанная в результатах поиска. Перспектива предоставляет более подробную визуализацию в зависимости от выбранного типа телеметрии. Таким образом содержимое визуализации изменяется для различных типов телеметрии.

Если щелкнуть в любом месте в колонке приложений, отобразится перспектива **Приложения** по умолчанию.

![Перспектива "Приложения" Application Insights](./media/app-insights-connector/applications-blade-drill-search.png)

В перспективе приведен обзор выбранного приложения.

В колонке **Доступность** приведено представление другой перспективы, в котором можно просмотреть результаты веб-тестов и связанные невыполненные запросы.

![Перспектива "Доступность" Application Insights](./media/app-insights-connector/availability-blade-drill-search.png)

Если щелкнуть в любом месте в колонках **Server Requests** (Запросы сервера) или **Сбои**, компоненты перспективы изменятся, предоставляя данные визуализации, связанные с запросами.

![Колонка "Сбои" Application Insights](./media/app-insights-connector/server-requests-failures-drill-search.png)

Если щелкнуть в любом месте в колонке **Исключения**, отобразятся данные визуализации, соответствующие исключениям.

![Колонка "Исключения" Application Insights](./media/app-insights-connector/exceptions-blade-drill-search.png)

Вне зависимости от того, что вы щелкнете на панели мониторинга **соединителя Application Insights** на странице **Поиск**, в любом запросе, возвращающем данные Application Insights, будет отображаться перспектива Application Insights. Например, если вы просматриваете данные Application Insights, в запросе **&#42;** также будет отображаться вкладка перспективы, как показано на следующем изображении:

![Application Insights](./media/app-insights-connector/app-insights-search.png)

Компоненты перспективы обновляются в зависимости от поискового запроса. Это означает, что результаты можно отфильтровать с помощью любого поля поиска, в результате чего вы сможете просматривать данные из:

- всех ваших приложений;
- одного выбранного приложения;
- группы приложений.

### <a name="pivot-to-an-app-in-the-azure-portal"></a>Сведение в приложение на портале Azure

Колонки Соединителя Application Insights предназначены для сведения данных в выбранное приложение Application Insights *при использовании портала Azure*. Решение можно использовать как высокоуровневую платформу мониторинга, которая позволяет устранять неполадки в приложении. При появлении потенциальной проблемы в одном из подключенных приложений вы можете получить о ней дополнительные сведения в поиске Log Analytics или свести их непосредственно в приложении Application Insights.

Чтобы свести данные, нажмите символ с многоточием (**…**), который отображается в конце каждой строки, а затем выберите **Open in Application Insights** (Открыть в Application Insights).

>[!NOTE]
>Параметр **Open in Application Insights** (Открыть в Application Insights) недоступен на портале Azure.

![Открыть в Application Insights](./media/app-insights-connector/open-in-app-insights.png)

### <a name="sample-corrected-data"></a>Данные исправления выборки

В Application Insights предоставлена функция *[исправления выборки](../app/sampling.md)*, с помощью которой можно уменьшить трафик данных телеметрии. При включении выборки в приложении Application Insights уменьшается количество записей, хранимых в Application Insights и в службе Log Analytics. В то время как на странице **соединителя Application Insights** и в перспективах данные остаются согласованными, для пользовательских запросов данные выборки необходимо исправлять вручную.

Ниже приведен пример исправления выборки в запросе поиска по журналам:

```
ApplicationInsights | summarize AggregatedValue = sum(SampledCount) by TelemetryType
```

Поле **Sampled Count** (Количество данных выборки) есть во всех записях и обозначает количество точек данных, представленных в записи. Если включить функцию выборки для вашего приложения Application Insights, значение поля **Sampled Count** (Количество данных выборки) будет больше 1. Чтобы подсчитать число записей, создаваемых приложением, суммируйте значения полей **Sampled Count** (Количество данных выборки).

Выборка влияет только на общее число записей, создаваемых приложением. Значение выборки не нужно исправлять для таких полей метрик, как **RequestDuration** или **AvailabilityDuration**, так как в этих полях отображается среднее значение представленных записей.

## <a name="input-data"></a>Входные данные

Решение принимает следующие типы данных телеметрии из подключенных приложений Application Insights:

- Доступность
- Исключения
- Requests
- Просмотры страниц. Чтобы ваша рабочая область получала данные о просмотре страниц, в приложении необходимо настроить сбор этих сведений. Дополнительные сведения см. в разделе [Просмотры страниц](../app/api-custom-events-metrics.md#page-views).
- Настраиваемые события. Чтобы ваша рабочая область получала данные о настраиваемых событиях, в приложении необходимо настроить сбор этих сведений. Дополнительные сведения см. в разделе [TrackEvent (Отслеживание событий)](../app/api-custom-events-metrics.md#trackevent).

Как только данные станут доступны, они поступят из Application Insights в Log Analytics.

## <a name="output-data"></a>Выходные данные

Для каждого типа входных данных создается запись с данными о *типе**ApplicationInsights*. Записи ApplicationInsights обладают свойствами, приведенными в следующих разделах.

### <a name="generic-fields"></a>Универсальные поля

| Свойство. | Описание |
| --- | --- |
| Type | ApplicationInsights |
| ClientIP |   |
| TimeGenerated | Время создания записи |
| ApplicationId | Ключ инструментирования приложения Application Insights |
| ApplicationName | Имя приложения Application Insights |
| RoleInstance | Идентификатор узла сервера |
| DeviceType | Устройство клиента |
| ScreenResolution |   |
| Continent | Континент, на котором создан запрос |
| Страна или регион | Страна или регион, где был создан запрос |
| Province | Провинция, область или район, в котором создан запрос |
| Город | Город, в котором создан запрос |
| isSynthetic | Указывает, был ли запрос создан пользователем или автоматически. True = автоматизированный метод или False = пользователь создан |
| SamplingRate | Процентная доля данных телеметрии, созданных с помощью пакета SDK, отправленного на портал. Диапазон 0,0–100,0. |
| SampledCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| IsAuthenticated | Значение true или false |
| OperationID | Элементы с одинаковым идентификатором операций отображаются на портале как связанные элементы. Как правило, это идентификатор запроса. |
| ParentOperationID | Идентификатор родительской операции |
| OperationName |   |
| SessionId | Идентификатор GUID для уникальной идентификации сеанса, в котором был создан запрос |
| SourceSystem | ApplicationInsights |

### <a name="availability-specific-fields"></a>Поля со сведениями о доступности

| Свойство. | Описание |
| --- | --- |
| TelemetryType | Доступность |
| AvailabilityTestName | Имя веб-теста |
| AvailabilityRunLocation | Географический источник HTTP-запроса |
| AvailabilityResult | Указывает успешный результат веб-теста |
| AvailabilityMessage | Сообщение, прикрепленное к веб-тесту |
| AvailabilityCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| DataSizeMetricValue | 1,0 или 0,0. |
| DataSizeMetricCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| AvailabilityDuration | Время длительности веб-теста (в миллисекундах) |
| AvailabilityDurationCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| AvailabilityValue |   |
| AvailabilityMetricCount |   |
| AvailabilityTestId | Уникальный идентификатор GUID веб-теста |
| AvailabilityTimestamp | Точная метка времени теста доступности |
| AvailabilityDurationMin | Для записей выборки в этом поле отображается минимальная продолжительность веб-теста (в миллисекундах) для представленных точек данных |
| AvailabilityDurationMax | Для записей выборки в этом поле отображается максимальная продолжительность веб-теста (в миллисекундах) для представленных точек данных |
| AvailabilityDurationStdDev | Для записей выборки в этом поле отображается стандартное отклонение всех значений продолжительности веб-тестов (в миллисекундах) для представленных точек данных |
| AvailabilityMin |   |
| AvailabilityMax |   |
| AvailabilityStdDev | &nbsp;  |

### <a name="exception-specific-fields"></a>Поля со сведениями об исключениях

| Type | ApplicationInsights |
| --- | --- |
| TelemetryType | Исключение |
| ExceptionType | Тип исключения |
| ExceptionMethod | Метод, который создает исключение |
| ExceptionAssembly | В сборке указана платформа и версия, а также маркер безопасности открытого ключа |
| ExceptionGroup | Тип исключения |
| ExceptionHandledAt | Указывает уровень, на котором обработано исключение |
| ExceptionCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| ExceptionMessage | Сообщение исключения |
| ExceptionStack | Полный стек исключения |
| ExceptionHasStack | Значение true, если исключение содержит стек |



### <a name="request-specific-fields"></a>Поля со сведениями о запросах

| Свойство. | Описание |
| --- | --- |
| Type | ApplicationInsights |
| TelemetryType | Запрос |
| ResponseCode | HTTP-отклик, отправленный клиенту. |
| RequestSuccess | Указывает успешное или неудачное выполнение. True или False. |
| RequestID | Идентификатор, позволяющий уникально идентифицировать запрос |
| RequestName | GET или POST + базовый URL-адрес |
| RequestDuration | Время длительности запроса (в секундах) |
| URL-адрес | URL-адрес запроса, не включая узел |
| Узел | Узел веб-сервера |
| URLBase | Полный URL-адрес запроса |
| ApplicationProtocol | Тип протокола, используемого приложением |
| RequestCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| RequestDurationCount | 100/(частота выборки). Например, 4 = &gt; 25 %. |
| RequestDurationMin | Для записей выборки в этом поле отображается минимальная продолжительность запроса (в миллисекундах) для представленных точек данных |
| RequestDurationMax | Для записей выборки в этом поле отображается максимальная продолжительность запроса (в миллисекундах) для представленных точек данных |
| RequestDurationStdDev | Для записей выборки в этом поле отображается стандартное отклонение всех значений продолжительности запросов (в миллисекундах) для представленных точек данных |

## <a name="sample-log-searches"></a>Пример поисков журналов

В этом решении нет примеров запросов поиска по журналам, отображаемых на панели мониторинга. Тем не менее пример запросов поиска по журналам с описаниями можно найти в разделе [Просмотр сведений о соединителе Application Insights](#view-application-insights-connector-information).

## <a name="removing-the-connector-with-powershell"></a>Удаление соединителя с помощью PowerShell
В связи с устареванием портала OMS на нем невозможно настроить или удалить существующие подключения. Вы можете удалить существующие соединители с помощью следующего сценария PowerShell. Чтобы выполнить эту операцию вы должны быть владельцем или участником рабочей области и читателем ресурса Application Insights.

```powershell
$Subscription_app = "App Subscription Name"
$ResourceGroup_app = "App ResourceGroup"
$Application = "Application Name"
$Subscription_workspace = "Workspace Subscription Name"
$ResourceGroup_workspace = "Workspace ResourceGroup"
$Workspace = "Workspace Name"

Connect-AzAccount
Set-AzContext -SubscriptionId $Subscription_app
$AIApp = Get-AzApplicationInsights -ResourceGroupName $ResourceGroup_app -Name $Application 
Set-AzContext -SubscriptionId $Subscription_workspace
Remove-AzOperationalInsightsDataSource -WorkspaceName $Workspace -ResourceGroupName $ResourceGroup_workspace -Name $AIApp.Id
```

Благодаря использованию следующего скрипта PowerShell, который запускает вызов API REST, вы можете получить список приложений. 

```powershell
Connect-AzAccount
$Tenant = "TenantId"
$Subscription_workspace = "Workspace Subscription Name"
$ResourceGroup_workspace = "Workspace ResourceGroup"
$Workspace = "Workspace Name"
$AccessToken = "AAD Authentication Token" 

Set-AzContext -SubscriptionId $Subscription_workspace
$LAWorkspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup_workspace -Name $Workspace

$Headers = @{
    "Authorization" = "Bearer $($AccessToken)"
    "x-ms-client-tenant-id" = $Tenant
}

$Connections = Invoke-RestMethod -Method "GET" -Uri "https://management.azure.com$($LAWorkspace.ResourceId)/dataSources/?%24filter=kind%20eq%20'ApplicationInsights'&api-version=2015-11-01-preview" -Headers $Headers
$ConnectionsJson = $Connections | ConvertTo-Json
```
Этот сценарий требует маркер проверки подлинности носителя, используемый в процессе проверки подлинности в Azure Active Directory. Одним из способов получения токена описан в статье [Data Sources — Create Or Update](/rest/api/loganalytics/datasources/createorupdate) (Источники данных: создание или обновление). Щелкните **Попробовать** и войдите в свою подписку Azure. Вы можете скопировать маркер носителя из **предварительного запроса**, как показано на следующем рисунке.


![Маркер носителя](media/app-insights-connector/bearer-token.png)


С помощью следующего запроса журнала вы можете получить список приложений.

```Kusto
ApplicationInsights | summarize by ApplicationName
```

## <a name="next-steps"></a>Дальнейшие действия

- Используйте [поиск по журналам](./log-query-overview.md), чтобы просматривать подробные сведения о приложениях Application Insights.