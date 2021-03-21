---
title: Ресурсы для создания пользовательских соединителей Azure Sentinel | Документация Майкрософт
description: Сведения о доступных ресурсах для создания настраиваемых соединителей для Azure Sentinel. К таким методам относятся агент Log Analytics и API, Logstash, Logic Apps, PowerShell и функции Azure.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/09/2021
ms.author: bagol
ms.openlocfilehash: 25f83088bdc55dbafe7ccf0ff06b0c6595c9ea71
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101724359"
---
# <a name="resources-for-creating-azure-sentinel-custom-connectors"></a>Ресурсы для создания настраиваемых соединителей Azure Sentinel

Azure Sentinel предоставляет широкий спектр [встроенных соединителей для служб Azure и внешних решений](connect-data-sources.md), а также поддерживает прием данных из некоторых источников без выделенного соединителя.

Если вы не можете подключить источник данных к Sentinel Azure с помощью любого из имеющихся решений, рассмотрите возможность создания собственного соединителя источников данных.

Полный список поддерживаемых соединителей см. в записи блога [Sentinel: Общие сведения о соединителях (CEF, syslog, Direct, агент, Custom и другие)](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-sentinel-the-connectors-grand-cef-syslog-direct-agent/ba-p/803891) .

## <a name="compare-custom-connector-methods"></a>Сравнение методов пользовательских соединителей

В следующей таблице сравниваются общие сведения о каждом методе создания пользовательских соединителей, описанных в этой статье. Чтобы получить дополнительные сведения о каждом методе, выберите ссылки в таблице.

