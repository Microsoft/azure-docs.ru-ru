---
title: Руководство по настройке OfficeSpace Software для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в OfficeSpace Software.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/02/2019
ms.author: Zhchia
ms.openlocfilehash: fc67d649e3a7cd094eb2c3d633244077abcab308
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94359935"
---
# <a name="tutorial-configure-officespace-software-for-automatic-user-provisioning"></a>Руководство по настройке OfficeSpace Software для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые необходимо выполнить в OfficeSpace Software и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в OfficeSpace Software.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент OfficeSpace Software](https://www.officespacesoftware.com/);
* учетная запись пользователя в OfficeSpace Software с разрешениями администратора.

## <a name="assigning-users-to-officespace-software"></a>Назначение пользователей OfficeSpace Software

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей необходимо решить, какие пользователи или группы в Azure AD будут иметь доступ к OfficeSpace Software. После этого пользователей и (или) группы можно будет назначить OfficeSpace Software, следуя приведенным ниже указаниям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-officespace-software"></a>Важные рекомендации по назначению пользователей OfficeSpace Software

* Рекомендуется назначить одного пользователя Azure AD платформе OfficeSpace Software для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя OfficeSpace Software в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-officespace-software-for-provisioning"></a>Настройка OfficeSpace Software для подготовки

1. Войдите в [консоль OfficeSpace Software](https://support.officespacesoftware.com/hc). Выберите **Settings > Connectors** (Параметры > Соединители).

    ![Консоль OfficeSpace Software](media/officespace-software-provisioning-tutorial/settings.png)

2.  Последовательно выберите **Directory Synchronization > SCIM** (Синхронизация каталогов > SCIM).

    ![Добавление SCIM в OfficeSpace Software](media/officespace-software-provisioning-tutorial/scim.png)

3.  Скопируйте **токен проверки подлинности SCIM**. Его нужно будет ввести в поле "Секретный токен" на вкладке "Подготовка" для приложения OfficeSpace Software на портале Azure.

    ![Создание токена OfficeSpace Software](media/officespace-software-provisioning-tutorial/token.png)

## <a name="add-officespace-software-from-the-gallery"></a>Добавление OfficeSpace Software из коллекции

Перед настройкой OfficeSpace Software для автоматической подготовки пользователей в Azure AD необходимо добавить OfficeSpace Software из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить OfficeSpace Software из коллекции приложений Azure AD, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **OfficeSpace Software**, выберите **OfficeSpace Software** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![OfficeSpace Software в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-officespace-software"></a>Настройка автоматической подготовки пользователей в OfficeSpace Software 

В этом разделе описано, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в OfficeSpace Software на основе их назначений в Azure AD.

> [!TIP]
> Вы также можете настроить для OfficeSpace Software единый вход на основе SAML, следуя инструкциям из [учебника по единому входу в OfficeSpace Software](./officespace-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-officespace-software-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей OfficeSpace Software в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **OfficeSpace Software**.

    ![Ссылка на OfficeSpace Software в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** в поле **URL-адрес клиента** введите URL-адрес в формате `https://<subdomain>.officespacesoftware.com/api/scim/v2/`. Например, `https://contoso.officespacesoftware.com/api/scim/v2/`. Введите полученное ранее значение **токена проверки подлинности SCIM** в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к OfficeSpace Software. Если установить подключение не удалось, убедитесь, что у учетной записи OfficeSpace Software есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с OfficeSpace Software**.

    ![Сопоставления пользователей OfficeSpace Software](media/officespace-software-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в OfficeSpace Software. Атрибуты, выбранные как свойства в разделе **Сопоставление**, используются для сопоставления учетных записей пользователей в OfficeSpace Software для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователя OfficeSpace Software](media/officespace-software-provisioning-tutorial/userattributes.png)

11. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Чтобы включить службу подготовки Azure AD для OfficeSpace Software, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

13. Определите пользователей или группы для подготовки в OfficeSpace Software, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

14. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением OfficeSpace Software.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)