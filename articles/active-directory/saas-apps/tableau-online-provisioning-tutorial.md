---
title: Руководство. Настройка Tableau Online для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Tableau Online.
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
ms.openlocfilehash: a42790e079985b003776b381c74f837b0ba619b1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94359210"
---
# <a name="tutorial-configure-tableau-online-for-automatic-user-provisioning"></a>Руководство. Настройка Tableau Online для автоматической подготовки пользователей

В этом учебнике показаны шаги, которые необходимо выполнить в Tableau Online и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей или групп в Tableau Online.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на основе службы подготовки пользователей Azure AD. Сведения о том, что делает эта служба, как она работает, а также часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и их отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом руководстве, предполагает, что у вас уже имеется:

*   Клиент Azure AD.
*   [клиент Tableau Online](https://www.tableau.com/);
*   учетная запись пользователя Tableau Online с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от [Rest API Tableau Online](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm). Этот API доступен для разработчиков Tableau Online.

## <a name="add-tableau-online-from-the-azure-marketplace"></a>Добавление Tableau Online из Azure Marketplace
Перед настройкой Tableau Online для автоматической подготовки пользователей в Azure AD добавьте Tableau Online из Azure Marketplace в список управляемых приложений SaaS.

Чтобы добавить Tableau Online из Azure Marketplace, сделайте следующее:

1. На [портале Azure](https://portal.azure.com) в области навигации слева выберите **Azure Active Directory**.

    ![Значок Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна выберите **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Tableau Online** и выберите **Tableau Online** на панели результатов. Нажмите **Добавить**, чтобы добавить приложение.

    ![Tableau Online в списке результатов](common/search-new-app.png)

## <a name="assign-users-to-tableau-online"></a>Назначение пользователей в приложении Tableau Online

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и группы, которые были назначены приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей решите, какие пользователи или группы в Azure AD должны иметь доступ к Tableau Online. Чтобы назначить этих пользователей или группы для Tableau Online, следуйте инструкциям в разделе[Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-tableau-online"></a>Важные советы по назначению пользователей в приложении Tableau Online

*   Мы рекомендуем назначить одного пользователя Azure AD для Tableau Online для тестирования конфигурации автоматической подготовки пользователей. Дополнительных пользователей или группы можно будет назначить позже.

*   При назначении пользователя в Tableau Online в диалоговом окне назначения выберите действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configure-automatic-user-provisioning-to-tableau-online"></a>Настройка автоматической подготовки пользователей в Tableau Online

В этом разделе описывается процедура настройки службы подготовки Azure AD. Используйте ее для создания, обновления и отключения пользователей или групп в Tableau Online в соответствии с назначениями пользователей или групп в Azure AD.

> [!TIP]
> Для Tableau Online также можно включить единый вход на основе SAML. Выполните инструкции из учебника по [настройке единого входа в Tableau Online](tableauonline-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="configure-automatic-user-provisioning-for-tableau-online-in-azure-ad"></a>Настройка автоматической подготовки пользователей для Tableau Online в Azure AD

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения** > **Все приложения** > **Tableau Online**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Tableau Online**.

    ![Ссылка на Tableau Online в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Подготовка Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningTab.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Режим подготовки Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningCredentials.png)

5. В разделе **Учетные данные администратора** введите домен, имя администратора, пароль администратора и URL-адрес содержимого учетной записи Tableau Online.

   * В поле **Домен** укажите поддомен на основе шага 6.

   * В поле **Имя администратора** укажите имя пользователя учетной записи администратора в клиенте Tableau Online. Например, admin@contoso.com.

   * В поле **Пароль администратора** введите пароль учетной записи администратора, соответствующий имени администратора.

   * В поле **URL-адрес содержимого** укажите поддомен на основе шага 6.

6. После входа в учетную запись администратора Tableau Online можно извлечь значения для полей **Домен** и **URL-адрес содержимого** из URL-адреса страницы администратора.

    * Скопировать **домен** для учетной записи Tableau Online можно из этой части URL-адреса:

        ![Домен Tableau Online](./media/tableau-online-provisioning-tutorial/DomainUrlPart.png)

    * Значение поля **URL-адрес содержимого** для учетной записи Tableau Online можно скопировать из этого раздела. Это значение определяется во время настройки учетной записи. В этом примере используется значение "contoso":

        ![URL-адрес содержимого Tableau Online](./media/tableau-online-provisioning-tutorial/ContentUrlPart.png)

        > [!NOTE]
        > Ваш **домен** может отличаться от приведенного здесь.

7. После заполнения полей, показанных на шаге 5, выберите **Проверить подключение**, чтобы убедиться, что Azure AD может подключаться к Tableau Online. Если установить подключение не удалось, убедитесь, что у учетной записи Tableau Online есть разрешения администратора, и повторите попытку.

    ![Проверить подключение в Tableau Online](./media/tableau-online-provisioning-tutorial/TestConnection.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление Tableau Online](./media/tableau-online-provisioning-tutorial/EmailNotification.png)

9. Нажмите кнопку **Сохранить**.

10. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Tableau Online**.

    ![Синхронизация пользователей Tableau Online](./media/tableau-online-provisioning-tutorial/UserMappings.png)

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты пользователей, синхронизированные из Azure AD в Tableau Online. Атрибуты, выбранные как свойства **Matching**, используются для сопоставления учетных записей пользователей в Tableau Online для операций обновления. Нажмите **Сохранить**, чтобы сохранить все изменения.

    ![Сопоставляемые атрибуты пользователей Tableau Online](./media/tableau-online-provisioning-tutorial/attribute.png)

12. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Tableau Online**.

    ![Синхронизация групп Tableau Online](./media/tableau-online-provisioning-tutorial/GroupMappings.png)

13. В разделе **Сопоставление атрибутов** просмотрите атрибуты групп, синхронизированных из Azure AD в Tableau Online. Атрибуты, выбранные как свойства **Matching**, используются для сопоставления учетных записей пользователей в Tableau Online для операций обновления. Нажмите **Сохранить**, чтобы сохранить все изменения.

    ![Сопоставляемые атрибуты групп Tableau Online](./media/tableau-online-provisioning-tutorial/GroupAttributeMapping.png)

14. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Чтобы включить службу подготовки Azure AD для Tableau Online, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningStatus.png)

16. Определите пользователей или группы, которые требуется подготовить для Tableau Online. В разделе **Параметры** выберите нужные значения в разделе **Область**.

    ![Область Tableau Online](./media/tableau-online-provisioning-tutorial/ScopeSync.png)

17. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Кнопка Save (Сохранить) в Tableau Online](./media/tableau-online-provisioning-tutorial/SaveProvisioning.png)

После этого начнется начальная синхронизация пользователей или групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Они происходят примерно каждые 40 минут, пока выполняется служба подготовки Azure AD. 

В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчета о подготовке. В отчете представлены сведения обо всех действиях, выполненных службой подготовки Azure AD в отношении Tableau Online.

Сведения о журналах подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="change-log"></a>Журнал изменений
* 30.09.2020 — добавлена поддержка атрибута "authSetting" для пользователей.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Managing user account provisioning for enterprise apps in the Azure portal](../app-provisioning/configure-automatic-user-provisioning-portal.md) (Управление подготовкой учетных записей пользователей для корпоративных приложений на портале Azure)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/tableau-online-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/tableau-online-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/tableau-online-provisioning-tutorial/tutorial_general_03.png