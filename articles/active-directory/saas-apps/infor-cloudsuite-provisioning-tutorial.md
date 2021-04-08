---
title: Руководство по Настройка Infor CloudSuite для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в Infor CloudSuite.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: Zhchia
ms.openlocfilehash: 8fdd2c8a326fbdc68d1aec65377f4c465c5ee4c1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96353907"
---
# <a name="tutorial-configure-infor-cloudsuite-for-automatic-user-provisioning"></a>Руководство по Настройка Infor CloudSuite для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Infor CloudSuite и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Infor CloudSuite.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [арендатор Infor CloudSuite;](https://www.infor.com/products/infor-os)
* учетная запись пользователя Infor CloudSuite с разрешениями администратора.

## <a name="assigning-users-to-infor-cloudsuite"></a>Назначение пользователей в Infor CloudSuite

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, каким пользователям и (или) группам в Azure AD нужно предоставить доступ к Infor CloudSuite. Затем назначьте этих пользователей и (или) группы приложению Infor CloudSuite, выполнив следующие инструкции:
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-infor-cloudsuite"></a>Важные рекомендации по назначению пользователей в Infor CloudSuite

* Рекомендуется назначить одного пользователя Azure AD в Infor CloudSuite для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Infor CloudSuite в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-infor-cloudsuite-for-provisioning"></a>Настройка Infor CloudSuite для подготовки

1. Войдите в [консоль администрирования Infor CloudSuite](https://www.infor.com/customer-center). Щелкните значок пользователя, а затем перейдите к разделу **управления пользователями**.

    ![Консоль администрирования Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/admin.png)

2.  Щелкните значок меню в верхнем левом углу экрана. Щелкните элемент **Manage** (Управление).

    ![Infor CloudSuite: добавление SCIM](media/infor-cloudsuite-provisioning-tutorial/manage.png)

3.  Перейдите к разделу **учетных записей SCIM**.

    ![Учетная запись SCIM Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/scim.png)

4.  Добавьте администратора, щелкнув значок плюса. Укажите **пароль SCIM** и введите тот же пароль в разделе **подтверждения пароля**. Щелкните значок папки, чтобы сохранить пароль. После этого отобразится **идентификатор пользователя**, созданный для администратора.

    ![Администратор Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/newuser.png)
    
    ![Пароль Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/password.png)

    :::image type="content" source="media/infor-cloudsuite-provisioning-tutorial/identifier.png" alt-text="Снимок экрана: консоль администрирования Infor CloudSuite с выделенной строкой таблицы. Эта строка содержит идентификатор пользователя, пароли и метку времени." border="false":::

5. Чтобы создать токен носителя, скопируйте **идентификатор пользователя** и **пароль SCIM**. Вставьте их в Notepad++, разделив двоеточием. Закодируйте строковое значение. Для этого перейдите к разделу **Plugins > MIME Tools > Base64 Encode** (Подключаемые модули > Средства MIME > Кодирование Base64). 

    :::image type="content" source="media/infor-cloudsuite-provisioning-tutorial/token.png" alt-text="Снимок экрана: документ Notepad++. В меню Plugins (Подключаемые модули) выделен раздел MIME Tools (Средства MIME). В меню MIME Tools (Средства MIME) выделен раздел Base64 Encode (Кодирование Base64)." border="false":::

3.  Скопируйте значение токена носителя. Его нужно будет ввести на вкладке "Подготовка" в поле "Токен секрета" для приложения Infor CloudSuite на портале Azure.

## <a name="add-infor-cloudsuite-from-the-gallery"></a>Добавление Infor CloudSuite из коллекции

Перед настройкой Infor CloudSuite для автоматической подготовки пользователей с помощью Azure AD необходимо добавить Infor CloudSuite из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Infor CloudSuite из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Infor CloudSuite**, выберите **Infor CloudSuite** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Infor CloudSuite в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-infor-cloudsuite"></a>Настройка автоматической подготовки пользователей в Infor CloudSuite 

В этом разделе описана настройка службы подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Infor CloudSuite на основе их назначений в Azure AD.

> [!TIP]
> Для Infor CloudSuite также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [учебнике по единому входу в Infor CloudSuite](./infor-cloud-suite-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

> [!NOTE]
> Дополнительные сведения о конечной точке SCIM Infor CloudSuite см. на [этой странице](https://docs.infor.com/mingle/12.0.x/en-us/minceolh/jho1449382121585.html#).

### <a name="to-configure-automatic-user-provisioning-for-infor-cloudsuite-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для Infor CloudSuite сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Infor CloudSuite**.

    ![Ссылка на Infor CloudSuite в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://mingle-t20b-scim.mingle.awsdev.infor.com/INFORSTS_TST/v2/scim` в поле **URL-адрес клиента**. Введите полученный ранее токен носителя в поле **Токен секрета**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Infor CloudSuite. Если установить подключение не удалось, убедитесь, что у учетной записи Infor CloudSuite есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Infor CloudSuite**.

    ![Сопоставление пользователей в Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Infor CloudSuite. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в Infor CloudSuite при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователей в Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/userattributes.png)

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Infor CloudSuite**.

    ![Сопоставление групп в Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Infor CloudSuite. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления групп в Infor CloudSuite при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты групп в Infor CloudSuite](media/infor-cloudsuite-provisioning-tutorial/groupattributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Infor CloudSuite, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей и (или) группы для подготовки в Infor CloudSuite, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Infor CloudSuite.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)