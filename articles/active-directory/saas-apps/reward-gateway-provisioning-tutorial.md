---
title: Учебник по настройке Reward Gateway для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в Reward Gateway.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: 2d51903aff6f3fd1cd53d85a980f1b5dc2a893e9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94354331"
---
# <a name="tutorial-configure-reward-gateway-for-automatic-user-provisioning"></a>Учебник по настройке Reward Gateway для автоматической подготовки пользователей

В этом учебнике показаны шаги, которые необходимо выполнить в Reward Gateway и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Reward Gateway.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [Клиент Reward Gateway](https://www.rewardgateway.com/).
* Учетная запись пользователя в Reward Gateway с разрешениями администратора.

## <a name="assigning-users-to-reward-gateway"></a>Назначение пользователей в Reward Gateway 

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Reward Gateway. После этого можно назначить этих пользователей и (или) группы в Reward Gateway, следуя инструкциям из статьи о [назначении пользователей или групп корпоративному приложению](../manage-apps/assign-user-or-group-access-portal.md).


## <a name="important-tips-for-assigning-users-to-reward-gateway"></a>Важные рекомендации по назначению пользователей в Reward Gateway 

* Рекомендуется назначить одного пользователя Azure AD в Reward Gateway для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Reward Gateway в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (при доступности). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="setup-reward-gateway--for-provisioning"></a>Настройка Reward Gateway для подготовки
Перед настройкой Reward Gateway для автоматической подготовки пользователей с помощью Azure AD необходимо включить подготовку SCIM в Reward Gateway.

1. Войдите в [консоль администрирования Reward Gateway](https://rewardgateway.photoshelter.com/login/). Щелкните **Integrations**(Интеграция).

    ![Снимок экрана консоли администрирования Reward Gateway с выделенным параметром Integrations (Интеграции).](media/reward-gateway-provisioning-tutorial/image00.png)

2.  Выберите **My Integrations** (Мои интеграции).

    ![Снимок экрана с двумя параметрами интеграции и выделенным параметром My Integrations (Мои интеграции).](media/reward-gateway-provisioning-tutorial/image001.png)

3.  Скопируйте значения **SCIM URL (v2)** (URL-адрес SCIM (версии 2)) и **OAuth Bearer Token** (Токен носителя OAuth). Эти значения будут указаны в полях "URL-адрес клиента" и "Секретный токен" на вкладке "Подготовка" приложения Reward Gateway на портале Azure.

    ![Снимок экрана панели My Integrations (Мои интеграции) с выделенным текстовым полем OAuth Bearer Token (Токен носителя OAuth).](media/reward-gateway-provisioning-tutorial/image03.png)

## <a name="add-reward-gateway-from-the-gallery"></a>Добавление Reward Gateway из коллекции

Чтобы настроить автоматическую подготовку пользователей для Reward Gateway в Azure AD, необходимо добавить Reward Gateway из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Reward Gateway из коллекции приложений Azure AD, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Reward Gateway**, выберите **Reward Gateway** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Reward Gateway в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-reward-gateway"></a>Настройка автоматической подготовки пользователей в Reward Gateway  

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Reward Gateway на основе их назначений в Azure AD.

> [!TIP]
> Для Reward Gateway также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [учебнике по единому входу в Reward Gateway](reward-gateway-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-reward-gateway-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Reward Gateway в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Reward Gateway**.

    ![Ссылка на Reward Gateway в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите полученные ранее значения **SCIM URL (v2)** (URL-адрес SCIM (версии 2)) и **OAuth Bearer Token** (Токен носителя OAuth) в поля **URL-адрес клиента** и **Секретный токен** соответственно. Щелкните элемент **Проверить подключение** и убедитесь, что Azure AD может подключиться к Reward Gateway. Если установить подключение не удалось, убедитесь, что в учетной записи Reward Gateway есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Reward Gateway** (Синхронизировать пользователей Azure Active Directory с Reward Gateway).

    ![Снимок экрана: раздел "Сопоставления" с выделенным параметром Synchronize Azure Active Directory Users to Workgrid (Синхронизировать пользователей Azure Active Directory с Reward Gateway).](media/reward-gateway-provisioning-tutorial/user-mappings.png)

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, синхронизированные из Azure AD в Reward Gateway. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Reward Gateway для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Снимок экрана: раздел "Сопоставление атрибутов" с шестью сопоставляемыми атрибутами.](media/reward-gateway-provisioning-tutorial/user-attributes.png)

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Reward Gateway, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей или группы для подготовки в Reward Gateway, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Reward Gateway.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

Сейчас Reward Gateway не поддерживает подготовку групп.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

[Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)