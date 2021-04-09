---
title: Руководство. Настройка Soloinsight-CloudGate SSO для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Soloinsight-CloudGate SSO.
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
ms.openlocfilehash: 5c42102c0704d7212411d6b86f4210ad8ecd885c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96347520"
---
# <a name="tutorial-configure-soloinsight-cloudgate-sso-for-automatic-user-provisioning"></a>Руководство. Настройка Soloinsight-CloudGate SSO для автоматической подготовки пользователей

В этом руководстве описаны шаги, которые нужно выполнить в Soloinsight-CloudGate SSO и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и (или) групп в Azure AD для Soloinsight-CloudGate SSO.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Soloinsight-CloudGate SSO](https://www.soloinsight.com/);
* учетная запись пользователя в Soloinsight-CloudGate SSO с разрешениями администратора.

## <a name="assigning-users-to-soloinsight-cloudgate-sso"></a>Назначение пользователей Soloinsight-CloudGate SSO

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Soloinsight-CloudGate SSO. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Soloinsight-CloudGate SSO, следуя приведенным ниже инструкциям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-soloinsight-cloudgate-sso"></a>Важные советы по назначению пользователей в Soloinsight-CloudGate SSO

* Рекомендуется назначить одного пользователя Azure AD в Soloinsight-CloudGate SSO для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Soloinsight-CloudGate SSO в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-soloinsight-cloudgate-sso-for-provisioning"></a>Настройка Soloinsight-CloudGate SSO для подготовки

1. Войдите в консоль администрирования [Soloinsight-CloudGate SSO](https://soloinsight.sigateway.com/login). Перейдите к разделу **Administration > System Settings** (Администрирование > Параметры системы).

    ![Консоль администрирования приложения Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/admin.png)

2.  Перейдите к разделу **General** (Общие).

    ![Раздел Add SCIM (Добавить SCIM) в Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/config.png)

3.  Прокрутите страницу вниз до конца, чтобы получить **URL-адрес клиента** и **секретный токен**. Скопируйте **секретный токен**. Его нужно будет ввести на вкладке "Подготовка" в поле "Секретный токен" для приложения Soloinsight-CloudGate SSO на портале Azure.

    ![Раздел Create Token (Создание токена) приложения Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/token.png)

## <a name="add-soloinsight-cloudgate-sso-from-the-gallery"></a>Добавление Soloinsight-CloudGate SSO из коллекции

Перед настройкой Soloinsight-CloudGate SSO для автоматической подготовки пользователей в Azure AD необходимо добавить Soloinsight-CloudGate SSO из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Soloinsight-CloudGate SSO из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Soloinsight-CloudGate SSO**, выберите **Soloinsight-CloudGate SSO** в области результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Soloinsight-CloudGate SSO в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-soloinsight-cloudgate-sso"></a>Настройка Soloinsight-CloudGate SSO для автоматической подготовки пользователей 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Soloinsight-CloudGate SSO на основе их назначений в Azure AD.

> [!TIP]
> Для Soloinsight-CloudGate SSO также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [руководстве по единому входу в Soloinsight-CloudGate SSO](./soloinsight-cloudgate-sso-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-soloinsight-cloudgate-sso-in-azure-ad"></a>Чтобы настроить в Azure AD автоматическую подготовку пользователей для Soloinsight-CloudGate SSO, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Soloinsight CloudGate SSO**.

    ![Ссылка на Soloinsight-CloudGate SSO в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://sigateway.com/scim/v2/sync/serviceproviderconfig` в поле **URL-адрес клиента**. Введите полученное ранее значение **токена проверки подлинности SCIM** в поле **Секретный токен**. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Soloinsight-CloudGate SSO. Если установить подключение не удалось, убедитесь, что у учетной записи Soloinsight-CloudGate SSO есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Soloinsight-CloudGate SSO**.

    ![Сопоставление пользователей в Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Soloinsight-CloudGate SSO. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления учетных записей пользователей в Soloinsight-CloudGate SSO для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователя в Soloinsight CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/userattributes.png)

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Soloinsight-CloudGate SSO**.

    ![Сопоставление групп в Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты групп, которые синхронизированы из Azure AD в Soloinsight-CloudGate SSO. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления групп в Soloinsight-CloudGate SSO для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты групп в Soloinsight CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/groupattributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Soloinsight-CloudGate SSO, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей и (или) группы для подготовки в Soloinsight-CloudGate SSO, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Soloinsight-CloudGate SSO.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)