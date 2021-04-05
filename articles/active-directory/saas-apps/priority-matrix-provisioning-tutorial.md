---
title: Учебник по настройке Priority Matrix для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Priority Matrix.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/08/2019
ms.author: Zhchia
ms.openlocfilehash: e79f21300325c6b451dd564bf2c69830f003f55c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94357867"
---
# <a name="tutorial-configure-priority-matrix-for-automatic-user-provisioning"></a>Учебник по настройке Priority Matrix для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Priority Matrix и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Priority Matrix.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [Клиент Priority Matrix](https://appfluence.com/pricing/)
* Учетная запись пользователя Priority Matrix с разрешениями администратора.

## <a name="assign-users-to-priority-matrix"></a>Назначение пользователей в Priority Matrix

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие "назначение". В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Priority Matrix. Когда данный вопрос будет решен, этих пользователей или группы можно будет назначить приложению Priority Matrix, следуя инструкциям:

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-priority-matrix"></a>Важные рекомендации по назначению пользователей в Priority Matrix

* Рекомендуется назначить одного пользователя Azure AD в Priority Matrix для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Priority Matrix в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-priority-matrix-for-provisioning"></a>Настройка Priority Matrix для подготовки

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для Priority Matrix, нужно получить из Priority Matrix определенные сведения о подготовке.

1. Войдите в [консоль администрирования Priority Matrix](https://sync.appfluence.com/accounts/login/?next=/accounts/provisioning).

3. Щелкните **Oauth login token** (Токен входа OAuth) для Priority Matrix.

    ![Добавление SCIM в Priority Matrix](media/priority-matrix-provisioning-tutorial/oauthlogin.png)

4. Нажмите кнопку **GET NEW TOKEN** (Получить новый токен). Скопируйте **строку токена**. Его нужно будет ввести на вкладке "Подготовка" в поле **Секретный токен** для приложения Priority Matrix на портале Azure. 

    ![Создание токена в Priority Matrix](media/priority-matrix-provisioning-tutorial/token.png)

## <a name="add-priority-matrix-from-the-gallery"></a>Добавление Priority Matrix из коллекции

Чтобы настроить Priority Matrix для автоматической подготовки пользователей в Azure AD, необходимо добавить Priority Matrix из коллекции приложений Azure AD в список управляемых приложений SaaS.

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Priority Matrix** и выберите **Priority Matrix** на панели результатов. 

    ![Priority Matrix в списке результатов](common/search-new-app.png)

5. Нажмите кнопку **Sign-up for Priority Matrix** (Зарегистрироваться в Priority Matrix), которая перенаправит вас на страницу входа в Priority Matrix. 

    ![Добавление OIDC в Priority Matrix](media/priority-matrix-provisioning-tutorial/signup.png)

6. Так как Priority Matrix — это приложение OpenIDConnect, для входа в Priority Matrix необходимо выбрать рабочую учетную запись Майкрософт.

    ![Вход в OIDC в Priority Matrix](media/priority-matrix-provisioning-tutorial/msftsignin.png)

7. После успешной проверки подлинности дайте согласие на странице согласия. Теперь приложение будет автоматически добавлено к вашему клиенту и вы перейдете в учетную запись Priority Matrix.

    ![Согласие для OIDC в Priority Matrix](media/priority-matrix-provisioning-tutorial/consent.png)

## <a name="configure-automatic-user-provisioning-to-priority-matrix"></a>Настройка автоматической подготовки пользователей в Priority Matrix 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Priority Matrix на основе их назначений в Azure AD.

> [!NOTE]
> Дополнительные сведения о конечной точке SCIM в Priority Matrix см. в статье [Подготовка пользователей в Priority Matrix](https://appfluence.com/help/article/user-provisioning/).

### <a name="to-configure-automatic-user-provisioning-for-priority-matrix-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей Priority Matrix в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Priority Matrix**.

    ![Ссылка на Priority Matrix в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://sync.appfluence.com/scim/v2/` в поле **URL-адрес клиента**. Введите в поле **Секретный токен** значение, которое вы ранее получили из Priority Matrix и сохранили. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Priority Matrix. Если установить подключение не удалось, убедитесь, что у учетной записи Priority Matrix есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Priority Matrix**.

    ![Сопоставления пользователей Priority Matrix](media/priority-matrix-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Priority Matrix. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления учетных записей пользователей в Priority Matrix для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователей Priority Matrix](media/priority-matrix-provisioning-tutorial/userattributes.png)

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Priority Matrix, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в Priority Matrix, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Priority Matrix.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)


