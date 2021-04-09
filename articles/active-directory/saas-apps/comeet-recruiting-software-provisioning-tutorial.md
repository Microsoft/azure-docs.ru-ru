---
title: Руководство по настройке Comeet Recruiting Software для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Comeet Recruiting Software.
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
ms.openlocfilehash: 288d1e6cec8ddcf7d4afe5a35f28a022c2a7be10
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96006718"
---
# <a name="tutorial-configure-comeet-recruiting-software-for-automatic-user-provisioning"></a>Руководство по настройке Comeet Recruiting Software для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые необходимо выполнить в Comeet Recruiting Software и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Comeet Recruiting Software.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [Клиент Comeet Recruiting Software](https://www.comeet.co/)
* Учетная запись пользователя в Comeet Recruiting Software с разрешениями администратора.

## <a name="add-comeet-recruiting-software-from-the-gallery"></a>Добавление Comeet Recruiting Software из коллекции

Перед настройкой Comeet Recruiting Software для автоматической подготовки пользователей в Azure AD необходимо добавить Comeet Recruiting Software из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Comeet Recruiting Software из коллекции приложений Azure AD, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Comeet Recruiting Software**, выберите пункт **Comeet Recruiting Software** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Comeet Recruiting Software в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-comeet-recruiting-software"></a>Назначение пользователей в Comeet Recruiting Software

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей необходимо решить, какие пользователи или группы в Azure AD будут иметь доступ к Comeet Recruiting Software. После этого пользователей и (или) группы можно будет назначить Comeet Recruiting Software, следуя приведенным ниже указаниям.

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-comeet-recruiting-software"></a>Важные советы по назначению пользователей в Comeet Recruiting Software

* Рекомендуется назначить одного пользователя Azure AD платформе Comeet Recruiting Software для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя Comeet Recruiting Software в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-comeet-recruiting-software"></a>Настройка автоматической подготовки пользователей в Comeet Recruiting Software 

В этом разделе описано, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Comeet Recruiting Software на основе их назначений в Azure AD.

> [!TIP]
> Вы также можете включить единый вход на основе SAML для Comeet Recruiting Software, следуя инструкциям в [руководстве по единому входу для Comeet Recruiting Software](comeetrecruitingsoftware-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-comeet-recruiting-software-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей для Comeet Recruiting Software в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Comeet Recruiting Software**.

    ![Ссылка на Comeet Recruiting Software в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите **URL-адрес клиента** и **секретный токен** учетной записи Comeet Recruiting Software, как описано на шаге 6.

6. В [консоли администрирования Comeet Recruiting Software](https://app.comeet.co/) выберите **Comeet > Параметры > Аутентификация > Microsoft Azure** и скопируйте **Секретный токен для вашей компании** в поле **Секретный токен** в Azure AD.

    ![Comeet Recruiting Software, подготовка](./media/comeet-recruiting-software-provisioning-tutorial/secret-token-1.png)

7. После заполнения поля, указанного в шаге 5, нажмите кнопку **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Comeet Recruiting Software. Если установить подключение не удалось, убедитесь, что в учетной записи Comeet Recruiting Software есть разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-token.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

9. Выберите команду **Сохранить**.

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Comeet** (Синхронизировать пользователей Azure Active Directory с Comeet).

    ![Comeet Recruiting Software, сопоставление пользователей](media/comeet-recruiting-software-provisioning-tutorial/user-mappings.png)

11. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Comeet Recruiting Software. Атрибуты, выбранные как свойства в разделе **Сопоставление**, используются для сопоставления учетных записей пользователей в Comeet Recruiting Software для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Comeet Recruiting Software, групповые атрибуты](media/comeet-recruiting-software-provisioning-tutorial/user-mapping-attributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Comeet Recruiting Software, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей или группы для подготовки в Comeet Recruiting Software, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Comeet Recruiting Software.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* Comeet Recruiting Software в настоящее время не поддерживает группы.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

