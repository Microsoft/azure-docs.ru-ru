---
title: Руководство по настройке Bonusly для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Bonusly.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: d8c3f64e5cb5269bfe7e555615f874ac3443c6eb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94357833"
---
# <a name="tutorial-configure-bonusly-for-automatic-user-provisioning"></a>Руководство по настройке Bonusly для автоматической подготовки пользователей

В этом руководстве описаны шаги, которые нужно выполнить в Bonusly и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Bonusly.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом руководстве, предполагает, что у вас уже имеется:

* клиент Azure AD;
* [клиент Bonusly](https://bonus.ly/pricing);
* учетная запись пользователя в Bonusly с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от [Rest API Bonusly](https://konghq.com/solutions/gateway/), доступного для разработчиков Bonusly.

## <a name="adding-bonusly-from-the-gallery"></a>Добавление Bonusly из коллекции

Перед настройкой Bonusly для автоматической подготовки пользователей в Azure AD необходимо добавить Bonusly из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Bonusly из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Bonusly**, выберите **Bonusly** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Bonusly в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-bonusly"></a>Назначение пользователей в Bonusly

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая "назначение". В контексте автоматической подготовки синхронизированы будут только те пользователи и группы, которые были "назначены" приложению в Azure AD. 

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Bonusly. Когда данный вопрос будет решен, этих пользователей или группы можно будет назначить приложению Bonusly, следуя инструкциям:

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-bonusly"></a>Важные рекомендации по назначению пользователей в Bonusly

* Рекомендуется назначить одного пользователя Azure AD в Bonusly для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Bonusly в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-bonusly"></a>Настройка автоматической подготовки пользователей в Bonusly

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Bonusly на основе их назначений в Azure AD.

> [!TIP]
> Для Bonusly также можно включить единый вход на основе SAML. Для этого выполните инструкции, приведенные в статье [Руководство. Интеграция Azure Active Directory с Bonusly](bonus-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-bonusly-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Bonusly, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com), а затем выберите **Корпоративные приложения**, **Все приложения** и **Bonusly**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Bonusly**.

    ![Ссылка на Bonusly в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ProvisioningTab.png" alt-text="Снимок экрана вкладки &quot;Bonusly — Подготовка&quot;. В разделе &quot;Управление&quot; выделен пункт &quot;Подготовка&quot;." border="false":::

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ProvisioningCredentials.png" alt-text="Снимок экрана, на котором показан список &quot;Режим подготовки&quot; с выделенным и выбранным пунктом &quot;Авто&quot;." border="false":::

5. В разделе **Учетные данные администратора** укажите **секретный токен** учетной записи Bonusly, как описано в шаге 6.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/secrettoken.png" alt-text="Снимок экрана: раздел &quot;Учетные данные администратора&quot;. Поле &quot;Секретный токен&quot; выделено, но оно пустое." border="false":::

6. **Секретный токен** учетной записи Bonusly можно найти в разделе **Admin > Company > Integrations** (Администрирование > Организация > Интеграции). В разделе **If you want to code** (Если вы хотите написать код ) выберите **API> Create New API Access Token** (Создать новый токен доступа к API), чтобы создать новый секретный токен.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/BonuslyIntegrations.png" alt-text="Снимок экрана меню Bonusly. В разделе Admin (Администрирование) выделен пункт Company (Компания). В разделе Company (Компания), выделена интеграция." border="false":::

    :::image type="content" source="./media/bonusly-provisioning-tutorial/BonsulyRestApi.png" alt-text="Снимок экрана, сайта Bonusly на котором показан раздел If you want to code (Если вы хотите создать код) с выделенным разделом API." border="false":::

    :::image type="content" source="./media/bonusly-provisioning-tutorial/CreateToken.png" alt-text="Снимок экрана сайта Bonusly. Открыта вкладка Services (Службы). В разделе Your API access tokens (Ваши маркеры доступа API) выделена ссылка Create new API access token (Создать новый маркер доступа API)." border="false":::

7. На следующем экране введите имя маркера доступа в предоставленном текстовом поле, а затем нажмите кнопку **Create Api Key** (Создать ключ API). Новый маркер доступа будет отображаться во всплывающем окне в течение нескольких секунд.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/Token01.png" alt-text="Снимок экрана сайта Bonusly со страницей New access token (Создание маркера доступа). Активное поле содержит текст My Token (Мой маркер), а кнопка Create API key (Создать ключ API ) выделена." border="false":::

    :::image type="content" source="./media/bonusly-provisioning-tutorial/Token02.png" alt-text="Снимок экрана сайта Bonusly. Отображается уведомление, содержащее текст New access token created (Созданный маркер доступа) и сам маркер (неразборчиво)." border="false":::

8. После заполнения полей, указанных на шаге 5, щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Bonusly. Если установить подключение не удалось, убедитесь, что у учетной записи Bonusly есть разрешения администратора, и повторите попытку.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/TestConnection.png" alt-text="Снимок экрана раздела &quot;Учетные данные администратора&quot; на портале Azure. Выделена кнопка &quot;Проверить подключение&quot;." border="false":::

9. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    :::image type="content" source="./media/bonusly-provisioning-tutorial/EmailNotification.png" alt-text="Снимок экрана с пустым полем электронной почты для уведомления. Виден параметр &quot;Отправлять мне уведомления электронной почтой о сбоях&quot;." border="false":::

10. Выберите команду **Сохранить**.

11. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Bonusly** (Синхронизировать пользователей Azure Active Directory с Bonusly).

    :::image type="content" source="./media/bonusly-provisioning-tutorial/UserMappings.png" alt-text="Снимок экрана: раздел &quot;Сопоставления&quot;. Под полем &quot;Имя&quot; выделен элемент &quot;Синхронизировать пользователей Azure Active Directory с Bonusly&quot;." border="false":::

12. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Bonusly. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Bonusly для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/UserAttributeMapping.png" alt-text="Снимок экрана: страница &quot;Сопоставление атрибутов&quot;. В таблице перечислены все атрибуты Azure Active Directory, соответствующие атрибутам Bonusly, с указанием состояния сопоставления." border="false":::

13. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

14. Чтобы включить службу подготовки Azure AD для Bonusly, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ProvisioningStatus.png" alt-text="Снимок экрана раздела &quot;Параметры&quot;. Переключатель состояния подготовки находится в положении Off (Выкл.)." border="false":::

15. Определите пользователей или группы для подготовки в Bonusly, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ScopeSync.png" alt-text="Снимок экрана, на котором показан список &quot;Область&quot;. В поле выбран параметр &quot;Синхронизация только назначенных пользователей и групп&quot;." border="false":::

16. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/SaveProvisioning.png" alt-text="Снимок экрана страницы &quot;Bonusly — Подготовка&quot; с выделенной кнопкой &quot;Сохранить&quot;." border="false":::

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Bonusly.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/bonusly-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/bonusly-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/bonusly-provisioning-tutorial/tutorial_general_03.png