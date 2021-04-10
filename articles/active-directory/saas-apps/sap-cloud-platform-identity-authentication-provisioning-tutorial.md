---
title: Руководство по Настройка SAP Cloud Platform Identity Authentication для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в SAP Cloud Platform Identity Authentication.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/19/2019
ms.author: Zhchia
ms.openlocfilehash: 419f25ee3df471bc2fc4526254f5677b8bd71856
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96342753"
---
# <a name="tutorial-configure-sap-cloud-platform-identity-authentication-for-automatic-user-provisioning"></a>Руководство по Настройка SAP Cloud Platform Identity Authentication для автоматической подготовки пользователей

В этом руководстве описаны действия, которые нужно выполнить в SAP Cloud Platform Identity Authentication и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в AP Cloud Platform Identity Authentication.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент SAP Cloud Platform Identity Authentication](https://cloudplatform.sap.com/pricing.html);
* учетная запись пользователя в SAP Cloud Platform Identity Authentication с разрешениями администратора.

## <a name="assigning-users-to-sap-cloud-platform-identity-authentication"></a>Назначение пользователей в SAP Cloud Platform Identity Authentication

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к SAP Cloud Platform Identity Authentication. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению SAP Cloud Platform Identity Authentication, выполнив следующие инструкции:
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-sap-cloud-platform-identity-authentication"></a>Важные советы по назначению пользователей в SAP Cloud Platform Identity Authentication

* Рекомендуется назначить одного пользователя Azure AD в SAP Cloud Platform Identity Authentication для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в SAP Cloud Platform Identity Authentication в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="setup-sap-cloud-platform-identity-authentication-for-provisioning"></a>Настройка SAP Cloud Platform Identity Authentication для подготовки

1. Войдите в [консоль SAP Cloud Platform Identity Authentication](https://sapmsftintegration.accounts.ondemand.com/admin). Перейдите в раздел **Users & Authorizations > Administrators** (Пользователи и авторизация > Администраторы).

    ![Консоль администрирования SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/adminconsole.png)

2.  Нажмите кнопку **+Add** (+Добавить) на панели слева, чтобы добавить в список нового администратора. Выберите команду **Add System** (Добавить систему) и введите имя системы.   

> [!NOTE]
> Для типа администратора в SAP Cloud Platform Identity Authentication нужно задать значение **System** (Системный). Если создать обычного пользователя с правами администратора, во время подготовки могут возникнуть ошибки, вызванные *отсутствием необходимых прав доступа*.   

3.  В разделе настройки авторизации переведите выключатель из значения **Manage Users** (Управление пользователями) в значение **Manage Groups** (Управление группами).

    ![SAP Cloud Platform Identity Authentication: добавление SCIM](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/configurationauth.png)

4. Вы получите сообщение электронной почты для активации учетной записи и установки пароля для **службы SAP Cloud Platform Identity Authentication**.

4.  Скопируйте **идентификатор пользователя** и **пароль**. Эти значения нужно будет указать в полях для имени и пароля администратора соответственно на вкладке подготовки приложения SAP Cloud Platform Identity Authentication на портале Azure.

## <a name="add-sap-cloud-platform-identity-authentication-from-the-gallery"></a>Добавление приложения SAP Cloud Platform Identity Authentication из коллекции

Перед настройкой SAP Cloud Platform Identity Authentication для автоматической подготовки пользователей в Azure AD необходимо добавить SAP Cloud Platform Identity Authentication из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Добавление SAP Cloud Platform Identity Authentication из коллекции приложений Azure AD:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **SAP Cloud Platform Identity Authentication**, выберите **SAP Cloud Platform Identity Authentication** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Приложения SAP Cloud Platform Identity Authentication в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-sap-cloud-platform-identity-authentication"></a>Настройка автоматической подготовки пользователей в SAP Cloud Platform Identity Authentication 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в SAP Cloud Platform Identity Authentication на основе их назначений в Azure AD.

> [!TIP]
> Для SAP Cloud Platform Identity Authentication также можно включить единый вход на основе SAML. Для этого выполните инструкции, приведенные в [руководстве по настройке единого входа для SAP Cloud Platform Identity Authentication](./sap-hana-cloud-platform-identity-authentication-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-sap-cloud-platform-identity-authentication-in-azure-ad"></a>Настройка автоматической подготовки пользователей для SAP Cloud Platform Identity Authentication в Azure AD:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **SAP Cloud Platform Identity Authentication**.

    ![Ссылка на SAP Cloud Platform Identity Authentication в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://<tenantID>.accounts.ondemand.com/service/scim ` в поле **URL-адрес клиента**. Введите **идентификатор пользователя** и **пароль**, полученные ранее, в поля для **имени администратора** и **пароля администратора** соответственно. Щелкните команду **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к SAP Cloud Platform Identity Authentication. Если установить подключение не удалось, убедитесь, что у учетной записи SAP Cloud Platform Identity Authentication есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/testconnection.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите действие **Синхронизировать пользователей Azure Active Directory в SAP Cloud Platform Identity Authentication**.

    ![Сопоставление пользователей в SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/mapping.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователя, синхронизированные из Azure AD в SAP Cloud Platform Identity Authentication. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в SAP Cloud Platform Identity Authentication для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователей в SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/userattributes.png)

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для SAP Cloud Platform Identity Authentication, в разделе **Параметры** установите переключатель **Состояние подготовки** в положение **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в SAP Cloud Platform Identity Authentication, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением SAP Cloud Platform Identity Authentication.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* Конечной точке SCIM для SAP Cloud Platform Identity Authentication требуются определенные атрибуты в определенном формате. Дополнительные сведения об этих атрибутах и их специальном формате можно найти [здесь](https://help.sap.com/viewer/6d6d63354d1242d185ab4830fc04feb1/Cloud/en-US/b10fc6a9a37c488a82ce7489b1fab64c.html#).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)