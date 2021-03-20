---
title: Руководство по настройке Atlassian Cloud для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в Atlassian Cloud.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2019
ms.author: jeedes
ms.openlocfilehash: 691d35267c255c933a8098b99301fbb795a3cd0c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96181073"
---
# <a name="tutorial-configure-atlassian-cloud-for-automatic-user-provisioning"></a>Руководство по настройке Atlassian Cloud для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Atlassian Cloud и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в [Atlassian Cloud](https://www.atlassian.com/licensing/cloud). Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Atlassian Cloud
> * Удаление пользователей в Atlassian Cloud, когда им больше не нужен доступ
> * Синхронизация атрибутов пользователей между Azure AD и Atlassian Cloud.
> * Подготовка групп и членство в группах в Atlassian Cloud
> * [Единый вход](./atlassian-cloud-tutorial.md) в Atlassian Cloud (рекомендуется)

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md)
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор).
* [Клиент Atlassian Cloud](https://www.atlassian.com/licensing/cloud)
* Учетная запись пользователя в Atlassian Cloud с разрешениями администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Atlassian Cloud](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-atlassian-cloud-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Atlassian Cloud для поддержки подготовки с помощью Azure AD

1. Перейдите в нужную папку: [Atlassian Organization Manager](https://admin.atlassian.com) **> выберите организацию > каталог**.

    ![Снимок экрана страницы администрирования с вызываемым параметром каталога.](./media/atlassian-cloud-provisioning-tutorial/select-directory.png)

2. Щелкните **Подготовка пользователей**, а затем выберите **Создать каталог**. Скопируйте **Directory base URL** (базовый URL-адрес каталога) и **Маркер носителя**, которые будут введены в полях **URL-адрес клиента** и **Секретный токен** на вкладке "Подготовка" приложения Atlassian Cloud на портале Azure AD, соответственно.

    ![Снимок экрана страницы администрирования с вызываемым параметром подготовки пользователей.](./media/atlassian-cloud-provisioning-tutorial/secret-token-1.png) ![Снимок экрана со страницей создания токена.](./media/atlassian-cloud-provisioning-tutorial/secret-token-2.png)
    ![Снимок экрана со страницей маркера времени демонстрационного каталога.](./media/atlassian-cloud-provisioning-tutorial/secret-token-3.png)


## <a name="step-3-add-atlassian-cloud-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Atlassian Cloud из коллекции приложений Azure AD

Добавьте Atlassian Cloud из коллекции приложений Azure AD, чтобы начать управление подготовкой в Atlassian Cloud. Если вы ранее настроили Atlassian Cloud для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Atlassian Cloud необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configuring-automatic-user-provisioning-to-atlassian-cloud"></a>Шаг 5. Настройка автоматической подготовки пользователей в Atlassian Cloud 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Atlassian Cloud на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-atlassian-cloud-in-azure-ad"></a>Чтобы настроить автоматическую подготовку учетных записей пользователей для Atlassian Cloud в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com), а затем выберите элементы **Корпоративные приложения**, **Все приложения** и **Atlassian Cloud**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Atlassian Cloud**.

    ![Ссылка на Atlassian Cloud в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите **URL-адрес клиента** и **Секретный токен**, полученный ранее из учетной записи Atlassian Cloud. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Atlassian Cloud. Если установить подключение не удалось, убедитесь, что в учетной записи Atlassian Cloud есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Atlassian Cloud**.

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Atlassian Cloud. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Atlassian Cloud при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |active|Логическое|
   |name.familyName|Строка|
   |name.givenName|Строка|
   |emails[type eq "work"].value|Строка|   

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Atlassian Cloud**.

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Atlassian Cloud. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Atlassian Cloud при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |externalId|Строка|
      |members|Справочник|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Atlassian Cloud, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в Atlassian Cloud, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

16. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут.

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="connector-limitations"></a>Ограничения соединителя

* Atlassian Cloud позволяет подготавливать пользователей только из [проверенных доменов](https://confluence.atlassian.com/cloud/organization-administration-938859734.html).
* Atlassian Cloud сейчас не поддерживает групповые переименования. Это означает, что любые изменения displayName группы в Azure AD не будут обновлены и отражены в Atlassian Cloud.
* Значение атрибута пользователя **mail** в Azure AD заполняется только в том случае, если у пользователя есть почтовый ящик Microsoft Exchange. Если у пользователя нет такой учетной записи Exchange, рекомендуется сопоставлять другой подходящий атрибут с атрибутом **emails** в Atlassian Cloud.

## <a name="change-log"></a>Журнал изменений

* 15.06.2020 — добавлена поддержка пакетной операции PATCH для групп.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/atlassian-cloud-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/atlassian-cloud-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/atlassian-cloud-provisioning-tutorial/tutorial-general-03.png