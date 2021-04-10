---
title: Руководство по настройке Smartsheet для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Smartsheet.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/07/2019
ms.author: jeedes
ms.openlocfilehash: e9ee994564e175d3c41cfd5ce415ead8c67df353
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100103572"
---
# <a name="tutorial-configure-smartsheet-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Smartsheet

В этом руководстве описаны шаги, которые нужно выполнить в Smartsheet и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и (или) групп в Azure AD для [Smartsheet](https://www.smartsheet.com/pricing). Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Smartsheet.
> * Удаление в Smartsheet пользователей, которым больше не нужен доступ.
> * Синхронизация атрибутов пользователей между Azure AD и Smartsheet.
> * Единый вход в Smartsheet (рекомендуется).

> [!NOTE]
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md)
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор).
* [Клиент Smartsheet](https://www.smartsheet.com/pricing).
* Учетная запись пользователя с тарифным планом Smartsheet Enterprise или Enterprise Premier и разрешениями системного администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Smartsheet](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-smartsheet-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Smartsheet для поддержки подготовки с помощью Azure AD

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для Smartsheet, нужно включить подготовку SCIM для Smartsheet.

1. Войдите с правами **системного администратора** на **[портал Smartsheet](https://app.smartsheet.com/b/home)** и перейдите к разделу **Account Admin** (Администратор учетных записей).

    ![Администратор учетных записей Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-accountadmin.png)

2. Перейдите в раздел **Security Controls > User Auto Provisioning > Edit** (Элементы управления безопасностью > Автоматическая подготовка пользователей > Изменить).

    ![Smartsheet: элементы управления безопасностью](media/smartsheet-provisioning-tutorial/smartsheet-securitycontrols.png)

3. Добавьте и проверьте домены электронной почты для всех пользователей, которых вы намерены подготавливать в Azure AD для Smartsheet. Выберите **Not Enabled** (Не включено), чтобы все действия по подготовке поступали только из Azure AD, а также убедитесь, что список пользователей Smartsheet синхронизирован с назначениями в Azure AD.

    ![Smartsheet: подготовка пользователей](media/smartsheet-provisioning-tutorial/smartsheet-userprovisioning.png)

4. Когда проверка завершится, следует активировать домен. 

    ![Smartsheet: активация домена](media/smartsheet-provisioning-tutorial/smartsheet-activatedomain.png)

5. Создайте **секретный маркер**, который требуется для настройки автоматической подготовки пользователей в Azure AD. Для этого перейдите в раздел **Apps and Integrations** (Приложения и интеграции).

    ![Снимок экрана: страница администрирования в Smartsheet, где выделены аватар пользователя и элемент Apps and Integrations (Приложения и интеграции).](media/smartsheet-provisioning-tutorial/Smartsheet05.png)

6. Выберите **API Access** (Доступ к API). Щелкните **Generate new access token** (Создать новый маркер доступа).

    ![Снимок экрана: диалоговое окно Personal Settings (Личные настройки), где выделены элементы API Access (Доступ к API) и Generate new access token (Создание нового маркера доступа).](media/smartsheet-provisioning-tutorial/Smartsheet06.png)

7. Укажите имя для маркера доступа к API. Нажмите кнопку **ОК**.

    ![Снимок экрана для шага 1 из 2 (создание маркера доступа к API), где выделен элемент "ОК".](media/smartsheet-provisioning-tutorial/Smartsheet07.png)

8. Скопируйте значение API Access Token (Маркер доступа к API) и сохраните его, так как больше вы не сможете открыть его для просмотра. Это значение необходимо указать в поле **Секретный токен** в Azure AD.

    ![Маркер Smartsheet](media/smartsheet-provisioning-tutorial/Smartsheet08.png)

## <a name="step-3-add-smartsheet-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Smartsheet из коллекции приложений Azure AD

Добавьте Smartsheet из коллекции приложений Azure AD, чтобы начать управление подготовкой для Smartsheet. Если вы уже настроили единый вход в Smartsheet, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Smartsheet нужно выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Чтобы обеспечить согласованность ролей, назначенных пользователям в Smartsheet и Azure AD, мы рекомендуем использовать те же назначения ролей, которые указаны в полном списке пользователей Smartsheet. Чтобы получить этот список пользователей из Smartsheet, последовательно выберите **Account Admin > User Management > More Actions > Download User List (csv)** (Администратор учетных записей > Управление пользователями > Другие действия > Скачать список пользователей в формате CSV).

* Для доступа к некоторым функциям в приложении Smartsheet пользователю нужно иметь несколько ролей. Дополнительные сведения о типах пользователей и разрешениях в Smartsheet см. в статье [User Types and Permissions](https://help.smartsheet.com/learning-track/shared-users/user-types-and-permissions) (Типы пользователей и разрешения).

*  Если пользователю назначено несколько ролей в Smartsheet, **ОБЯЗАТЕЛЬНО** убедитесь, что эти назначения ролей правильно реплицированы в Azure AD, иначе пользователи могут безвозвратно потерять доступ к объектам Smartsheet. Каждая уникальная роль в Smartsheet **ОБЯЗАТЕЛЬНО** назначается отдельной группе в Azure AD. Пользователь **ОБЯЗАТЕЛЬНО** должен быть добавлен во все группы, соответствующие нужным ролям. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

## <a name="step-5-configure-automatic-user-provisioning-to-smartsheet"></a>Шаг 5. Настройка автоматической подготовки пользователей в Smartsheet 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Smartsheet на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-smartsheet-in-azure-ad"></a>Чтобы настроить в Azure AD автоматическую подготовку пользователей для Smartsheet, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Smartsheet**.

    ![Ссылка на Smartsheet в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите полученные ранее значения **SCIM 2.0 base URL** (Базовый URL-адрес SCIM 2.0) и Access Token (Маркер доступа) в поля **URL-адрес клиента** и **Секретный токен** соответственно. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Smartsheet. Если установить подключение не удалось, убедитесь, что у учетной записи Smartsheet есть разрешения системного администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Smartsheet** (Синхронизировать пользователей Azure Active Directory со Smartsheet).

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Smartsheet. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в Smartsheet при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |userName|Строка|&check;|
   |active|Логическое|
   |title|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |emails[type eq "work"].value|Строка|
   |externalId|Строка|
   |Роли|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Строка|


10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Smartsheet, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Укажите пользователей и (или) группы для подготовки в Smartsheet, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="connector-limitations"></a>Ограничения соединителя

* Smartsheet не поддерживает обратимое удаление. Когда атрибут пользователя **active** принимает значение False, Smartsheet удаляет этого пользователя без возможности восстановления.

## <a name="change-log"></a>Журнал изменений

* 16.06.2020 — добавлена поддержка корпоративных атрибутов расширения для пользователей: Cost Center (Центр затрат), Division (Подразделение), Manager (Руководитель) и Department (Отдел).
* 10.02.2021 г. — добавлена поддержка основных атрибутов "emails[type eq "work"]" для пользователей.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)