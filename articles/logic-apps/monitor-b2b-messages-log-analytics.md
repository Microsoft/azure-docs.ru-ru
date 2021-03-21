---
title: Мониторинг сообщений B2B с помощью Azure Monitor
description: Устранение неполадок с сообщениями AS2, X12 и EDIFACT путем настройки журналов Azure Monitor и сбора диагностических данных для Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 01/30/2020
ms.openlocfilehash: 765c15897bd5d435503d3bef07e76a93b148971c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100596851"
---
# <a name="set-up-azure-monitor-logs-and-collect-diagnostics-data-for-b2b-messages-in-azure-logic-apps"></a>Настройка журналов Azure Monitor и сбор диагностических данных для сообщений B2B в Azure Logic Apps

После настройки связи B2B между торговыми партнерами в учетной записи интеграции эти партнеры могут обмениваться сообщениями с помощью таких протоколов, как AS2, X12 и EDIFACT. Чтобы убедиться, что это взаимодействие работает так, как вы ожидаете, можно настроить [журналы Azure Monitor](../azure-monitor/logs/data-platform-logs.md) для учетной записи интеграции. [Azure Monitor](../azure-monitor/overview.md) помогает отслеживать облачные и локальные среды, чтобы можно было легко поддерживать доступность и производительность. С помощью журналов Azure Monitor можно записывать и хранить данные о данных и событиях времени выполнения, таких как события триггера, события запуска и события действий в [рабочей области log Analytics](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace). Для сообщений ведение журнала также собирает следующие сведения:

* число и состояние сообщений;
* состояние подтверждений;
* Взаимосвязи между сообщениями и подтверждениями
* подробное описание ошибок при сбоях;

