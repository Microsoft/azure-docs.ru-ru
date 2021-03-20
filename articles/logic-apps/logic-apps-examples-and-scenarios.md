---
title: Примеры & распространенных сценариев
description: Примеры, распространенные сценарии, учебники и пошаговые руководства для Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/28/2020
ms.openlocfilehash: 5f7afe044ab4f782f2028598c2c56e75edf95860
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92317378"
---
# <a name="common-scenarios-examples-tutorials-and-walkthroughs-for-azure-logic-apps"></a>Распространенные сценарии, примеры, учебники и пошаговые руководства для Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) помогает координировать и интегрировать различные службы, предоставляя [сотни готовых соединителей](../connectors/apis-list.md), начиная от SQL Server или от SAP до Azure Cognitive Services. Служба Logic Apps работает без серверов, поэтому вам не нужно заботиться об экземплярах и масштабировании. Достаточно определить рабочий процесс с триггером и действия для него. Базовая платформа обеспечивает масштабирование, доступность и производительность. Logic Apps особенно полезен для случаев использования и сценариев, в которых необходимо координировать действия в нескольких системах и службах.

Для получения сведений о возможностях и шаблонах, поддерживаемых Azure Logic Apps, в этой статье описываются общие начальные точки, примеры и сценарии.

## <a name="common-starting-points-for-logic-app-workflows"></a>Общие начальные точки для рабочих процессов приложения логики