|Описание метода  |Функция | Бессерверные приложения    |Сложность  |
|---------|---------|---------|---------|
|**[Агент Log Analytics](#connect-with-the-log-analytics-agent)** <br>Лучшие сведения о сборе файлов из локальной среды и источников IaaS   | Только коллекция файлов  |   Нет      |Низкий         |
|**[Logstash](#connect-with-logstash)** <br>Подходящие для локальных источников и IaaS, любой источник, для которого доступен подключаемый модуль, и организации, уже знакомые с Logstash  | Доступные подключаемые модули, а также пользовательский подключаемый модуль, возможности обеспечивают значительную гибкость.   |   Не требуется запустить кластер ВИРТУАЛЬНОЙ машины или виртуальной машины           |   Низшую поддерживает множество сценариев с подключаемыми модулями      |
|**[Logic Apps](#connect-with-logic-apps)** <br>Высокая стоимость; Избегайте больших объемов данных <br>Лучшие для облачных источников с низким уровнем громкости  | Программирование без кода обеспечивает ограниченную гибкость, не требуя поддержки реализации алгоритмов.<br><br> Если доступное действие уже не поддерживает ваши требования, создание настраиваемого действия может усложнить работу.    |    Да         |   Низшую Простая разработка с некотором кодом      |
|**[PowerShell](#connect-with-powershell)** <br>Оптимально подходит для создания прототипов и периодических отправок файлов | Прямая поддержка сбора файлов. <br><br>PowerShell можно использовать для получения дополнительных источников, но для этого потребуется написание кода и Настройка сценария как службы.      |Нет               |  Низкий       |
|**[Log Analytics API](#connect-with-the-log-analytics-api)** <br>Подходит для независимых поставщиков программных продуктов, реализующих интеграцию, и для уникальных требований к сбору   | Поддерживает все возможности, доступные в коде.  | Зависит от реализации           |     Высокий    |
|**[Функции Azure](#connect-with-azure-functions)** Лучшие решения для облачных источников больших объемов и уникальных требований к сбору  | Поддерживает все возможности, доступные в коде.  |  Да             |     Высоком требуется знание программирования    |
|     |         |                |

> [!TIP]
> Сравнение использования Logic Apps и функций Azure для одного и того же соединителя см. в следующих статьях:
> 
> - [Быстрое получение журналов брандмауэра веб-приложения в Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/ingest-fastly-web-application-firewall-logs-into-azure-sentinel/ba-p/1238804)
> - Office 365 (сообщество Azure Sentinel GitHub): [соединитель приложения логики](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks/Get-O365Data)  |  [соединитель функций Azure](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/O365%20Data)
> 

## <a name="connect-with-the-log-analytics-agent"></a>Подключение с помощью агента Log Analytics

Если источник данных доставляет события в файлы, для создания настраиваемого соединителя рекомендуется использовать агент Azure Monitor Log Analytics.

- Дополнительные сведения см. [в разделе Сбор пользовательских журналов в Azure Monitor](../azure-monitor/agents/data-sources-custom-logs.md).

- Пример этого метода см. в разделе [Сбор пользовательских источников данных JSON с помощью агента log Analytics для Linux в Azure Monitor](../azure-monitor/agents/data-sources-json.md).

## <a name="connect-with-logstash"></a>Подключение с помощью Logstash

Если вы знакомы с [Logstash](https://www.elastic.co/logstash), вам может потребоваться использовать Logstash с [подключаемым модулем вывода Logstash для](connect-logstash.md) создания настраиваемого соединителя с помощью метки Azure.

С помощью подключаемого модуля выходных данных Azure Sentinel Logstash можно использовать любые подключаемые модули ввода и фильтрации Logstash, а также настроить Sentinel Azure в качестве выходных данных для конвейера Logstash. Logstash имеет обширную библиотеку подключаемых модулей, которая позволяет вводить данные из различных источников, таких как концентраторы событий, Apache Kafka, файлы, базы данных и облачные службы. Используйте подключаемые модули фильтрации для анализа событий, фильтрации ненужных событий, маскировки значений и многого другого.

Примеры использования Logstash в качестве настраиваемого соединителя см. в следующих статьях:

- [Поиск капитала с одним нарушением екомендуется в журналах AWS с помощью Sentinel Azure](https://techcommunity.microsoft.com/t5/azure-sentinel/hunting-for-capital-one-breach-ttps-in-aws-logs-using-azure/ba-p/1019767) (блог)
- [Radware с руководством по реализации Sentinel Azure](https://support.radware.com/ci/okcsFattach/get/1025459_3)

Примеры полезных подключаемых модулей Logstash см. в следующих статьях:

- [Подключаемый модуль ввода Cloudwatch](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-cloudwatch.html)
- [Подключаемый модуль концентраторов событий Azure](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-azure_event_hubs.html)
- [Подключаемый модуль входного облачного хранилища Google](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-google_cloud_storage.html)
- [Подключаемый модуль ввода Google_pubsub](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-google_pubsub.html)

> [!TIP]
> Logstash также поддерживает масштабированный сбор данных с помощью кластера. Дополнительные сведения см. в статье [Использование виртуальной машины Logstash с балансировкой нагрузки в масштабе](https://techcommunity.microsoft.com/t5/azure-sentinel/scaling-up-syslog-cef-collection/ba-p/1185854).
>

## <a name="connect-with-logic-apps"></a>Подключение с помощью Logic Apps

Используйте [приложение логики Azure](../logic-apps/index.yml) , чтобы создать бессерверный настраиваемый соединитель для Azure Sentinel.

> [!NOTE]
> При создании бессерверных соединителей, использующих Logic Apps, использование Logic Apps для соединителей может оказаться дорогостоящим для больших объемов данных.
>
> Этот метод рекомендуется использовать только для источников данных с низким объемом или для передачи данных.
>

1. **Используйте один из следующих триггеров для запуска Logic Apps**:

    |Триггер  |Описание  |
    |---------|---------|
    |**Повторяющаяся задача**     |   Например, спланируйте приложение логики для регулярного получения данных из конкретных файлов, баз данных или внешних API. <br>Дополнительные сведения см. [в разделе Создание, планирование и выполнение повторяющихся задач и рабочих процессов в Azure Logic Apps](../connectors/connectors-native-recurrence.md).      |
    |**Активация по запросу**     | Запустите приложение логики по запросу для ручного сбора и тестирования данных. <br>Дополнительные сведения см. в разделе  [вызов, активация или вложение приложений логики с помощью конечных точек HTTPS](../logic-apps/logic-apps-http-endpoint.md).        |
    |**Конечная точка HTTP/S**     |  Рекомендуется для потоковой передачи, а также в случае, если исходная система может начать передачу данных. <br>Дополнительные сведения см. в разделе [вызов конечных точек службы по протоколу HTTP или HTTPS](../connectors/connectors-native-http.md).       |
    |     |         |

1. **Используйте любой из соединителей приложения логики, считывающих данные для получения событий**. Пример:

    - [Подключение к REST API](/connectors/custom-connectors/)
    - [Подключение к SQL Server](/connectors/sql/)
    - [Подключение к файловой системе](/connectors/filesystem/)

    > [!TIP]
    > Пользовательские соединители для интерфейсов API RESTFUL, серверов SQL Server и файловых систем также поддерживают извлечение данных из локальных источников данных. Дополнительные сведения см. в документации по [установке локального шлюза данных](/connectors/filesystem/) .
    >

1. **Подготовка сведений, которые необходимо получить**.

    Например, [действие анализ JSON](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action) используется для доступа к свойствам в содержимом JSON, что позволяет выбрать эти свойства из списка динамического содержимого при указании входных данных для приложения логики.

    Дополнительные сведения см. [в разделе Выполнение операций с данными в Azure Logic Apps](../logic-apps/logic-apps-perform-data-operations.md).

1. **Запишите данные в log Analytics**.

    Дополнительные сведения см. в документации по [сборщику данных Azure log Analytics](/connectors/azureloganalyticsdatacollector/) .

Примеры создания пользовательского соединителя для Sentinel Azure с помощью Logic Apps см. в следующих статьях:

- [Создание конвейера данных с помощью API сборщика данных](/connectors/azureloganalyticsdatacollector/)
- [Palo Alto Присма Logic App Connector с помощью веб-перехватчика](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks/Ingest-Prisma) (сообщество Azure Sentinel GitHub)
- [Обеспечение безопасности вызовов Microsoft Teams с помощью запланированной активации](https://techcommunity.microsoft.com/t5/azure-sentinel/secure-your-calls-monitoring-microsoft-teams-callrecords/ba-p/1574600) (блог)
- Прием [индикаторов угроз АЛИЕНВАУЛТ откс в Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/ingesting-alien-vault-otx-threat-indicators-into-azure-sentinel/ba-p/1086566) (блог)
- [Отправка журналов TAP proofpoint On Demand в Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/sending-proofpoint-tap-logs-to-azure-sentinel/ba-p/767727) (блог)


## <a name="connect-with-powershell"></a>Подключение с помощью PowerShell

[Сценарий PowerShell upload-азмониторлог](https://www.powershellgallery.com/packages/Upload-AzMonitorLog/) позволяет использовать PowerShell для потоковой передачи событий или контекстной информации в Azure Sentinel из командной строки. Эта потоковая передача эффективно создает пользовательский соединитель между источником данных и Azure Sentinel.

Например, следующий сценарий передает CSV-файл в метку Azure:

``` PowerShell
Import-Csv .\testcsv.csv
| .\Upload-AzMonitorLog.ps1
-WorkspaceId '69f7ec3e-cae3-458d-b4ea-6975385-6e426'
-WorkspaceKey $WSKey
-LogTypeName 'MyNewCSV'
-AddComputerName
-AdditionalDataTaggingName "MyAdditionalField"
-AdditionalDataTaggingValue "Foo"
```

Сценарий [PowerShell upload-азмониторлог](https://www.powershellgallery.com/packages/Upload-AzMonitorLog/) использует следующие параметры:

|Параметр  |Описание  |
|---------|---------|
|**WorkspaceId**     |   ИДЕНТИФИКАТОР рабочей области Sentinel Azure, где будут храниться данные.  [Найдите идентификатор и ключ рабочей области](#find-your-workspace-id-and-key).  |
|**WorkspaceKey**     |   Первичный или вторичный ключ для рабочей области Sentinel Azure, в которой будут храниться данные. [Найдите идентификатор и ключ рабочей области](#find-your-workspace-id-and-key).  |
|**логтипенаме**     |    Имя пользовательской таблицы журнала, в которой будут храниться данные. Суффикс **_CL** будет автоматически добавлен в конец имени таблицы.  |
|**аддкомпутернаме**     |   Если этот параметр существует, сценарий добавляет имя текущего компьютера в каждую запись журнала в поле **Computer**.      |
|**тагжедазурересаурцеид**     | Если этот параметр существует, скрипт связывает все отправленные записи журнала с указанным ресурсом Azure. <br><br>Эта ассоциация позволяет перегружать записи журнала для запросов контекста ресурсов и соответствует управлению доступом на основе ролей, ориентированных на ресурсы.       |
|**аддитионалдататаггингнаме**     |      Если этот параметр существует, скрипт добавляет еще одно поле в каждую запись журнала с настроенным именем и значением, настроенным для параметра **аддитионалдататаггингвалуе** . <br><br>В этом случае **аддитионалдататаггингвалуе** не должен быть пустым. |
|**аддитионалдататаггингвалуе**     |   Если этот параметр существует, скрипт добавляет еще одно поле в каждую запись журнала с настроенным значением и именем поля, настроенным для параметра **аддитионалдататаггингнаме** . <br><br>Если параметр **аддитионалдататаггингнаме** пуст, но задано значение, то имя поля по умолчанию — это **тегирования**.       |
|     |         |

### <a name="find-your-workspace-id-and-key"></a>Поиск идентификатора и ключа рабочей области

Найдите сведения о параметрах **WorkspaceID** и **WorkspaceKey** в Azure Sentinel:

1. В поле Метка Azure выберите **Параметры** слева и перейдите на вкладку **параметры рабочей области** .

1. В разделе **Начало работы с log Analytics**  >  **1 подключение к источнику данных** выберите **Управление агентами Windows и Linux**.

1. Найдите идентификатор рабочей области, первичный ключ и вторичный ключ на вкладках **серверы Windows** .
## <a name="connect-with-the-log-analytics-api"></a>Подключение к Log Analytics API

Вы можете передать события в Azure Sentinel с помощью Log Analytics API сборщика данных, чтобы напрямую вызвать конечную точку RESTFUL.

При непосредственном вызове конечной точки RESTFUL требуется дополнительное программирование, а также обеспечивается дополнительная гибкость.

Дополнительные сведения см. в разделе [API сборщика данных log Analytics](../azure-monitor/logs/data-collector-api.md), особенно в следующих примерах.

- [C#](../azure-monitor/logs/data-collector-api.md#c-sample)
- [Python 2;](../azure-monitor/logs/data-collector-api.md#python-2-sample)

## <a name="connect-with-azure-functions"></a>Подключение с помощью функций Azure

Используйте функции Azure совместно с API RESTFUL и различными языками программирования, такими как [PowerShell](../azure-functions/functions-reference-powershell.md), для создания бессерверного соединителя.

Примеры использования этого метода см. в следующих статьях:

- [Подключение стандарта VMware для облачной конечной точки к Azure Sentinel с помощью функции Azure](connect-vmware-carbon-black.md)
- [Подключение Okta Single Sign-On к Azure Sentinel с помощью функции Azure](connect-okta-single-sign-on.md)
- [Подключение Proofpoint On Demand TAP к Azure Sentinel с помощью функции Azure](connect-proofpoint-tap.md)
- [Подключение виртуальной машины Qualys к Azure Sentinel с помощью функции Azure](connect-qualys-vm.md)
- [Прием данных форматов XML, CSV или других форматов](../azure-monitor/logs/create-pipeline-datacollector-api.md#ingesting-xml-csv-or-other-formats-of-data)
- [Мониторинг масштабирования с помощью Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/monitoring-zoom-with-azure-sentinel/ba-p/1341516) (блог) Azure
- [Развертывание приложение-функция для получения данных API управления Office 365 в Azure Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/O365%20Data) (сообщество Azure Sentinel GitHub)

## <a name="parse-your-custom-connector-data"></a>Анализ данных пользовательского соединителя

Вы можете использовать встроенный метод синтаксического анализа пользовательского соединителя для извлечения нужных сведений и заполнения соответствующих полей в Azure Sentinel.

Пример:

- **Если вы использовали Logstash**, используйте подключаемый модуль фильтра [грок](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) для анализа данных.
- **Если вы использовали функцию Azure**, проанализируйте данные с помощью кода. Дополнительные сведения см. в разделе [средства синтаксического анализа](normalization.md#parsers).

В Azure Sentinel поддерживается синтаксический анализ во время выполнения запроса. Анализ во время выполнения запроса позволяет отправлять данные в исходном формате, а затем при необходимости выполнять анализ по требованию.

Анализ во время выполнения запроса также означает, что вам не нужно заранее знать точную структуру данных, при создании пользовательского соединителя или даже сведений, которые необходимо извлечь. Вместо этого проанализируйте данные в любое время, даже во время расследования.

> [!NOTE]
> Обновление средства синтаксического анализа также применяется к данным, которые вы уже получили в Azure Sentinel.
> 
## <a name="next-steps"></a>Дальнейшие действия

Используйте данные, полученные в Azure Sentinel, чтобы защитить среду с помощью любого из следующих процессов:

- [Контроль с помощью оповещений](quickstart-get-visibility.md)
- [Визуализируйте и Отслеживайте данные](tutorial-monitor-your-data.md)
- [Изучение инцидентов](tutorial-investigate-cases.md)
- [Обнаружение угроз](tutorial-detect-threats-built-in.md)
- [Автоматизация предотвращения угроз](tutorial-respond-threats-playbook.md)
- [Поиск угроз](hunting.md)