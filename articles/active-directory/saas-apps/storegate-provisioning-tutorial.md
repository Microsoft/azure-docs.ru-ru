---
title: Руководство по настройке Storegate для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Storegate.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/15/2019
ms.author: Zhchia
ms.openlocfilehash: c984beff630ef90ea33a13e2fef1bca0189c2314
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94357952"
---
# <a name="tutorial-configure-storegate-for-automatic-user-provisioning"></a>Руководство по настройке Storegate для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Storegate и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и (или) групп в Storegate.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Storegate](https://www.storegate.com);
* учетная запись пользователя в Storegate с разрешениями администратора.

## <a name="assign-users-to-storegate"></a>Назначение пользователей Storegate

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие "назначение". В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Storegate. Затем назначьте этих пользователей и (или) группы приложению Storegate, следуя приведенным ниже указаниям.

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-storegate"></a>Важные советы по назначению пользователей в Storegate

* Рекомендуется назначить одного пользователя Azure AD в Storegate для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Storegate в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-storegate-for-provisioning"></a>Настройка Storegate для подготовки

Перед настройкой Storegate для автоматической подготовки пользователей с помощью Azure AD вам необходимо получить определенные сведения о подготовке из Storegate.

1. Войдите в [консоль администрирования Storegate](https://ws1.storegate.com/identity/core/login?signin=c71fb8fe18243c571da5b333d5437367) и перейдите к разделу параметров, щелкнув значок пользователя в верхнем правом углу страницы и выбрав **Параметры учетной записи**.

    ![Добавление SCIM в Storegate](media/storegate-provisioning-tutorial/admin.png)

2. В разделе параметров выберите **Команда > Параметры** и проверьте, включен ли переключатель в разделе **Единый вход**.

    ![Параметры Storegate](media/storegate-provisioning-tutorial/team.png)

    ![Выключатель Storegate](media/storegate-provisioning-tutorial/sso.png)

3. Скопируйте **URL-адрес клиента** и **токен**. Эти значения нужно будет указать в полях **URL-адрес клиента** и **Секретный токен** соответственно на вкладке "Подготовка" приложения Storegate на портале Azure. 

    ![Создание токена в Storegate](media/storegate-provisioning-tutorial/token.png)

## <a name="add-storegate-from-the-gallery"></a>Добавление Storegate из коллекции

Чтобы настроить Storegate для автоматической подготовки пользователей в Azure AD, необходимо добавить Storegate из коллекции приложений Azure AD в список управляемых приложений SaaS.

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Storegate** и выберите **Storegate** в области результатов. 

    ![Storegate в списке результатов](common/search-new-app.png)

5. Нажмите кнопку **Sign-up for Storegate** (Зарегистрироваться в Storegate), которая перенаправит вас на страницу входа в Storegate. 

    ![Добавление OIDC в Storegate](media/storegate-provisioning-tutorial/signup.png)

6.  Войдите в [консоль администрирования Storegate](https://ws1.storegate.com/identity/core/login?signin=c71fb8fe18243c571da5b333d5437367) и перейдите к разделу параметров, щелкнув значок пользователя в верхнем правом углу страницы и выбрав **Параметры учетной записи**.

    ![Вход в систему Storegate](media/storegate-provisioning-tutorial/admin.png)

7. В разделе параметров выберите **Команда > Параметры** и щелкните переключатель в разделе "Единый вход". Запустится процесс предоставления согласия. Щелкните **Активировать**.

    ![Команда Storegate](media/storegate-provisioning-tutorial/team.png)

    ![Единый вход в Storegate](media/storegate-provisioning-tutorial/sso.png)

    ![Активация Storegate](media/storegate-provisioning-tutorial/activate.png)

8. Так как Storegate — это приложение OpenIDConnect, для входа в Storegate необходимо выбрать рабочую учетную запись Майкрософт.

    ![Вход в Storegate с помощью OIDC](media/storegate-provisioning-tutorial/login.png)

9. После успешной проверки подлинности дайте согласие на странице согласия. После этого приложение автоматически добавится к вашему клиенту, и вы будете перенаправлены в свою учетную запись Storegate.

    ![Согласие OIDC в Storegate](media/storegate-provisioning-tutorial/accept.png)

## <a name="configure-automatic-user-provisioning-to-storegate"></a>Настройка автоматической подготовки пользователей в Storegate 

В этом разделе описана настройка службы подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Storegate на основе их назначений в Azure AD.

> [!NOTE]
> Дополнительные сведения о конечной точке SCIM Storegate см. на [этой странице](https://en-support.storegate.com/article/step-by-step-instruction-how-to-enable-azure-provisioning-to-your-storegate-team-account/).

### <a name="to-configure-automatic-user-provisioning-for-storegate-in-azure-ad"></a>Настройка автоматической подготовки пользователей для Storegate в Azure AD

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Storegate**.

    ![Ссылка на Storegate в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://dialpad.com/scim` в поле **URL-адрес клиента**. Введите в поле **Секретный токен** полученное и сохраненное ранее значение из Storegate. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Storegate. Если установить подключение не удалось, убедитесь, что у учетной записи Storegate есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Storegate** (Синхронизировать пользователей Azure Active Directory с Storegate).

    ![Сопоставления пользователей Storegate](media/storegate-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставление атрибутов** просмотрите атрибуты пользователей, синхронизированные из Azure AD в Storegate. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Storegate для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователей Storegate](media/storegate-provisioning-tutorial/userattributes.png)

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Storegate, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей или группы для подготовки в Storegate, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Storegate.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