Azure Monitor позволяет создавать [запросы журналов](../azure-monitor/logs/log-query-overview.md) , помогающие находить и просматривать эти сведения. Вы также можете [использовать эти диагностические данные с другими службами Azure](../logic-apps/monitor-logic-apps-log-analytics.md#extend-data), такими как служба хранилища Azure и концентраторы событий Azure.

Чтобы настроить ведение журнала для учетной записи интеграции, [установите решение Logic Apps B2B](#install-b2b-solution) в портал Azure. Это решение предоставляет агрегированные сведения для событий сообщений B2B. Затем, чтобы включить ведение журнала и создать запросы для этих сведений, настройте [журналы Azure Monitor](#set-up-resource-logs).

В этой статье показано, как включить ведение журнала Azure Monitor для учетной записи интеграции.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область Log Analytics. См. дополнительные сведения о [создании рабочей области Log Analytics](../azure-monitor/logs/quick-create-workspace.md).

* Приложение логики, которое настроено с Azure Monitor ведением журнала и отправляет эту информацию в Log Analytics рабочую область. Узнайте [, как настроить журналы Azure Monitor для приложения логики](../logic-apps/monitor-logic-apps.md).

* Учетная запись интеграции, связанная с приложением логики. Узнайте [, как связать учетную запись интеграции с приложением логики](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md).

<a name="install-b2b-solution"></a>

## <a name="install-logic-apps-b2b-solution"></a>Установка решения Logic Apps B2B

Прежде чем журналы Azure Monitor могут отвестить сообщения B2B для приложения логики, добавьте **Logic Apps B2B** решение в рабочую область log Analytics.

1. В поле поиска [портал Azure](https://portal.azure.com)введите `log analytics workspaces` , а затем выберите **log Analytics рабочие области**.

   ![Выберите "Log Analytics рабочие области".](./media/monitor-b2b-messages-log-analytics/find-select-log-analytics-workspaces.png)

1. В разделе **log Analytics рабочие области** выберите рабочую область.

   ![Выбор рабочей области Log Analytics](./media/monitor-b2b-messages-log-analytics/select-log-analytics-workspace.png)

1. В области Обзор в разделе Начало **работы с log Analytics**  >  **настроить решения мониторинга** выберите пункт **Просмотреть решения**.

   ![На панели "Обзор" выберите "Просмотр решений".](./media/monitor-b2b-messages-log-analytics/log-analytics-workspace.png)

1. В области Обзор выберите **Добавить**.

   ![В области "Обзор" добавьте новое решение](./media/monitor-b2b-messages-log-analytics/add-logic-apps-management-solution.png)

1. После открытия **Marketplace** в поле поиска введите `logic apps b2b` и выберите **Logic Apps B2B**.

   ![В Marketplace выберите "Управление Logic Apps".](./media/monitor-b2b-messages-log-analytics/select-logic-apps-b2b-solution.png)

1. В области описание решения выберите **создать**.

   ![Выберите "создать", чтобы добавить решение "Logic Apps B2B"](./media/monitor-b2b-messages-log-analytics/create-logic-apps-b2b-solution.png)

1. Проверьте и подтвердите Log Analytics рабочей области, в которую нужно установить решение, и нажмите кнопку **создать** еще раз.

   ![Выберите "создать" для "Logic Apps B2B"](./media/monitor-b2b-messages-log-analytics/confirm-log-analytics-workspace.png)

   После того, как Azure развернет решение в группу ресурсов Azure, содержащую рабочую область Log Analytics, решение появится в области сводки вашей рабочей области. При обработке сообщений B2B на этой панели обновляется число сообщений.

   ![Панель сводки по рабочей области](./media/monitor-b2b-messages-log-analytics/b2b-overview-messages-summary.png)

<a name="set-up-resource-logs"></a>

## <a name="set-up-azure-monitor-logs"></a>настройка журналов Azure Monitor;

Вы можете включить ведение журнала Azure Monitor непосредственно из учетной записи интеграции.

1. На [портале Azure](https://portal.azure.com) найдите и выберите свою учетную запись интеграции.

   ![Поиск и выбор учетной записи интеграции](./media/monitor-b2b-messages-log-analytics/find-integration-account.png)

1. В меню учетной записи интеграции в разделе **мониторинг** выберите **параметры диагностики**. Выберите **Добавить параметр диагностики**.

   ![В разделе "Мониторинг" выберите "параметры диагностики".](./media/monitor-b2b-messages-log-analytics/monitor-diagnostics-settings.png)

1. Чтобы создать параметр, выполните следующие действия.

   1. Укажите имя параметра.

   1. Установите флажок **Отправить в Log Analytics**.

   1. В поле **Подписка** выберите подписку Azure, связанную с рабочей областью log Analytics.

   1. Для **log Analytics рабочей области** выберите рабочую область, которую хотите использовать.

   1. В разделе **Журнал** выберите категорию **интегратионаккаунттраккинжевентс** , которая указывает категорию событий, которую необходимо записать.

   1. Когда все будет готово, нажмите кнопку **Сохранить**.

   Пример: 

   ![Настройка журналов Azure Monitor для сбор диагностических данных](./media/monitor-b2b-messages-log-analytics/send-diagnostics-data-log-analytics-workspace.png)

<a name="view-message-status"></a>

## <a name="view-message-status"></a>Просмотр состояния сообщения

После запуска приложения логики можно просмотреть состояние и данные об этих сообщениях в рабочей области Log Analytics.

1. В поле поиска [портал Azure](https://portal.azure.com) найдите и откройте рабочую область log Analytics.

1. В меню рабочей области выберите Logic Apps B2B **Сводка по рабочей области**  >  .

   ![Панель сводки по рабочей области](./media/monitor-b2b-messages-log-analytics/b2b-overview-messages-summary.png)

   > [!NOTE]
   > Если на плитке Logic Apps B2B не отображаются результаты сразу после выполнения, попробуйте выбрать **Обновить** или подождать некоторое время, прежде чем повторять попытку.

   По умолчанию на плитке **Logic Apps B2B** отображаются данные за один день. Чтобы изменить область данных на другой интервал, выберите элемент управления областью в верхней части страницы:

   ![Изменение интервала](./media/monitor-b2b-messages-log-analytics/change-summary-interval.png)

1. После появления панели мониторинга состояния сообщений можно просмотреть дополнительные сведения о сообщениях определенного типа, данные о которых отображаются за один день. Выберите плитку **AS2**, **X12** или **EDIFACT**.

   ![Просмотр состояний сообщений](./media/monitor-b2b-messages-log-analytics/workspace-summary-b2b-messages.png)

   Отобразится список сообщений в соответствии с выбранной плиткой. Ниже представлен пример списка сообщений AS2.

   ![Состояния и сведения о сообщениях AS2](./media/monitor-b2b-messages-log-analytics/as2-message-results-list.png)

   Для получения дополнительных сведений о свойствах для каждого типа сообщений см. указанные ниже описания свойств сообщения:

   * [Свойства сообщения AS2](#as2-message-properties)
   * [Свойства сообщения X12](#x12-message-properties)
   * [Свойства сообщения EDIFACT](#EDIFACT-message-properties)

<!--
1. To view or export the inputs and outputs for specific messages, select those messages, and select **Download**. When you're prompted, save the .zip file to your local computer, and then extract that file.

   The extracted folder includes a folder for each selected message. If you set up acknowledgements, the message folder also includes files with acknowledgement details. Each message folder has at least these files:
   
   * Human-readable files with the input payload and output payload details
   * Encoded files with the inputs and outputs

   For each message type, you can find the folder and file name formats here:

   * [AS2 folder and file name formats](#as2-folder-file-names)
   * [X12 folder and file name formats](#x12-folder-file-names)
   * [EDIFACT folder and file name formats](#edifact-folder-file-names)

   ![Download message files](./media/monitor-b2b-messages-log-analytics/download-messages.png)

1. To view all actions that have the same run ID, on the **Log Search** page, select a message from the message list.

   You can sort these actions by column, or search for specific results.

   * To search results with prebuilt queries, select **Favorites**.

   * Learn [how to build queries by adding filters](../logic-apps/create-monitoring-tracking-queries.md). Or learn more about [how to find data with log searches in Azure Monitor logs](../azure-monitor/logs/log-query-overview.md).

   * To change query in the search box, update the query with the columns and values that you want to use as filters.
-->

<a name="message-list-property-descriptions"></a>

## <a name="property-descriptions-and-name-formats-for-as2-x12-and-edifact-messages"></a>Описания свойств и форматы имен для сообщений AS2, X12 и EDIFACT

Для каждого типа сообщения ниже приведены описания свойств и форматы имен для загруженных файлов сообщений.

<a name="as2-message-properties"></a>

### <a name="as2-message-property-descriptions"></a>Описания свойств сообщения AS2

Ниже приведены описания свойств для каждого сообщения AS2.

| Свойство. | Описание |
|----------|-------------|
| **Отправитель** | Гостевой партнер, указанный в **параметрах получения**, или главный партнер, указанный в **параметрах отправки**, для соглашения AS2 |
| **Получателю** | Главный партнер, указанный в **параметрах получения**, или гостевой партнер, указанный в **параметрах отправки**, для соглашения AS2 |
| **Приложение логики** | Приложение логики, в котором настроены действия AS2 |
| **Состояние** | Состояние сообщения AS2 <br>Success: получено или отправлено корректное сообщение AS2, MDN не настроено. <br>Success: получено или отправлено корректное сообщение AS2, MDN настроено и получено либо отправлено. <br>Failed: получено некорректное сообщение AS2. MDN не настроено. <br>Pending: получено или отправлено корректное сообщение AS2. MDN настроено и ожидается. |
| **ПОДТВЕРЖДАЕТ** | Состояние сообщения MDN <br>Accepted: получено или отправлено положительное MDN. <br>Pending: ожидается получение или отправка MDN. <br>Rejected: получено или отправлено отрицательное MDN. <br>Not Required: в соглашении не настроено MDN. |
| **Направление** | Направление сообщения AS2 |
| **Tracking ID** | Идентификатор для корреляции всех триггеров и действий в приложении логики |
| **Код сообщения (ID)** | Идентификатор сообщения AS2, полученный из заголовков сообщения AS2 |
| **Timestamp** | Время обработки сообщения действием AS2 |
|||

<!--
<a name="as2-folder-file-names"></a>

### AS2 name formats for downloaded message files

Here are the name formats for each downloaded AS2 message folder and files.

| Folder or file | Name format |
|----------------|-------------|
| Message folder | [sender]\_[receiver]\_AS2\_[correlation-ID]\_[message-ID]\_[timestamp] |
| Input, output, and if set up, acknowledgement files | **Input payload**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_input_payload.txt </p>**Output payload**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_output\_payload.txt </p></p>**Inputs**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_inputs.txt </p></p>**Outputs**: [sender]\_[receiver]\_AS2\_[correlation-ID]\_outputs.txt |
|||
-->

<a name="x12-message-properties"></a>

### <a name="x12-message-property-descriptions"></a>Описания свойств сообщения X12

Ниже приведены описания свойств для каждого сообщения X12.

| Свойство. | Описание |
|----------|-------------|
| **Отправитель** | Гостевой партнер, указанный в **параметрах получения**, или главный партнер, указанный в **параметрах отправки**, для соглашения X12 |
| **Получателю** | Главный партнер, указанный в **параметрах получения**, или гостевой партнер, указанный в **параметрах отправки**, для соглашения X12 |
| **Приложение логики** | Приложение логики, в котором настроены действия X12 |
| **Состояние** | Состояние сообщения X12 <br>Success: получено или отправлено корректное сообщение X12. Функциональное подтверждение не настроено. <br>Success: получено или отправлено корректное сообщение X12. Функциональное подтверждение настроено и либо получено, либо отправлено. <br>Failed: получено или отправлено некорректное сообщение X12. <br>Pending: получено или отправлено корректное сообщение X12. Функциональное подтверждение настроено и ожидается его получение. |
| **ПОДТВЕРЖДАЕТ** | Состояние функционального подтверждения (997). <br>Accepted: получено или отправлено положительное функциональное подтверждение. <br>Rejected: получено или отправлено отрицательное функциональное подтверждение. <br>Pending: ожидается функциональное подтверждение, но оно еще не получено. <br>Pending: функциональное подтверждение создано, но его не удалось отправить в партнер. <br>Not Required: функциональное подтверждение не настроено. |
| **Направление** | Направление сообщения X12 |
| **Tracking ID** | Идентификатор для корреляции всех триггеров и действий в приложении логики |
| **Тип сообщения** | Тип сообщения EDI X12 |
| **ICN** | Контрольный номер обмена для сообщения X12 |
| **TSCN** | Контрольный номер набора транзакций для сообщения X12 |
| **Timestamp** | Время обработки сообщения действием X12 |
|||

<!--
<a name="x12-folder-file-names"></a>

### X12 name formats for downloaded message files

Here are the name formats for each downloaded X12 message folder and files.

| Folder or file | Name format |
|----------------|-------------|
| Message folder | [sender]\_[receiver]\_X12\_[interchange-control-number]\_[global-control-number]\_[transaction-set-control-number]\_[timestamp] |
| Input, output, and if set up, acknowledgement files | **Input payload**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_input_payload.txt </p>**Output payload**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_output\_payload.txt </p></p>**Inputs**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_inputs.txt </p></p>**Outputs**: [sender]\_[receiver]\_X12\_[interchange-control-number]\_outputs.txt |
|||
-->

<a name="EDIFACT-message-properties"></a>

### <a name="edifact-message-property-descriptions"></a>Описания свойств сообщения EDIFACT

Ниже приведены описания свойств для каждого сообщения EDIFACT.

| Свойство. | Описание |
|----------|-------------|
| **Отправитель** | Гостевой партнер, указанный в **параметрах получения**, или главный партнер, указанный в **параметрах отправки**, для соглашения EDIFACT |
| **Получателю** | Главный партнер, указанный в **параметрах получения**, или гостевой партнер, указанный в **параметрах отправки**, для соглашения EDIFACT |
| **Приложение логики** | Приложение логики, в котором настроены действия EDIFACT |
| **Состояние** | Состояние сообщения EDIFACT <br>Success: получено или отправлено корректное сообщение EDIFACT. Функциональное подтверждение не настроено. <br>Success: получено или отправлено корректное сообщение EDIFACT. Функциональное подтверждение настроено и либо получено, либо отправлено. <br>Failed: получено или отправлено некорректное сообщение EDIFACT <br>Pending: получено или отправлено корректное сообщение EDIFACT. Функциональное подтверждение настроено и ожидается его получение. |
| **ПОДТВЕРЖДАЕТ** | Состояние функционального подтверждения (CONTRL) <br>Accepted: получено или отправлено положительное функциональное подтверждение. <br>Rejected: получено или отправлено отрицательное функциональное подтверждение. <br>Pending: ожидается функциональное подтверждение, но оно еще не получено. <br>Pending: функциональное подтверждение создано, но его не удалось отправить в партнер. <br>Not Required: функциональное подтверждение не настроено. |
| **Направление** | Направление сообщения EDIFACT |
| **Tracking ID** | Идентификатор для корреляции всех триггеров и действий в приложении логики |
| **Тип сообщения** | Тип сообщения EDIFACT |
| **ICN** | Контрольный номер обмена сообщениями EDIFACT |
| **TSCN** | Контрольный номер набора транзакций для сообщения EDIFACT |
| **Timestamp** | Время обработки сообщения действием EDIFACT |
|||

<!--
<a name="edifact-folder-file-names"></a>

### EDIFACT name formats for downloaded message files

Here are the name formats for each downloaded EDIFACT message folder and files.

| Folder or file | Name format |
|----------------|-------------|
| Message folder | [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_[global-control-number]\_[transaction-set-control-number]\_[timestamp] |
| Input, output, and if set up, acknowledgement files | **Input payload**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_input_payload.txt </p>**Output payload**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_output\_payload.txt </p></p>**Inputs**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_inputs.txt </p></p>**Outputs**: [sender]\_[receiver]\_EDIFACT\_[interchange-control-number]\_outputs.txt |
|||
-->

## <a name="next-steps"></a>Дальнейшие действия

* [Создание, мониторинг и отслеживание запросов](../logic-apps/create-monitoring-tracking-queries.md)
