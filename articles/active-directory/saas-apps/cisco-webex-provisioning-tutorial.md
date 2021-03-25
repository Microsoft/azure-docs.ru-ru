---
title: Руководство по настройке Cisco Webex для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в Cisco Webex.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/12/2019
ms.author: zhchia
ms.openlocfilehash: bc05e83ac6c7f0f7c5e9a571c1fa7397af858f44
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96180106"
---
# <a name="tutorial-configure-cisco-webex-for-automatic-user-provisioning"></a>Руководство по настройке Cisco Webex для автоматической подготовки пользователей

В этом руководстве описаны шаги, которые нужно выполнить в Cisco Webex и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отмены подготовки пользователей в Cisco Webex.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель доступен в режиме предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [Клиент Cisco Webex](https://www.webex.com/pricing/index.html).
* Учетная запись пользователя в Cisco Webex с разрешениями администратора.

## <a name="adding-cisco-webex-from-the-gallery"></a>добавление Cisco Webex из коллекции;

Перед настройкой Cisco Webex для автоматической подготовки пользователей в Azure AD необходимо добавить Cisco Webex из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Cisco Webex из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Cisco Webex**, выберите **Cisco Webex** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Cisco Webex в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-cisco-webex"></a>Назначение пользователей в Cisco Webex

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая "назначение". В контексте автоматической подготовки синхронизированы будут только те пользователи и группы, которые были "назначены" приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Cisco Webex. После этого можно назначить этих пользователей для приложения Cisco Webex, выполнив следующие действия:

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-cisco-webex"></a>Важные рекомендации по назначению пользователей в Cisco Webex

* Рекомендуется назначить одного пользователя Azure AD в Cisco Webex для проверки конфигурации автоматической подготовки пользователей. Дополнительные пользователи могут быть назначены позднее.

* При назначении пользователя в Cisco Webex в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если она доступна). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-cisco-webex"></a>Настройка автоматической подготовки пользователей в Cisco Webex

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Cisco Webex на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-cisco-webex-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей Cisco Webex в Azure AD, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com) и последовательно выберите **Корпоративные приложения**, **Все приложения**, а затем **Cisco Webex**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Cisco Webex**.

    ![Ссылка на Cisco Webex в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    :::image type="content" source="common/provisioning.png" alt-text="Снимок экрана: меню на портале Azure. В разделе &quot;Управление&quot; выделен пункт &quot;Подготовка&quot;." border="false":::

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    :::image type="content" source="common/provisioning-automatic.png" alt-text="Снимок экрана: список &quot;Режим подготовки&quot; с выделенным пунктом Automatic (Автоматически)." border="false":::

5. В разделе **Учетные данные администратора** укажите **URL-адрес клиента** и **Маркер секрета** для вашей учетной записи Cisco Webex.

    :::image type="content" source="./media/cisco-webex-provisioning-tutorial/secrettoken1.png" alt-text="Снимок экрана: раздел &quot;Учетные данные администратора&quot;. Поля &quot;URL-адрес клиента&quot; и &quot;Секретный токен&quot; выделены, но пусты." border="false":::

6.  В поле **URL-адрес клиента** введите значение в формате `https://api.ciscospark.com/v1/scim/[OrgId]`. Чтобы получить `[OrgId]`, войдите в [Центр управления Cisco Webex](https://admin.webex.com/login). Щелкните имя своей организации в левом нижнем углу и скопируйте значение из **идентификатора организации**. 

    * Чтобы получить значение для поля **Секретный токен**, перейдите по этому [URL-адресу](https://idbroker.webex.com/idb/saml2/jsp/doSSO.jsp?type=login&goto=https%3A%2F%2Fidbroker.webex.com%2Fidb%2Foauth2%2Fv1%2Fauthorize%3Fresponse_type%3Dtoken%26client_id%3DC4ca14fe00b0e51efb414ebd45aa88c1858c3bfb949b2405dba10b0ca4bc37402%26redirect_uri%3Dhttp%253A%2f%2flocalhost%253A3000%2fauth%2fcode%26scope%3Dspark%253Apeople_read%2520spark%253Apeople_write%2520Identity%253ASCIM%26state%3Dthis-should-be-a-random-string-for-security-purpose). На открывшейся странице входа в Webex войдите с помощью полной учетной записи администратора Cisco Webex для своей организации. Появится страница ошибки с сообщением о том, что сайт недоступен, но это обычное явление.

        :::image type="content" source="./media/cisco-webex-provisioning-tutorial/test.png" alt-text="Снимок экрана: веб-страница с сообщением об ошибке. Сообщение информирует о том, что сайт недоступен, и содержит несколько советов по устранению неполадок." border="false":::
 
    * Скопируйте значение созданного токена носителя из URL-адреса, выделенного ниже. Этот токен действителен в течение 365 дней.
        
        :::image type="content" source="./media/cisco-webex-provisioning-tutorial/test1.png" alt-text="Снимок экрана с длинным URL-адресом. Часть адреса невозможно расшифровать, но он выделен и помечен как токен носителя." border="false":::

7. После заполнения полей, указанных в шаге 5, нажмите кнопку **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Cisco Webex. Если установить подключение не удалось, убедитесь, что у учетной записи Cisco Webex есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)
   
8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

9. Выберите команду **Сохранить**.

10. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Cisco Webex**.

    :::image type="content" source="./media/cisco-webex-provisioning-tutorial/usermapping.png" alt-text="Снимок экрана: раздел &quot;Сопоставления&quot; на портале Azure. В разделе &quot;Имя&quot; выделен элемент Synchronize Azure Active Directory Users to CiscoSpark (Cинхронизировать пользователей Azure Active Directory с CiscoSpark)." border="false":::

11. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы с Cisco Webex из Azure AD. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Cisco Webex для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    :::image type="content" source="./media/cisco-webex-provisioning-tutorial/usermappingattributes.png" alt-text="Снимок экрана: раздел &quot;Сопоставления атрибутов&quot;, в котором показаны атрибуты Azure Active Directory, соответствующие атрибуты CiscoSpark и состояние сопоставления." border="false":::

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Cisco Webex, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей или группы для подготовки в Cisco Webex, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения синхронизации и перейти по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Cisco Webex.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* Приложение Cisco Webex компании Cisco сейчас находится на стадии раннего тестирования (EFT). Для получения дополнительной информации обратитесь в [группу поддержки Cisco](https://www.webex.co.in/support/support-overview.html). 
* Дополнительные сведения о конфигурации Cisco Webex см. в документации Cisco [здесь](https://help.webex.com/en-us/aumpbz/Synchronize-Azure-Active-Directory-Users-into-cisco-webex-Control-Hub).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
