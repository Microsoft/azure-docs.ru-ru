---
title: Руководство по настройке Visitly для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Visitly.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/30/2019
ms.author: Zhchia
ms.openlocfilehash: ff3f3ab65df2d801b7c962de7cce645e9fc00b30
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94358615"
---
# <a name="tutorial-configure-visitly-for-automatic-user-provisioning"></a>Руководство по настройке Visitly для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Visitly и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Visitly.

> [!NOTE]
> В этом учебнике рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, а также часто задаваемые вопросы см. в статье [Что представляет собой автоматическая подготовка пользователей для приложений SaaS в Azure AD?](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Visitly](https://www.visitly.io/pricing/);
* учетная запись пользователя в Visitly с разрешениями администратора.

## <a name="assign-users-to-visitly"></a>Назначение пользователей в Visitly 

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и группы, которые были назначены приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей решите, какие пользователи или группы в Azure AD должны иметь доступ к Visitly. После этого можно будет назначить этих пользователей или группы приложению Visitly, следуя инструкциям:
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-visitly"></a>Важные рекомендации по назначению пользователей в Visitly 

* Мы рекомендуем назначить одного пользователя Azure AD в Visitly, чтобы проверить настройки автоматической подготовки пользователей. Дополнительных пользователей или группы можно назначить позже.

* При назначении пользователя в Visitly в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью Доступ по умолчанию исключаются из подготовки.

## <a name="set-up-visitly-for-provisioning"></a>Настройка Visitly для подготовки

Перед настройкой автоматической подготовки пользователей с помощью Azure AD в Visitly необходимо включить подготовку SCIM (System for Cross-domain Identity Management).

1. Войдите в приложение [Visitly](https://app.visitly.io/login). Выберите **Integrations (Интеграции)**  > **Host Synchronization (Синхронизация узла)** .

    ![Синхронизация узла](media/Visitly-provisioning-tutorial/login.png)

2. Выберите раздел **Azure AD**.

    ![Раздел "Azure AD"](media/Visitly-provisioning-tutorial/integration.png)

3. Скопируйте значение поля **API Key** (Ключ API). Эти значения нужно ввести в поле **Секретный токен** на вкладке **Подготовка** для приложения Visitly на портале Azure.

    ![Ключ API](media/Visitly-provisioning-tutorial/token.png)


## <a name="add-visitly-from-the-gallery"></a>Добавление Visitly из коллекции

Чтобы настроить Visitly для автоматической подготовки пользователей в Azure AD, добавьте Visitly из коллекции приложений Azure AD в список управляемых приложений SaaS.

Чтобы добавить приложение Visitly из коллекции Azure AD, выполните следующие инструкции:

1. На [портале Azure](https://portal.azure.com) в области навигации слева выберите **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Visitly**, выберите **Visitly** на панели результатов, а затем нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Visitly в списке результатов](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-visitly"></a>Настройка автоматической подготовки пользователей в Visitly 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей или групп в Visitly на основе их назначений в Azure AD.

> [!TIP]
> Чтобы включить единый вход на основе SAML в Visitly, следуйте инструкциям в [этом учебнике](Visitly-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="configure-automatic-user-provisioning-for-visitly-in-azure-ad"></a>Настройка автоматической подготовки пользователей для Visitly в Azure AD

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения** > **Все приложения**.

    ![Все приложения](common/enterprise-applications.png)

2. В списке приложений выберите **Visitly**.

    ![Ссылка на Visitly в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Вкладка "Подготовка"](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Для параметра "Режим подготовки к работе" задано значение "Авто"](common/provisioning-automatic.png)

5. В разделе "Учетные данные администратора" введите `https://api.visitly.io/v1/usersync/SCIM` и полученное ранее значение **API Key** (Ключ API) в поля **URL-адрес клиента** и **Секретный токен** соответственно. На портале Azure выберите **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Visitly. Если установить подключение не удалось, убедитесь, что у учетной записи Visitly есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента и токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Уведомление по электронной почте](common/provisioning-notification-email.png)

7. Нажмите кнопку **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Visitly** (Синхронизировать пользователей Azure Active Directory с Visitly).

    ![Сопоставления пользователей Visitly](media/visitly-provisioning-tutorial/usermapping.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователя, синхронизированные из Azure AD в Visitly. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Visitly для операций обновления. Чтобы зафиксировать изменения, щелкните **Сохранить**.

    ![Атрибуты пользователя Visitly](media/visitly-provisioning-tutorial/userattribute.png)

10. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в [этом учебнике](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Visitly, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей или группы для подготовки в Visitly, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей или групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Дополнительные сведения о том, сколько продлится подготовка пользователей или групп, см. в разделе [Сколько времени занимает подготовка пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Visitly. Дополнительные сведения см. в статье [Проверка состояния подготовки пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Сведения о чтении журналов подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

В Visitly не поддерживается безвозвратное удаление. Применяется только обратимое удаление.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Managing user account provisioning for enterprise apps in the Azure portal](../app-provisioning/configure-automatic-user-provisioning-portal.md) (Управление подготовкой учетных записей пользователей для корпоративных приложений на портале Azure)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
