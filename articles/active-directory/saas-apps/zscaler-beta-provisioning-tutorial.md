---
title: Руководство по настройке Zscaler (бета-версия) для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Zscaler (бета-версия).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 0d4945ee97a46c78aac3c4ac508c5f89f5942296
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97937167"
---
# <a name="tutorial-configure-zscaler-beta-for-automatic-user-provisioning"></a>Руководство по настройке Zscaler (бета-версия) для автоматической подготовки пользователей

В этом руководстве описаны действия, которые нужно выполнить в Zscaler (бета-версия) и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Zscaler (бета-версия).

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>


## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом руководстве, предполагает, что у вас уже имеется:

* клиент Azure AD;
* клиент Zscaler (бета-версия);
* учетная запись пользователя в Zscaler (бета-версия) с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от API SCIM Zscaler (бета-версия). Этот API доступен для разработчиков Zscaler (бета-версия), использующих учетные записи с пакетом Enterprise.

## <a name="adding-zscaler-beta-from-the-gallery"></a>Добавление Zscaler Beta из коллекции

Перед настройкой Zscaler (бета-версия) для автоматической подготовки пользователей в Azure AD необходимо добавить Zscaler (бета-версия) из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Zscaler (бета-версия) из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Zscaler Beta**, выберите **Zscaler Beta** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Zscaler Beta в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-zscaler-beta"></a>Назначение пользователей в Zscaler (бета-версия)

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая "назначение". В контексте автоматической подготовки синхронизированы будут только те пользователи и группы, которые были "назначены" приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, каким пользователям или группам в Azure AD нужно предоставить доступ к Zscaler (бета-версия). Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Zscaler (бета-версия), следуя инструкциям:

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-zscaler-beta"></a>Важные рекомендации по назначению пользователей в Zscaler (бета-версия)

* Рекомендуется назначить одного пользователя Azure AD в Zscaler (бета-версия) для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Zscaler (бета-версия) в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если это доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-zscaler-beta"></a>Настройка автоматической подготовки пользователей в Zscaler (бета-версия)

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Zscaler (бета-версия) на основе их назначений в Azure AD.

