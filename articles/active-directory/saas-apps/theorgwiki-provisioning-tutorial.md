---
title: Руководство по настройка TheOrgWiki для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для TheOrgWiki.
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
ms.openlocfilehash: 8238b9902aafcabc079c551a0eabc7170042209a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94357629"
---
# <a name="tutorial-configure-theorgwiki-for-automatic-user-provisioning"></a>Руководство по настройке TheOrgWiki для автоматической подготовки пользователей

В этом руководстве описаны шаги, которые нужно выполнить в TheOrgWiki и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и групп в Azure AD для TheOrgWiki.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [Клиент OrgWiki.](https://www.theorgwiki.com/welcome/)
* Учетная запись пользователя TheOrgWiki с разрешениями администратора.

## <a name="assign-users-to-theorgwiki"></a>Назначение пользователей в TheOrgWiki

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие "назначение". В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к TheOrgWiki. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению TheOrgWiki по приведенным ниже инструкциям.

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-theorgwiki"></a>Важные замечания о назначении пользователей для TheOrgWiki

* Рекомендуется назначить одного пользователя Azure AD для TheOrgWiki, чтобы проверить конфигурацию автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для TheOrgWiki в диалоговом окне назначения нужно выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-theorgwiki-for-provisioning"></a>Настройка подготовки в TheOrgWiki

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для TheOrgWiki, нужно включить подготовку SCIM для TheOrgWiki.

1. Войдите в [консоль администрирования TheOrgWiki](https://www.theorgwiki.com/login/). Щелкните **Admin Console** (Консоль администрирования).

    ![Снимок экрана: страница TheOrgWiki, где выделены аватар пользователя и консоль администрирования.](media/theorgwiki-provisioning-tutorial/login.png)

2. В консоли администрирования щелкните вкладку **Settings** (Параметры). 

    ![Снимок экрана: консоль администрирования TheOrgWiki, где выделена вкладка параметров.](media/theorgwiki-provisioning-tutorial/settings.png)
    
3. Откройте раздел **Service Accounts** (Учетные записи служб).

    ![Снимок экрана: страница Service Accounts (Учетные записи служб) в консоли администрирования TheOrgWiki.](media/theorgwiki-provisioning-tutorial/serviceaccount.png)

4. Щелкните **+ Service Account** (Добавить учетную запись службы). В разделе **Service Account Type** (Тип учетной записи службы) выберите **Token Based** (На основе токенов). Выберите команду **Сохранить**.

    ![Снимок экрана: диалоговое окно New Service Account (Создание учетной записи службы), где выделены элементы Service Account Type (Тип учетной записи службы), Token Based (На основе токенов) и Save (Сохранить).](media/theorgwiki-provisioning-tutorial/auth.png)

5.  Скопируйте значение **Active Tokens** (Активные токены). Его нужно будет ввести в поле "Секретный токен" на вкладке "Подготовка" для приложения TheOrgWiki на портале Azure.
     
    ![Снимок экрана: диалоговое окно подготовки Manage Tokens for SCIM (Управление токенами для SCIM).](media/theorgwiki-provisioning-tutorial/token.png)

## <a name="add-theorgwiki-from-the-gallery"></a>Добавление TheOrgWiki из коллекции

Чтобы настроить автоматическую подготовку пользователей для TheOrgWiki с помощью Azure AD, нужно добавить TheOrgWiki из коллекции приложений Azure AD в список управляемых приложений SaaS.

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **TheOrgWiki** и выберите **TheOrgWiki** на панели результатов. 

    ![TheOrgWiki в списке результатов](common/search-new-app.png)

5. Нажмите кнопку **Sign-up for TheOrgWiki** (Зарегистрироваться в TheOrgWiki), которая перенаправит вас на страницу входа в TheOrgWiki. 

    ![Снимок экрана: страница входа в TheOrgWiki, где выделен URL-адрес страницы](media/theorgwiki-provisioning-tutorial/image00.png)

6.  В правом верхнем углу выберите **Login** (Войти).

    ![Снимок экрана: правый верхний угол страницы входа, где выделен элемент Login (Войти).](media/theorgwiki-provisioning-tutorial/image02.png)

7. Так как TheOrgWiki — это приложение OpenIDConnect, для входа в TheOrgWiki необходимо выбрать рабочую учетную запись Майкрософт.

    ![Снимок экрана: страница входа в TheOrgWiki, где выделен элемент Sign in with Microsoft (Войдите с помощью учетной записи Майкрософт).](media/theorgwiki-provisioning-tutorial/image03.png)
    
8. После успешной проверки подлинности приложение будет автоматически добавлено в ваш клиент, и вы будете перенаправлены в свою учетную запись TheOrgWiki.

    ![Добавление SCIM в TheOrgWiki](media/theorgwiki-provisioning-tutorial/image04.png)

## <a name="configure-automatic-user-provisioning-to-theorgwiki"></a>Настройка автоматической подготовки пользователей для TheOrgWiki 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TheOrgWiki на основе их назначений в Azure AD.


### <a name="to-configure-automatic-user-provisioning-for-theorgwiki-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для TheOrgWiki сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **TheOrgWiki**.

    ![Ссылка на TheOrgWiki в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://<TheOrgWiki Subdomain        value>.theorgwiki.com/api/v2/scim/v2/` в поле **URL-адрес клиента**. 

    Например, `https://test1.theorgwiki.com/api/v2/scim/v2/`.

> [!NOTE]
> **Значение поддомена** можно задать для TheOrgWiki только во время первоначального процесса регистрации.
 
6. Введите в поле **Секретный токен** значение токена, полученное ранее из TheOrgWiki. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к TheOrgWiki. Если установить подключение не удалось, убедитесь, что у учетной записи TheOrgWiki есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

7. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

8. Выберите команду **Сохранить**.

9. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to TheOrgWiki** (Синхронизировать пользователей Azure Active Directory с TheOrgWiki).

    ![Сопоставления пользователей TheOrgWiki](media/theorgwiki-provisioning-tutorial/usermapping.png)

10. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в TheOrgWiki. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в TheOrgWiki при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователя TheOrgWiki](media/theorgwiki-provisioning-tutorial/userattribute.png).

11. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Чтобы включить службу подготовки Azure AD для TheOrgWiki, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

13. Укажите пользователей или группы для подготовки в TheOrgWiki, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

14. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Дополнительные сведения о том, сколько продлится подготовка пользователей и (или) групп, см. в разделе [Сколько времени занимает подготовка пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением TheOrgWiki. Дополнительные сведения см. в статье [Проверка состояния подготовки пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Сведения о чтении журналов подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md).