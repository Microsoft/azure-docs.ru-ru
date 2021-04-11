---
title: Параметры платформы интеграции и автоматизации в Azure
description: 'Сравнение облачных служб Майкрософт, которые оптимизированы для выполнения задач интеграции: Power Automate, Logic Apps, Функции и Веб-задания.'
ms.topic: overview
ms.date: 04/09/2018
ms.custom: mvc
ms.openlocfilehash: e6bd97469a6b7b6c42427e130ac103954377c652
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105643366"
---
# <a name="choose-the-right-integration-and-automation-services-in-azure"></a>Выбор правильных служб интеграции и автоматизации в Azure

В этой статье сравниваются следующие облачные службы Майкрософт:

* [Microsoft Power Automate](https://flow.microsoft.com/) (прежнее название — Microsoft Flow)
* [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)
* [Функции Azure](https://azure.microsoft.com/services/functions/)
* [веб-задания службы приложений Azure](../app-service/webjobs-create.md)

Все эти службы используются для настройки интеграции и автоматизации бизнес-процессов. Они позволяют определять входные данные, действия, условия и выходные данные. Каждую отдельную службу можно запускать по расписанию или активировать по запросу. Каждая служба обладает уникальными преимуществами, но в этой статье описываются их различия. 

Если вы ищете более общее сравнение функций Azure с другими вариантами служб вычислений Azure, см. статью [Criteria for choosing an Azure compute service](/azure/architecture/guide/technology-choices/compute-comparison) (Критерии выбора службы вычислений Azure) и [Choosing an Azure compute option for microservices](/azure/architecture/microservices/design/compute-options) (Выбор варианта службы вычислений Azure для микрослужб).

## <a name="compare-microsoft-power-automate-and-azure-logic-apps"></a>Сравнение Microsoft Power Automate и Azure Logic Apps

Power Automate и Logic Apps спроектированы в рамках подхода *designer-first* как службы интеграции, с помощью которых можно создавать рабочие процессы. Обе службы интегрируются с разными корпоративными приложениями и приложениями SaaS. 

Служба Power Automate создана на основе Logic Apps. В этих двух службах используется один и тот же конструктор рабочих процессов и одни и те же [соединители](../connectors/apis-list.md). 

Служба Power Automate помогает офисным сотрудникам самостоятельно выполнять простые операции интеграции (например, процесс утверждения на основе библиотеки документов SharePoint), не обращаясь к разработчикам или ИТ-специалистам. Logic Apps также обеспечивает расширенную интеграцию (например, в процессах B2B), когда нужно выполнять операции Azure DevOps и применять методы обеспечения безопасности корпоративного класса. Как правило, с течением времени бизнес-процессы становятся более сложными. Поэтому на первых этапах можно использовать поток, а затем при необходимости преобразовать его в приложение логики.

Сведения в таблице ниже помогут определить, какую из этих двух служб лучше всего использовать для определенной интеграции.

|  | Power Automate | Logic Apps |
| --- | --- | --- |
| **Пользователи** |Офисные сотрудники, бизнес-пользователи, администраторы SharePoint |Профессиональные интеграторы и разработчики, ИТ-специалисты |
| **Сценарии** |Самообслуживание |Расширенные интеграции |
| **Средство разработки** |В браузере и мобильном приложении, только пользовательский интерфейс |В браузере и [Visual Studio](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) доступно [представление кода](../logic-apps/logic-apps-author-definitions.md). |
| **Управление жизненным циклом приложений (ALM)** |Разработка и тестирование в непроизводственных средах, распространение в рабочей среде по готовности |Azure DevOps: система управления версиями, тестирование, поддержка, автоматизация и управление в [Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) |
| **Возможности для администраторов** |Управление средами Power Automate и политиками защиты от потери данных, отслеживание лицензирования: [Центр администрирования](https://admin.flow.microsoft.com). |Управление группами ресурсов, подключениями, доступом и ведением журнала: [Портал Azure](https://portal.azure.com) |
| **Безопасность** |Журналы аудита безопасности Microsoft 365, защита от потери данных, [шифрование неактивных](https://wikipedia.org/wiki/Data_at_rest#Encryption) конфиденциальных данных |Обеспечение безопасности Azure: [система безопасности Azure](https://www.microsoft.com/en-us/trustcenter/Security/AzureSecurity), [Центр безопасности Azure](https://azure.microsoft.com/services/security-center/), [журналы аудита](https://azure.microsoft.com/blog/azure-audit-logs-ux-refresh/) |

## <a name="compare-azure-functions-and-azure-logic-apps"></a>Сравнение служб "Функции Azure" и Azure Logic Apps

Функции и Logic Apps — это службы Azure для использования бессерверных рабочих нагрузок. Функции Azure — это бессерверная служба вычислений, а Azure Logic Apps предоставляет бессерверные рабочие процессы. В обеих службах можно создавать сложные *оркестрации*. Оркестрация — это коллекция функций или шагов, называемых *действиями*  в Logic Apps, которые выполняются для реализации сложных задач. Например, для обработки пакета заказов можно запустить параллельное выполнение множества экземпляров функции, дождаться завершения их работы, а затем выполнить функцию, которая вычислит все полученные результаты.

Для Функций Azure оркестрации разрабатываются путем написания кода и использования [расширения "Устойчивые функции"](durable/durable-functions-overview.md). Для Logic Apps оркестрации можно создавать с помощью графического пользовательского интерфейса или изменения файлов конфигурации.

Вы можете комбинировать и сопоставлять служба при создании оркестраций, вызывая функции из приложений логики и приложения логики из функций. Выберите способ создания оркестраций с учетом ваших предпочтений и возможностей, предоставляемых каждой службой. В следующей таблице представлены некоторые основные различия между этими службами:

|  | Устойчивые функции | Logic Apps |
| --- | --- | --- |
| **Разработка** | Code-first (императивный подход) | Designer-first (декларативный подход) |
| **Соединение** | [Около десяти встроенных типов привязки](functions-triggers-bindings.md#supported-bindings); написание кода для пользовательских привязок | [Большая коллекция соединителей](../connectors/apis-list.md), [пакет интеграции Enterprise для сценариев B2B](../logic-apps/logic-apps-enterprise-integration-overview.md), [создание пользовательских соединителей](../logic-apps/custom-connector-overview.md) |
| **Действия** | Каждое действие является функцией Azure; написание кода для функций действий |[Большая коллекция готовых действий](../logic-apps/logic-apps-workflow-actions-triggers.md)|
| **Мониторинг** | [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) | [Портал Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md), [журналы Azure Monitor](../logic-apps/monitor-logic-apps.md)|
| **Управление** | [REST API](durable/durable-functions-http-api.md), [Visual Studio](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer) | [Портал Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md), [REST API](/rest/api/logic/), [PowerShell](/powershell/module/az.logicapp), [Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md) |
| **Контекст выполнения** | Можно запускать [локально](functions-runtime-overview.md) или в облаке | Можно запускать только в облаке|

<a name="function"></a>

## <a name="compare-functions-and-webjobs"></a>Сравнение функций и веб-заданий

Подобно решению "Функции Azure", компонент "Веб-задания" службы приложений Azure с пакетом SDK для веб-заданий является службой интеграции на основе модели *code-first*, предназначенной для разработчиков. Обе службы созданы на основе [службы приложений Azure](../app-service/overview.md) и поддерживают следующие возможности: [интеграция системы управления версиями](../app-service/deploy-continuous-deployment.md), [проверка подлинности](../app-service/overview-authentication-authorization.md) и [мониторинг с помощью интеграции Application Insights](functions-monitoring.md).

### <a name="webjobs-and-the-webjobs-sdk"></a>Веб-задания и пакет SDK для веб-заданий

Компонент Службы приложений *Веб-задания* дает возможность выполнить код или скрипт в контексте веб-приложения Службы приложений. *SDK для веб-заданий* — это предназначенная для веб-заданий платформа, которая упрощает написание кода для реагирования на события в службах Azure. Например, вы можете отреагировать на создание большого двоичного объекта образа в службе хранилища Azure, создав эскиз. Пакет SDK для веб-заданий выполняется как консольное приложение .NET, которое можно развернуть в веб-задании. 

Веб-задания и пакет SDK для веб-заданий лучше всего использовать вместе, но можно и по отдельности. Веб-задания могут выполнять любые программы или скрипты, выполняемые в песочнице службы приложений. Консольное приложение пакета SDK для веб-заданий можно запускать там же, где и остальные консольные приложения, например на локальных серверах.

### <a name="comparison-table"></a>Сравнительная таблица

Решение "Функции Azure" создано на основе пакета SDK для веб-заданий, поэтому оно использует много тех же триггеров событий и соединений, что и другие службы Azure. Ниже приведены некоторые факторы, которые следует учесть при выборе между решением "Функции Azure" и компонентом "Веб-задания" с пакетом SDK для веб-заданий.

|  | Функции | Компонент "Веб-задания" с пакетом SDK для веб-заданий |
| --- | --- | --- |
|**[Бессерверная модель приложения](https://azure.microsoft.com/solutions/serverless/) с [автоматическим масштабированием](event-driven-scaling.md)**|✔||
|**[Разработка и тестирование в браузере](./functions-get-started.md)** |✔||
|**[Оплата по мере использования](consumption-plan.md)**|✔||
|**[Интеграция с Logic Apps](functions-twitter-email.md)**|✔||
| **События триггера** |[Таймер](functions-bindings-timer.md)<br>[Очереди и большие двоичные объекты службы хранилища Azure](functions-bindings-storage-blob.md)<br>[Очереди и разделы служебной шины Azure](functions-bindings-service-bus.md)<br>[Azure Cosmos DB](functions-bindings-cosmosdb.md)<br>[Центры событий Azure](functions-bindings-event-hubs.md)<br>[HTTP или веб-перехватчик (GitHub, Slack)](functions-bindings-http-webhook.md)<br>[Сетка событий Azure](functions-bindings-event-grid.md)|[Таймер](functions-bindings-timer.md)<br>[Очереди и большие двоичные объекты службы хранилища Azure](functions-bindings-storage-blob.md)<br>[Очереди и разделы служебной шины Azure](functions-bindings-service-bus.md)<br>[Azure Cosmos DB](functions-bindings-cosmosdb.md)<br>[Центры событий Azure](functions-bindings-event-hubs.md)<br>[Файловая система](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Files/FileTriggerAttribute.cs)|
| **Поддерживаемые языки**  |C#<br>F#<br>JavaScript<br>Java<br>Python<br>PowerShell |C#<sup>1</sup>|
|**Диспетчеры пакетов**|NPM и NuGet|NuGet<sup>2</sup>|

<sup>1</sup> Компонент "Веб-задания" (без пакета SDK для веб-заданий) поддерживает C#, Java, JavaScript, Bash, CMD-файлы, BAT-файлы, PowerShell, PHP, TypeScript, Python и т. д. Это не полный список. Веб-задания могут выполнять любые программы или скрипты, выполняемые в песочнице службы приложений.

<sup>2</sup> Компонент "Веб-задания" (без пакета SDK для веб-заданий) поддерживает NPM и NuGet.

### <a name="summary"></a>Сводка

Функции Azure обеспечивают большую продуктивность разработки, чем компонент "Веб-задания" службы приложений Azure. Они также предусматривают больше вариантов языков программирования, сред разработки, интеграции служб Azure и ценовых категорий. В большинстве случаев они являются оптимальным вариантом.

Ниже приведены два сценария, для которых компонент "Веб-задания" может быть лучшим вариантом.

* Требуется больший контроль над кодом, который прослушивает события, — над объектом `JobHost`. Функции предоставляют ограниченное количество методов настройки поведения `JobHost` в файле [host.json](functions-host-json.md). Иногда необходимо выполнять действия, которые невозможно указать с помощью строки в файле JSON. Например, только пакет SDK для веб-заданий позволяет настроить пользовательскую политику повтора для службы хранилища Azure.
* У вас есть приложение Службы приложений, для которого необходимо выполнить фрагменты кода и управлять ими совместно в той же среде Azure DevOps.

Для других сценариев, в которых необходимо выполнять фрагменты кода для интеграции со службами Azure или сторонними службами, выберите решение "Функции Azure", а не компонент "Веб-задания" с пакетом SDK для веб-заданий.

<a name="together"></a>

## <a name="power-automate-logic-apps-functions-and-webjobs-together"></a>Совместное использование Power Automate, Logic Apps, Функций и Веб-заданий

Вам не обязательно выбирать какую-то одну из этих служб. Их можно интегрировать друг с другом и с внешними службами.

Поток может вызвать приложение логики. Приложение логики и функция могут вызывать друг друга. Пример см. в статье [Создание функции, интегрируемой с Azure Logic Apps](functions-twitter-email.md).

Качество интеграции между Power Automate, Logic Apps и Функциями постоянно улучшается. Вы можете создать что-нибудь в одной службе, а использовать в других.

Дополнительные сведения о службах интеграции см. в следующих источниках:

* [Leveraging Azure Functions & Azure App Service for integration scenarios by Christopher Anderson](https://www.biztalk360.com/integrate-2016-resources/leveraging-azure-functions-azure-app-service-integration-scenarios/) (Использование функций Azure и службы приложений Azure в сценариях интеграции. Автор: Кристофер Андерсон (Christopher Anderson))
* [Integrations Made Simple by Charles Lamanna](https://www.biztalk360.com/integrate-2016-resources/integrations-made-simple/) (Упрощенные возможности интеграции. Автор: Чарльз Ламанна (Charles Lamanna))
* [Запись веб-трансляции по Logic Apps](https://aka.ms/logicappslive)
* [Часто задаваемые вопросы о Power Automate](/power-automate/frequently-asked-questions)

## <a name="next-steps"></a>Дальнейшие действия

Начните работу с создания потока, приложения логики или приложения-функции. Выберите любую из следующих ссылок:

* [Начало работы с Power Automate](/power-automate/getting-started)
* [Создайте приложение логики](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [Создание первой функции Azure](./functions-get-started.md)