> [!TIP]
> Для Zscaler (бета-версия) также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [руководстве по единому входу в Zscaler (бета-версия)](zscaler-beta-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

> [!NOTE]
> При подготовке или отмене подготовки пользователей и групп рекомендуется периодически перезапускать подготовку, чтобы членство в группах правильно обновлялось. При перезапуске служба повторно оценит все группы и обновит членство.  

### <a name="to-configure-automatic-user-provisioning-for-zscaler-beta-in-azure-ad"></a>Настройка автоматической подготовки пользователей для Zscaler (бета-версия) в Azure AD

1. Войдите на [портал Azure](https://portal.azure.com), а затем выберите элементы **Корпоративные приложения**, **Все приложения** и **Zscaler (бета-версия)** .

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Zscaler Beta**.

    ![Ссылка на Zscaler Beta в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![На ней есть список вкладок, упорядоченных по категориям, с названием Zscaler Beta - Provisioning / Enterprise Application (Zscaler (бета-версия) — подготовка / Корпоративное приложение). Выбрана вкладка Provision (Подготовка) в категории Manage (Управление).](./media/zscaler-beta-provisioning-tutorial/provisioning-tab.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![В раскрывающемся списке Provisioning Mode (Режим подготовки) выбран режим Automatic (Автоматически). Здесь есть поля для учетных данных администратора (Admin Credentials), которые используются для подключения к API Zscaler (бета-версия), а также кнопка Test Connection (Проверить подключение).](./media/zscaler-beta-provisioning-tutorial/provisioning-credentials.png)

5. В разделе **Admin Credentials** (Учетные данные администратора) заполните поля **Tenant URL** (URL-адрес клиента) и **Secret Token** (Секретный токен) учетной записи Zscaler (бета-версия), как описано на шаге 6.

6. Чтобы получить **URL-адрес клиента** и **секретный токен**, перейдите в раздел **Administration > Authentication Settings** (Администрирование > Параметры проверки подлинности) в пользовательском интерфейсе портала Zscaler (бета-версия) и выберите **SAML** в разделе **Authentication Type** (Тип проверки подлинности).

    ![На экране Authentication Settings (Параметры проверки подлинности) в разделе Authentication Profile (Профиль проверки подлинности) для параметра Directory Type (Тип каталога) выбрано значение Hosted DB, а для параметра Authentication Type (Тип проверки подлинности) — SAML.](./media/zscaler-beta-provisioning-tutorial/secret-token-1.png)

    Выберите **Configure SAML** (Настроить SAML), чтобы открыть параметры **настройки SAML**.

    ![На экране Configure SAML (Настройка SAML) выбраны параметры Enable SAML Auto-Provisioning (Включить автоматическую подготовку SAML) и Enable SCIM-Based Provisioning options (Включить параметры подготовки на основе SCIM). Текстовые поля Base URL (Базовый URL-адрес) и Bearer Token (Токен носителя) выделены.](./media/zscaler-beta-provisioning-tutorial/secret-token-2.png)

    Выберите **Enable SCIM-Based Provisioning** (Включить подготовку на основе SCIM), чтобы получить **базовый URL-адрес** и **токен носителя**, а затем сохраните настройки. Скопируйте **базовый URL-адрес** в поле **URL-адрес клиента**, а **токен носителя** — в поле **Секретный токен** на портале Azure.

7. После заполнения полей, указанных на шаге 5, щелкните элемент **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Zscaler (бета-версия). Если установить подключение не удалось, убедитесь, что у учетной записи Zscaler (бета-версия) есть разрешения администратора, и повторите попытку.

    ![На экране Admin Credentials (Учетные данные администратора) в полях Tenant URL (URL-адрес клиента) и Secret Token (Секретный токен) указаны значения, а кнопка Test Connection (Проверить подключение) выделена.](./media/zscaler-beta-provisioning-tutorial/test-connection.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Текстовое поле Notification Email (Уведомление по электронной почте) пустое, а флажок Send an email notification when a failure occurs (Отправлять уведомление по электронной почте при сбое) снят.](./media/zscaler-beta-provisioning-tutorial/notification.png)

9. Нажмите кнопку **Сохранить**.

10. В разделе **Mappings** (Сопоставления) выберите **Synchronize Azure Active Directory Users to Zscaler Beta** (Синхронизировать пользователей Azure Active Directory в Zscaler (бета-версия)).

    ![Параметр Synchronize Azure Active Directory Users to Zscaler Beta (Синхронизировать пользователей Azure Active Directory в Zscaler (бета-версия)) выбран и включен.](./media/zscaler-beta-provisioning-tutorial/user-mappings.png)

11. В разделе **Attribute Mapping** (Сопоставления атрибутов) просмотрите атрибуты пользователей, синхронизированные в Azure AD и Zscaler (бета-версия). Атрибуты, выбранные как **совпадающие** свойства, используются для сопоставления учетных записей пользователей в Zscaler (бета-версия) для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![В разделе Attribute Mappings (Сопоставления атрибутов) для атрибутов пользователей атрибуты Active Directory отображаются рядом с атрибутами Zscaler (бета-версия), с которыми они синхронизируются. Одна пара атрибутов отображается как Matching (Совпадающие).](./media/zscaler-beta-provisioning-tutorial/user-attribute-mappings.png)

12. В разделе **Mappings** (Сопоставления) выберите **Synchronize Azure Active Directory Groups to Zscaler Beta** (Синхронизировать группы Azure Active Directory в Zscaler (бета-версия)).

    ![Параметр Synchronize Azure Active Directory Groups to Zscaler Beta (Синхронизировать группы Azure Active Directory в Zscaler (бета-версия)) выбран и включен.](./media/zscaler-beta-provisioning-tutorial/group-mappings.png)

13. В разделе **Attribute Mappings** (Сопоставления атрибутов) просмотрите атрибуты групп, синхронизированные из Azure AD в Zscaler (бета-версия). Атрибуты, выбранные как **совпадающие** свойства, используются для сопоставления групп в Zscaler (бета-версия) для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![В разделе Attribute Mappings (Сопоставления атрибутов) для атрибутов групп атрибуты Active Directory отображаются рядом с атрибутами Zscaler (бета-версия), с которыми они синхронизируются. Одна пара атрибутов отображается как Matching (Совпадающие).](./media/zscaler-beta-provisioning-tutorial/group-attribute-mappings.png)

14. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Чтобы включить службу подготовки Azure AD для Zscaler (бета-версия), измените значение параметра **Provisioning Status** (Состояние подготовки) в разделе **Settings** (Параметры) на **On** (Включено).

    ![Для параметра Provisioning Status (Состояние подготовки) выбрано значение On (Вкл.).](./media/zscaler-beta-provisioning-tutorial/provisioning-status.png)

16. Определите пользователей и (или) группы для подготовки в Zscaler (бета-версия), выбрав нужные значения в поле **Scope** (Область) в разделе **Settings** (Параметры).

    ![Показан раскрывающийся список Scope (Область) и выбран параметр Sync only assigned users and groups (Синхронизация только назначенных пользователей и групп). Другое доступное значение — Sync all users and groups (Синхронизация всех пользователей и групп).](./media/zscaler-beta-provisioning-tutorial/scoping.png)

17. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Кнопка Save (Сохранить) в верхней части страницы Zscaler Beta - Provisioning (Zscaler (бета-версия) — подготовка) выделена. Также есть кнопка Discard (Отменить).](./media/zscaler-beta-provisioning-tutorial/save-provisioning.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **сведений о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Zscaler (бета-версия).

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zscaler-beta-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-beta-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-beta-provisioning-tutorial/tutorial-general-03.png