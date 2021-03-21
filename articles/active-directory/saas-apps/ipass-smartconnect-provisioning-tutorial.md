---
title: Руководство. Настройка iPass SmartConnect для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в iPass SmartConnect.
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
ms.openlocfilehash: 405a7bc3b653ca7bca026d3318763a4922244e88
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97093712"
---
# <a name="tutorial-configure-ipass-smartconnect-for-automatic-user-provisioning"></a>Руководство. Настройка iPass SmartConnect для автоматической подготовки пользователей

В этом руководстве описаны шаги, которые нужно выполнить в iPass SmartConnect и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и (или) групп в Azure AD для iPass SmartConnect.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [Клиент iPass SmartConnect.](https://www.ipass.com/buy-ipass/)
* Учетная запись пользователя в iPass SmartConnect с разрешениями администратора.

## <a name="assigning-users-to-ipass-smartconnect"></a>Назначение пользователей в iPass SmartConnect

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к iPass SmartConnect. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению iPass SmartConnect, следуя приведенным ниже инструкциям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-ipass-smartconnect"></a>Важные советы по назначению пользователей в iPass SmartConnect

* Рекомендуется назначить одного пользователя Azure AD в iPass SmartConnect для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в iPass SmartConnect в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="setup-ipass-smartconnect-for-provisioning"></a>Настройка iPass SmartConnect для подготовки

Перед настройкой iPass SmartConnect для автоматической подготовки пользователей с помощью Azure AD вам необходимо получить сведения о конфигурации из консоли администрирования iPass SmartConnect.

1. Токен носителя, который требуется для проверки подлинности в конечной точке SCIM iPass SmartConnect, предоставляется только при начальной настройке iPass SmartConnect. 
2. Если у вас нет токена носителя, обратитесь в [службу поддержки iPass SmartConnect](mailto:help@ipass.com), чтобы получить новый.

## <a name="add-ipass-smartconnect-from-the-gallery"></a>Добавление iPass SmartConnect из коллекции

Чтобы настроить iPass SmartConnect для автоматической подготовки пользователей в Azure AD, необходимо добавить iPass SmartConnect из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить iPass SmartConnect из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **iPass SmartConnect**, выберите **iPass SmartConnect** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![iPass SmartConnect в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-ipass-smartconnect"></a>Настройка автоматической подготовки пользователей в iPass SmartConnect 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в iPass SmartConnect на основе их назначений в Azure AD.

> [!TIP]
>  Для iPass SmartConnect также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [руководстве по единому входу в iPass SmartConnect](ipasssmartconnect-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-ipass-smartconnect-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для iPass SmartConnect сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **iPass SmartConnect**.

    ![Ссылка iPass SmartConnect в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://openmobile.ipass.com/moservices/scim/v1` в поле **URL-адрес клиента**. Введите полученное ранее значение токена носителя в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к iPass SmartConnect. Если установить подключение не удалось, убедитесь, что у учетной записи iPass SmartConnect есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с iPass SmartConnect**.

    :::image type="content" source="media/ipass-smartconnect-provisioning-tutorial/usermapping.png" alt-text="Снимок экрана: раздел &quot;Сопоставления&quot;. Под полем &quot;Имя&quot; отображается элемент &quot;Синхронизировать пользователей Azure Active Directory с iPass SmartConnect&quot;." border="false":::

9. В разделе **Сопоставление атрибутов** просмотрите атрибуты пользователей, синхронизированные из Azure AD в iPass SmartConnect. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в iPass SmartConnect для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    :::image type="content" source="media/ipass-smartconnect-provisioning-tutorial/userattribute.png" alt-text="Снимок экрана: страница &quot;Сопоставление атрибутов&quot;. В таблице перечислены атрибуты Azure Active Directory и iPass SmartConnect с указанием приоритета сопоставления." border="false":::


10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для iPass SmartConnect, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в iPass SmartConnect, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением iPass SmartConnect.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* iPass SmartConnect принимает только имена пользователей, имена доменов которых зарегистрированы в консоли администрирования iPass SmartConnect.  

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