Каждое приложение логики запускается при помощи [*триггера*](../logic-apps/logic-apps-overview.md#logic-app-concepts) (это всегда только один триггер). Он инициирует рабочий процесс приложения логики и передает в него данные. Некоторые соединители предоставляют триггер одного из следующих типов:

* *Опрашивающий триггер* — регулярно проверяет конечную точку службы на наличие новых данных. Когда появляются новые данные, триггер создает и запускает новый экземпляр рабочего процесса, передавая эти данные в качестве входных.

* *Извещающий триггер* — прослушивает данные в конечной точке службы и ожидает определенного события. Когда происходит это событие, триггер немедленно срабатывает, то есть создает и запускает новый экземпляр рабочего процесса, передавая в качестве входных все доступные данные.

Ниже приведены примеры, описывающие часто используемые триггеры.

* Триггеры *опроса* :

  * [Триггер **повторения**](../connectors/connectors-native-recurrence.md) позволяет задать дату и время начала, а также повторение для срабатывания приложения логики. Например, вы можете выбрать дни недели и время суток для запуска приложения логики. Дополнительные сведения см. в следующих статьях:<p>

    * [Планирование и выполнение повторяющихся автоматизированных задач, процессов и рабочих процессов с помощью Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md)
    * [Учебник. Создание автоматических повторяющихся рабочих процессов на основе расписания с помощью Azure Logic Apps](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)

  * Триггер **при получении сообщения электронной почты** позволяет приложению логики проверять наличие нового сообщения электронной почты от любого поставщика почты, поддерживаемого Logic Apps, например [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), [Gmail](/connectors/gmail/), [Outlook.com](/connectors/outlook/)и т. д.

    > [!IMPORTANT]
    > Только учетные записи для бизнеса G-Suite могут использовать соединитель Gmail без ограничений в приложениях логики. Если у вас есть учетная запись потребителя Gmail, вы можете использовать этот соединитель только с определенными утвержденными Google службами. Кроме того, вы можете [создать клиентское приложение Google, которое будет использоваться для проверки подлинности в соединителе Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Дополнительные сведения см. в статье [Политики безопасности и конфиденциальности данных для соединителей Google в Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

    Дополнительные сведения см. в следующих статьях:<p>

    * [Руководство по созданию автоматических рабочих процессов на основе утверждений с помощью Azure Logic Apps](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
    * [Руководство по Автоматизация задач обработки сообщений электронной почты с помощью Azure Logic Apps, Функций Azure и службы хранилища Azure](../logic-apps/tutorial-process-email-attachments-workflow.md)

  * Триггер [ **http**](../connectors/connectors-native-http.md) может вызывать конечную точку службы по протоколу HTTP или HTTPS. Дополнительные сведения см. в разделе [вызов, активация или вложение рабочих процессов с помощью конечных точек HTTP](../logic-apps/logic-apps-http-endpoint.md).

* Триггеры *push-уведомлений* :

  * Триггер [ **запроса**](../connectors/connectors-native-reqres.md) может принимать входящие HTTPS запросы.

  * [Триггер **Веб-перехватчик HTTP**](../connectors/connectors-native-webhook.md) подписывается на конечную точку службы, регистрируя в ней свой *URL-адрес обратного вызова*. В этом сценарии служба сама уведомляет триггер о том, что произошло указанное событие, то есть триггер может не опрашивать службу.

После наступления указанного события срабатывает триггер, который создает новый экземпляр рабочего процесса приложения логики и выполняет действия в рабочем процессе. Вам будут доступны все данные триггера на любом этапе рабочего процесса. Например, Twitter **в новом триггере твита** передает содержимое твита в приложение логики. Чтобы приступить к работе с Azure Logic Apps, воспользуйтесь следующими разделами краткого руководства:

* [Краткое руководство. Создание первого автоматизированного рабочего процесса с помощью Azure Logic Apps портал Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [Краткое руководство. Создание автоматизированных задач, процессов и рабочих процессов с помощью Azure Logic Apps — Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Краткое руководство. Создание автоматизированных рабочих процессов приложения логики и управление ими с помощью Visual Studio Code](../logic-apps/quickstart-create-logic-apps-visual-studio-code.md)

## <a name="control-flow-and-error-handling-capabilities"></a>Возможности потока управления и обработки ошибок

Приложения логики включают широкие возможности для расширенного потока управления, такие как условия, переключатели, циклы и области. Чтобы обеспечить устойчивость решений, в рабочих процессах можно также реализовать обработку ошибок и исключений.

* Выполнение различных действий на основе [условных операторов](../logic-apps/logic-apps-control-flow-conditional-statement.md) и [операторов switch](../logic-apps/logic-apps-control-flow-switch-statement.md).
* [Повторяющиеся действия или обработка элементов в массивах и коллекциях с помощью циклов](../logic-apps/logic-apps-control-flow-loops.md).
* [Группировка действий с использованием областей](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md).
* [Добавление обработки ошибок и исключений в рабочий процесс](../logic-apps/logic-apps-exception-handling.md)
* [Вариант использования. Как компания в сфере здравоохранения использует обработку исключений в приложении логики для рабочих процессов HL7 FHIR](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)

## <a name="create-custom-apis-and-connectors"></a>Создание настраиваемых интерфейсов API и соединителей

Для систем и служб, у которых нет опубликованных соединителей, можно также расширить приложения логики.

* [Создание настраиваемых API для вызова из Azure Logic Apps](../logic-apps/logic-apps-create-api-app.md)
* [Регулярно проверять наличие новых данных или событий с помощью шаблона триггера опроса](../logic-apps/logic-apps-create-api-app.md#polling-triggers)
* [Ожидание и прослушивание новых данных или событий с помощью шаблона триггера веб-перехватчика](../logic-apps/logic-apps-create-api-app.md#webhook-triggers)
* [Выполнение длительных задач с помощью шаблона действия опроса](../logic-apps/logic-apps-create-api-app.md#async-pattern)
* [Выполнение длительных задач с помощью шаблона действия веб-перехватчика](../logic-apps/logic-apps-create-api-app.md#webhook-actions)
* [Пользовательские соединители в Azure Logic Apps](../logic-apps/custom-connector-overview.md)

## <a name="build-business-to-business-b2b-solutions"></a>Создание решений "бизнес-бизнес" (B2B)

Для корпоративных решений интеграции и эффективного взаимодействия между организациями можно создавать автоматизированные масштабируемые рабочие процессы для этих сценариев с помощью Пакет интеграции Enterprise (EIP) с Azure Logic Apps. Хотя организации используют разные протоколы и форматы, они могут обмениваться сообщениями в электронном формате. EIP преобразует различные форматы в формат, в котором системы Организации могут обрабатывать и поддерживать стандартные отраслевые протоколы, включая AS2, X12, EDIFACT и RosettaNet. Чтобы создать эти решения, создайте учетную запись интеграции, которая представляет собой отдельный ресурс Azure, предоставляющий защищенный, масштабируемый и управляемый контейнер для артефактов, которые вы определяете и используете в рабочих процессах приложения логики. Например, артефакты включают торговые партнеры, соглашения, карты, схемы, сертификаты и конфигурации пакетной службы.

* [Обзор. решения для интеграции B2B Enterprise с Azure Logic Apps и Пакет интеграции Enterprise](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [Создание корпоративных учетных записей интеграции и управление ими для корпоративных интеграций B2B в Azure Logic Apps.](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

## <a name="access-azure-virtual-network-resources"></a>Доступ к ресурсам виртуальной сети Azure

Иногда приложениям логики и учетным записям интеграции требуется доступ к защищенным ресурсам, таким как виртуальные машины (VM) и другие системы или службы, которые находятся в виртуальной сети Azure. Чтобы настроить этот доступ, можно создать среду службы интеграции (ISE), в которой можно создавать и запускать приложения логики. ISE — это частный и изолированный экземпляр службы Logic Apps, использующий выделенные ресурсы, такие как хранилище, и выполняется отдельно от общедоступной, «глобальной», многоклиентской Logic Apps службы. Отделение изолированного закрытого экземпляра от общедоступного глобального экземпляра также помогает снизить влияние других клиентов Azure на производительность приложений, которая также называется эффектом "бесшумного" соседей.

* [Обзор: доступ к ресурсам виртуальной сети Azure из Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)
* [Подключение к виртуальным сетям Azure из Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)

## <a name="deploy-manage-and-monitor-logic-apps"></a>Развертывание приложений логики, управление ими и наблюдение за ними

Можно полностью разработать и развернуть приложения логики с помощью Visual Studio, Azure DevOps или любых других инструментов системы управления версиями и автоматической сборки. Для поддержки развертывания рабочих процессов и зависимых подключений в шаблоне ресурсов приложения логики используют шаблоны развертывания ресурсов Azure. Средства Visual Studio автоматически создают эти шаблоны, которые можно записать после изменения в систему управления версиями для управления версиями. Для ведения журналов уведомлений и диагностики состояния рабочего процесса служба Azure Logic Apps обеспечивает средства мониторинга и уведомления.

### <a name="deploy"></a>Развернуть

* [Краткое руководство. Создание автоматизированных задач, процессов и рабочих процессов с помощью Azure Logic Apps — Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Обзор: Автоматизация развертывания приложений логики](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
* [Создание шаблонов Azure Resource Manager для автоматизации развертываний для Azure Logic Apps](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
* [Развертывание шаблонов Azure Resource Manager для Azure Logic Apps](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md)
* [Пример. подключение к очередям служебной шины Azure из Azure Logic Apps и развертывание с помощью Azure Pipelines в Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-service-bus-queues-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Пример. подключение к учетным записям хранения Azure из Azure Logic Apps и развертывание с помощью Azure Pipelines в Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-storage-accounts-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Пример. Настройка действия приложения-функции для Azure Logic Apps и развертывание с помощью Azure Pipelines в Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/set-up-an-azure-function-app-action-for-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Пример. подключение к учетной записи интеграции из Azure Logic Apps и развертывание с помощью Azure Pipelines в Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-an-integration-account-from-azure-logic-apps-and-deploy-by-using-azure-devops-pipelines/)
* [Пример. Координация Azure Pipelines с помощью Azure Logic Apps](/samples/azure-samples/azure-logic-apps-pipeline-orchestration/azure-devops-orchestration-with-logic-apps/)

### <a name="manage"></a>Управление

* [Управление приложениями логики с помощью Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)
* [Создание учетных записей интеграции для интеграции B2B Enterprise и управление ими](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)
* [Управление средой службы интеграции (ISE) в Azure Logic Apps](../logic-apps/ise-manage-integration-service-environment.md)

### <a name="monitor"></a>Монитор

* [Отслеживание состояния выполнения, просмотр журнала триггеров и настройка оповещений для Azure Logic Apps](../logic-apps/monitor-logic-apps.md)
* [Настройка журналов Azure Monitor и получение диагностических данных для Azure Logic Apps](../logic-apps/monitor-logic-apps-log-analytics.md)
* [Настройка журналов Azure Monitor и сбор диагностических данных для сообщений B2B в Azure Logic Apps](../logic-apps/monitor-b2b-messages-log-analytics.md)
* [Просмотр и создание запросов для мониторинга и отслеживания в журналах Azure Monitor для Azure Logic Apps](../logic-apps/create-monitoring-tracking-queries.md)

## <a name="handle-content-types-conversions-and-transformations"></a>Работа с типами содержимого, преобразованиями и преобразованиями

Можно использовать и преобразовывать различные типы содержимого с помощью многих функций, написанных на [языке определения рабочего процесса](./logic-apps-workflow-definition-language.md) Azure Logic Apps. Например, можно осуществлять взаимное преобразование строк, JSON и XML с помощью выражений рабочего процесса `@json()` и `@xml()`. Обработчик Logic Apps сохраняет типы содержимого, чтобы обеспечить передачу содержимого между службами без потерь.

* [Обрабатывайте типы содержимого в Azure Logic Apps](../logic-apps/logic-apps-content-type.md), например `application/` ,, `application/octet-stream` и `multipart/formdata`
* [Справочное руководство по использованию функций в выражениях для Azure Logic Apps и Power Automate](../logic-apps/workflow-definition-language-functions-reference.md)
* [Схема языка определения рабочих процессов в Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md)

## <a name="other-integrations-and-capabilities"></a>Прочие возможности интеграции и функции

Azure Logic Apps интегрируется со многими службами, такими как службы "функции Azure", "Управление API Azure", "служба приложений Azure" и "пользовательские конечные точки HTTP", например "остальные" и "SOAP".

* [Вызов функций Azure из Azure Logic Apps](../logic-apps/logic-apps-azure-functions.md)
* [Учебник. вызов или Активация приложений логики с помощью функций Azure и служебной шины Azure](../logic-apps/logic-apps-scenario-function-sb-trigger.md)
* [Руководство. Создание информационной панели для потоковой передачи Customer Insights с помощью Azure Logic Apps и функций Azure](../logic-apps/logic-apps-scenario-social-serverless.md)
* [Руководство. Создание функции, которая интегрируется с Azure Logic Apps и Azure Cognitive Services для анализа записей Twitter тональности](../azure-functions/functions-twitter-email.md)
* [Руководство. Создание информационной панели социального страхования на основе искусственного интеллекта с помощью Power BI и Azure Logic Apps](https://aka.ms/logicappsdemo)
* [Руководство. Отслеживание изменений виртуальной машины с помощью Azure Logic Apps и службы "Сетка событий Azure"](../event-grid/monitor-virtual-machine-changes-event-grid-logic-app.md)
* [Руководство по удаленному мониторингу и отправке уведомлений в Центре Интернета вещей с помощью службы Azure Logic Apps, обеспечивающей подключение между Центром Интернета вещей и почтовым ящиком](../iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps.md)
* [Блог. вызов служб SOAP с помощью Azure Logic Apps](/archive/blogs/logicapps/using-soap-services-with-logic-apps)

## <a name="end-to-end-scenarios"></a>Комплексные сценарии

* [Документация по интеграции комплексного управления случаями с помощью таких служб Azure, как Logic Apps](https://aka.ms/enterprise-integration-e2e-case-management-utilities-logic-apps)

## <a name="customer-stories"></a>Истории клиентов

Узнайте, как Azure Logic Apps, а также другие службы Azure и продукты Майкрософт помогли [этим компаниям](https://aka.ms/logic-apps-customer-stories) повысить гибкость и сосредоточиться на основных бизнес-целях благодаря упрощению, организации, автоматизации и оркестрации сложных процессов.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [соединителях для Logic Apps](../connectors/apis-list.md)
* Узнайте о [сценариях интеграции B2B Enterprise с Azure Logic Apps](../logic-apps/logic-apps-enterprise-integration-overview.md)