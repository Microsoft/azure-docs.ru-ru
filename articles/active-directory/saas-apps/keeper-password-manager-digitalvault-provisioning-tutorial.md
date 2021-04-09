---
title: Руководство по Настройка автоматической подготовки пользователей в Keeper Password Manager & Digital Vault с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Keeper Password Manager & Digital Vault.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2019
ms.author: jeedes
ms.openlocfilehash: 2670dc0cb56805a2afa966bee1d2aa52b6c8e46a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94358989"
---
# <a name="tutorial-configure-keeper-password-manager--digital-vault-for-automatic-user-provisioning"></a>Руководство по Настройка автоматической подготовки пользователей в Keeper Password Manager & Digital Vault

В этом учебнике описаны шаги, которые нужно выполнить в Keeper Password Manager & Digital Vault и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и групп в AAD для Keeper Password Manager & Digital Vault.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Keeper Password Manager & Digital Vault](https://keepersecurity.com/pricing.html?t=e);
* учетная запись пользователя Keeper Password Manager & Digital Vault с разрешениями администратора.

## <a name="add-keeper-password-manager--digital-vault-from-the-gallery"></a>Добавление Keeper Password Manager & Digital Vault из коллекции

Прежде чем настроить в Keeper Password Manager & Digital Vault автоматическую подготовку пользователей в Azure AD, нужно добавить Keeper Password Manager & Digital Vault из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Keeper Password Manager & Digital Vault из коллекции приложений Azure AD, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Keeper Password Manager & Digital Vault**, выберите **Keeper Password Manager & Digital Vault** на панели результатов, а затем нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Keeper Password Manager & Digital Vault в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-keeper-password-manager--digital-vault"></a>Назначение пользователей для Keeper Password Manager & Digital Vault

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и группы в Azure AD должны иметь доступ к Keeper Password Manager & Digital Vault. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Keeper Password Manager & Digital Vault с помощью следующих действий:

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-keeper-password-manager--digital-vault"></a>Важные замечания о назначении пользователей для Keeper Password Manager & Digital Vault

* Рекомендуется назначить одного пользователя Azure AD для Keeper Password Manager & Digital Vault, чтобы проверить конфигурацию автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Keeper Password Manager & Digital Vault в диалоговом окне назначения нужно выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-keeper-password-manager--digital-vault"></a>Настройка автоматической подготовки пользователей в Keeper Password Manager & Digital Vault 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Keeper Password Manager & Digital Vault на основе их назначений в Azure AD.

> [!TIP]
> Вы также можете настроить для Keeper Password Manager & Digital Vault единый вход на основе SAML, выполнив инструкции из [руководства по единому входу для Keeper Password Manager & Digital Vault](keeperpasswordmanager-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-keeper-password-manager--digital-vault-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для Keeper Password Manager & Digital Vault выполните следующие действия:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Keeper Password Manager & Digital Vault**.

    ![Keeper Password Manager & Digital Vault в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите **URL-адрес арендатора** и **секретный токен** учетной записи Keeper Password Manager & Digital Vault, как описано на шаге 6.

6. Войдите в [консоль администрирования Keeper Password Manager & Digital Vault](https://keepersecurity.com/console/#login). Щелкните **Admin** (Администратор), затем выберите существующий узел или создайте новый. Перейдите на вкладку **Provisioning** (Подготовка) и щелкните **Add Method** (Добавить метод).

    ![Консоль администрирования Keeper Password Manager & Digital Vault](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-admin-console.png)

    Выберите **SCIM (System for Cross-Domain Identity Management)** .

    ![Добавление SCIM в Keeper Password Manager & Digital Vault](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-add-scim.png)

    Щелкните **Create Provisioning Token** (Создать маркер подготовки).

    ![Создание конечной точки для Keeper Password Manager & Digital Vault](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-create-endpoint.png)

    Скопируйте значения **URL** (URL-адрес) и **Token** (Токен), затем вставьте их в поля **URL-адрес арендатора** и **Секретный токен** в Azure AD. Щелкните **Save** (Сохранить), чтобы завершить настройку подготовки в Keeper.

    ![Создание токена в Keeper Password Manager & Digital Vault](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-create-token.png)

7. Завершив заполнение полей, которые указаны на шаге 5, щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Keeper Password Manager & Digital Vault. Если установить подключение не удалось, убедитесь, что у учетной записи Keeper Password Manager & Digital Vault есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

9. Выберите команду **Сохранить**.

10. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Keeper Password Manager & Digital Vault**.

    ![Сопоставления пользователей в Keeper Password Manager & Digital Vault](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-user-mappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Keeper Password Manager & Digital Vault. Атрибуты, выбранные как свойства с меткой **Соответствие**, используются для сопоставления учетных записей пользователей в Keeper Password Manager & Digital Vault при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователя Keeper](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-user-attributes.png)

12. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Keeper Password Manager & Digital Vault**.

    ![Сопоставления групп в Keeper Password Manager & Digital Vault](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-group-mappings.png)

13. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Keeper Password Manager & Digital Vault. Атрибуты, выбранные как свойства с меткой **Соответствие**, используются для сопоставления групп в Keeper Password Manager & Digital Vault при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты групп в Keeper](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-group-attributes.png)

14. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Чтобы включить службу подготовки Azure AD для Keeper Password Manager & Digital Vault, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

16. Укажите пользователей и (или) группы для подготовки в Keeper Password Manager & Digital Vault, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

17. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Keeper Password Manager & Digital Vault.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* Для Keeper Password Manager & Digital Vault важно, чтобы атрибуты **emails** (адреса электронной почты) и **userName** (имя пользователя) имели одинаковые значения в источнике данных, так как любое изменение одного из этих атрибутов автоматически изменяет значение второго.
* Keeper Password Manager & Digital Vault не поддерживает удаление пользователей, а только их отключение. Отключенные пользователи отображаются как заблокированные в пользовательском интерфейсе консоли администрирования Keeper Password Manager & Digital Vault.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

