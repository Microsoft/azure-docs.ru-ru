---
title: Настройка мастера Log Analytics в Azure AD | Документация Майкрософт
description: Сведения о настройке Log Analytics.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: markvi
author: MarkusVi
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: ca84fa57cb3a26337038275d1b7491154915c90e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100574372"
---
# <a name="tutorial-configure-the-log-analytics-wizard"></a>Руководство по настройке мастера Log Analytics


В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка журналов аудита и входа в рабочей области Log Analytics;
> * выполнение запросов с помощью языка запросов Kusto;
> * создание правила генерации оповещений, которое отправляет оповещения при обращениях к конкретной учетной записи;
> * создание пользовательской книги с помощью шаблона быстрого запуска;
> * добавление запроса в существующий шаблон книги.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure хотя бы с одним администратором с лицензией P1. Если у вас нет подписки Azure, можно [зарегистрироваться и получить бесплатную пробную версию](https://azure.microsoft.com/free/).

- Клиент Azure AD.

- Пользователь, который является глобальным администратором или администратором безопасности для этого клиента Azure AD.


Ознакомьтесь со следующими статьями:

- [Руководство. Сбор и анализ журналов ресурсов из ресурса Azure](../../azure-monitor/essentials/tutorial-resource-logs.md)

- [Интеграция журналов действий с Log Analytics](./howto-integrate-activity-logs-with-log-analytics.md)

- [Управление учетными записями для аварийного доступа в Azure AD](../roles/security-emergency-access.md)

- [Краткий справочник по KQL](/azure/data-explorer/kql-quick-reference)

- [Изучение и предоставление общего доступа к данным об использовании c интерактивными книгами в Application Insights](../../azure-monitor/visualize/workbooks-overview.md)



## <a name="configure-a-workspace"></a>Настройка рабочей области 

В этой процедуре описана настройка журналов аудита и входа в рабочей области Log Analytics.
Настройка рабочей области Log Analytics состоит из двух основных этапов:
 
1. Создание рабочей области Log Analytics.
2. Настройка параметров диагностики.

**Настройка рабочей области** 


1. Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор.

2. Выполните поиск по запросу **рабочие области Log Analytics**.

    ![Поиск ресурсов, служб и документов](./media/tutorial-log-analytics-wizard/search-services.png)

3. На странице "Рабочие области Log Analytics" щелкните **Добавить**.

    ![Снимок экрана: кнопка "Добавить" на странице "Рабочие области Log Analytics"](./media/tutorial-log-analytics-wizard/add.png)

4.  На странице **Создание рабочей области Log Analytics** выполните следующие шаги:

    ![Создание рабочей области Log Analytics](./media/tutorial-log-analytics-wizard/create-log-analytics-workspace.png)

    1. Выберите свою подписку.

    2. Выберите группу ресурсов.
 
    3. В текстовом поле **Имя** введите имя рабочей области, например MytestWorkspace1.

    4. Выберите нужный регион.

5. Щелкните **Просмотр и создание**.

    ![Проверка и создание](./media/tutorial-log-analytics-wizard/review-create.png)

6. Щелкните **Создать** и дождитесь успешного завершения развертывания. Возможно, потребуется обновить страницу, чтобы увидеть новую рабочую область.

    ![Создание](./media/tutorial-log-analytics-wizard/create-workspace.png)

7. Выполните поиск по запросу **Azure Active Directory**.

    ![Снимок экрана: Azure Active Directory в поиске Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

8. В разделе **Мониторинг** щелкните **Параметр диагностики**.

    ![Снимок экрана: параметры диагностики, выбранные в разделе "Мониторинг".](./media/tutorial-log-analytics-wizard/diagnostic-settings.png)

9. На странице **Параметры диагностики** щелкните **Добавить параметр диагностики**.

    ![Добавление параметра диагностики](./media/tutorial-log-analytics-wizard/add-diagnostic-setting.png)

10. На странице **Параметр диагностики** выполните следующие шаги.

    ![Выбор параметров диагностики](./media/tutorial-log-analytics-wizard/select-diagnostics-settings.png)

    1. В разделе **Сведения о категории** выберите **AuditLogs** и **SigninLogs**.

    2. В разделе **Сведения о назначении** выберите **Отправить в Log Analytics**, а затем выберите созданную рабочую область Log Analytics. 
   
    3. Выберите команду **Сохранить**. 

## <a name="run-queries"></a>Выполнение запросов  

В этой процедуре показано, как выполнять запросы на **языке запросов Kusto**.


**Выполнение запроса**


1. Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор.

2. Выполните поиск по запросу **Azure Active Directory**.

    ![Снимок экрана: Azure Active Directory в поиске Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. В разделе **Мониторинг** щелкните **Журналы**.

4. На странице **Журналы** щелкните **Начало работы**.

5. В текстовом поле **Поиск* введите нужный запрос.

6. Нажмите кнопку **Запустить**.  


### <a name="kql-query-examples"></a>Примеры запросов на языке Kusto

Получение 10 случайных записей из входных данных:

`SigninLogs | take 10`

Получение списка операций входа, во время которых успешно применен условный доступ:

`SigninLogs | where ConditionalAccessStatus == "success" | project UserDisplayName, ConditionalAccessStatus` 


Подсчет количества успешных попыток:

`SigninLogs | where ConditionalAccessStatus == "success" | project UserDisplayName, ConditionalAccessStatus | count`


Совокупное число успешных операций входа по каждому сочетанию пользователя и дня:

`SigninLogs | where ConditionalAccessStatus == "success" | summarize SuccessfulSign-ins = count() by UserDisplayName, bin(TimeGenerated, 1d)` 


Просмотр количества выполнений пользователем определенной операции за определенный период времени:

`AuditLogs | where TimeGenerated > ago(30d) | where OperationName contains "Add member to role" | summarize count() by OperationName, Identity`


Сводные результаты по имени операции:

`AuditLogs | where TimeGenerated > ago(30d) | where OperationName contains "Add member to role" | project OperationName, Identity | evaluate pivot(OperationName)`


Объединение журналов аудита и вход с помощью внутреннего соединения:

`AuditLogs |where OperationName contains "Add User" |extend UserPrincipalName = tostring(TargetResources[0].userPrincipalName) | |project TimeGenerated, UserPrincipalName |join kind = inner (SigninLogs) on UserPrincipalName |summarize arg_min(TimeGenerated, *) by UserPrincipalName |extend SigninDate = TimeGenerated` 


Просмотр количества входов по типу клиентского приложения:

`SigninLogs | summarize count() by ClientAppUsed`

Подсчет входов по дням:

`SigninLogs | summarize NumberOfEntries=count() by bin(TimeGenerated, 1d)`

Получение 5 случайных записей со столбцами, выбранными для отображения результатов:

`SigninLogs | take 5 | project ClientAppUsed, Identity, ConditionalAccessStatus, Status, TimeGenerated `


Получение 5 первых записей в порядке убывания со столбцами, выбранными для отображения результатов

`SigninLogs | take 5 | project ClientAppUsed, Identity, ConditionalAccessStatus, Status, TimeGenerated `

Создание нового столбца путем объединения значений двух других столбцов:

`SigninLogs | limit 10 | extend RiskUser = strcat(RiskDetail, "-", Identity) | project RiskUser, ClientAppUsed`




## <a name="create-an-alert-rule"></a>Создание правила оповещения  

В этой процедуре описано, как настроить отправку уведомлений об использовании аварийной учетной записи.

**Чтобы создать правило оповещения, сделайте следующее.**


1. Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор.

2. Выполните поиск по запросу **Azure Active Directory**.

    ![Снимок экрана: Azure Active Directory в поиске Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. В разделе **Мониторинг** щелкните **Журналы**.

4. На странице **Журналы** щелкните **Начало работы**.

5. В текстовом поле **Поиск** введите `SigninLogs |where UserDisplayName contains "BreakGlass" | project UserDisplayName`.

6. Нажмите кнопку **Запустить**.  

7. На панели инструментов нажмите кнопку **Создать правило генерации оповещений**.

    ![Новое правило генерации оповещений](./media/tutorial-log-analytics-wizard/new-alert-rule.png)

8. На странице **Создание правила генерации оповещений** убедитесь, что область указана правильно.

9. В разделе **Условие** щелкните вариант **Whenever the average custom log search is greater than <logic undefined> count** (Всякий раз, когда среднее количество результатов настраиваемого поиска по журналам превышает <logic undefined>).

    ![Условие по умолчанию](./media/tutorial-log-analytics-wizard/default-condition.png)

10. На странице **Настроить логику сигналов** в разделе **Логика оповещений** выполните следующие шаги.

    ![Логика оповещений](./media/tutorial-log-analytics-wizard/alert-logic.png)

    1. Для параметра **На основе** выберите **Число результатов**.

    2. Для параметра **Оператор** выберите **Больше чем**.

    3. Для параметра **Пороговое значение** выберите значение **0**. 

11. На странице **Настроить логику сигналов** в разделе **Вычисляется на основе** выполните следующие шаги.

    ![Вычисляется на основе](./media/tutorial-log-analytics-wizard/evaluated-based-on.png)

    1. Для параметра **Период (в минутах)** выберите значение **5**.

    2. Для параметра **Частота (в минутах)** выберите значение **5**.

    3. Нажмите кнопку **Готово**. 

12. В области **Группа действий** щелкните **Выбор группы действий**. 

    ![Группа действий](./media/tutorial-log-analytics-wizard/action-group.png)

13. В области **Выбор группы действий для прикрепления к этому правилу генерации оповещений** щелкните **Создание группы действий**. 

    ![Создание группы действий](./media/tutorial-log-analytics-wizard/create-action-group.png)

14. На странице **Создание группы действий** выполните следующие шаги.

    ![Подробности об экземпляре](./media/tutorial-log-analytics-wizard/instance-details.png)

    1. В текстовом поле **Имя группы действий** введите **Моя группа действий**.

    2. В текстовом поле **Отображаемое имя** введите **Мои действия**.

    3. Щелкните **Review + create** (Просмотреть и создать). 

    4. Нажмите кнопку **Создать**.


15. В разделе **Настройка действия** выполните следующие шаги.

    ![Настройка действий](./media/tutorial-log-analytics-wizard/customize-actions.png)

    1. Щелкните **Тема сообщения**.

    2. В текстовом поле **Строка темы** введите значение `Breakglass account has been used`.

16. В разделе **Сведения о правиле генерации оповещений** выполните следующие шаги.

    ![Сведения о правиле генерации оповещений](./media/tutorial-log-analytics-wizard/alert-rule-details.png)

    1. В текстовом поле **Имя правила генерации оповещений** введите значение `Breakglass account`.

    2. В текстовом поле **Описание** введите значение `Your emergency access account has been used`.

17. Выберите **Создать правило генерации оповещений**.   


## <a name="create-a-custom-workbook"></a>Создание пользовательской книги

В этой процедуре показано, как создать книгу с помощью шаблона быстрого запуска.




1. Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор.

2. Выполните поиск по запросу **Azure Active Directory**.

    ![Снимок экрана: Azure Active Directory в поиске Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. В разделе **Мониторинг** щелкните **Книги**.

    ![Снимок экрана: раздел "Мониторинг" на портале Azure с выбранным параметром "Книги"](./media/tutorial-log-analytics-wizard/workbooks.png)

4. В разделе **Быстрый запуск** щелкните **Пустой**.

    ![Быстрый запуск](./media/tutorial-log-analytics-wizard/quick-start.png)

5. Нажмите кнопку **Добавить**.

    ![Добавление книги](./media/tutorial-log-analytics-wizard/add-workbook.png)

6. Щелкните **Добавить текст**.

    ![Добавление текста](./media/tutorial-log-analytics-wizard/add-text.png)


7. В текстовом поле введите значение `# Client apps used in the past week`, а затем щелкните **Редактирование завершено**.

    ![Текст книги](./media/tutorial-log-analytics-wizard/workbook-text.png)

8. В новой книге щелкните **Добавить**, а затем выберите **Добавить запрос**.

    ![Добавить запрос](./media/tutorial-log-analytics-wizard/add-query.png)

9. В текстовом поле запроса введите строку `SigninLogs | where TimeGenerated > ago(7d) | project TimeGenerated, UserDisplayName, ClientAppUsed | summarize count() by ClientAppUsed`.

10. Нажмите кнопку **Запуск запроса**.

    ![Снимок экрана: кнопка "Выполнить запрос".](./media/tutorial-log-analytics-wizard/run-workbook-query.png)

11. На панели инструментов в разделе **Визуализация** щелкните **Круговая диаграмма**.

    ![Круговая диаграмма](./media/tutorial-log-analytics-wizard/pie-chart.png)

12. Щелкните **Редактирование завершено**.

    ![Редактирование завершено](./media/tutorial-log-analytics-wizard/done-workbook-editing.png)



## <a name="add-a-query-to-a-workbook-template"></a>Добавление запроса в шаблон книги  

В этой процедуре показано, как добавить запрос в существующий шаблон книги. Этот пример основан на запросе, который возвращает распределение успешных и неуспешных попыток условного доступа.


1. Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор.

2. Выполните поиск по запросу **Azure Active Directory**.

    ![Снимок экрана: Azure Active Directory в поиске Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. В разделе **Мониторинг** щелкните **Книги**.

    ![Снимок экрана: меню с разделом "Мониторинг" и выбранным параметром "Книги"](./media/tutorial-log-analytics-wizard/workbooks.png)

4. В разделе **Условный доступ** щелкните **Сведения об условном доступе и создание отчетов**.

    ![Снимок экрана: вариант "Сведения об условном доступе и создание отчетов".](./media/tutorial-log-analytics-wizard/conditional-access-template.png)

5. На панели инструментов нажмите кнопку **Правка**.

    ![Снимок экрана: кнопка "Изменить".](./media/tutorial-log-analytics-wizard/edit-workbook-template.png)

6. На панели инструментов щелкните три точки, затем **Добавить** и выберите **Добавить запрос**.

    ![Добавление запроса по книге](./media/tutorial-log-analytics-wizard/add-custom-workbook-query.png)

7. В текстовом поле запроса введите строку `SigninLogs | where TimeGenerated > ago(20d) | where ConditionalAccessPolicies != "[]" | summarize dcount(UserDisplayName) by bin(TimeGenerated, 1d), ConditionalAccessStatus`.

8. Нажмите кнопку **Запуск запроса**.

    ![Снимок экрана: кнопка "Выполнить запрос" для запроса.](./media/tutorial-log-analytics-wizard/run-workbook-insights-query.png)

9. Щелкните **Диапазон времени**, а затем выберите **Задать в запросе**.

10. Щелкните **Визуализация**, а затем выберите **Линейчатая диаграмма**. 

11. Щелкните **Дополнительные параметры**, введите `Conditional Access status over the last 20 days` в качестве заголовка диаграммы и щелкните **Редактирование завершено**. 

    ![Настройка заголовка диаграммы](./media/tutorial-log-analytics-wizard/set-chart-title.png)








## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье, чтобы узнать, как управлять удостоверениями устройств с помощью портала Azure.
> [!div class="nextstepaction"]
> [Мониторинг](overview-monitoring.md)