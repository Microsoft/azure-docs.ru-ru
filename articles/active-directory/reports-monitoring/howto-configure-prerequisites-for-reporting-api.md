---
title: Предварительные требования для API отчетов Azure Active Directory Документация Майкрософт
description: Узнайте о предварительных требованиях для доступа к API отчетов Azure AD
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: ada19f69-665c-452a-8452-701029bf4252
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 03/04/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4ca618dc234ff4383083ff5eb21299fb1e3e7128
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96348744"
---
# <a name="prerequisites-to-access-the-azure-active-directory-reporting-api"></a>Предварительные требования для доступа к API отчетов Azure Active Directory

[API-интерфейсы отчетов Azure Active Directory (Azure AD)](./concept-reporting-api.md) предоставляют программный доступ к данным с помощью набора API-интерфейсов на базе REST. Эти API можно вызывать из языков программирования и средств.

API отчетов использует [OAuth](../../api-management/api-management-howto-protect-backend-with-aad.md) для авторизации доступа к веб-API.

Чтобы подготовить доступ к API отчетов, сделайте следующее:

1. [Назначение ролей](#assign-roles)
2. [Требования к лицензии](#license-requirements)
3. [Регистрация приложения](#register-an-application)
4. [Предоставление разрешений](#grant-permissions)
5. [Сбор параметров конфигурации](#gather-configuration-settings)

## <a name="assign-roles"></a>Назначение ролей

Чтобы получить доступ к данным отчетов через API, необходима одна из следующих ролей:

- Читатель сведений о безопасности

- Администратор безопасности

- глобальный администратор

## <a name="license-requirements"></a>Требования к лицензии

Чтобы получить доступ к отчетам о входе для клиента, клиент Azure AD должен иметь связанную лицензию Azure AD Premium. Для доступа к отчетам о входе для любого клиента Azure AD требуется лицензия Azure AD Premium P1 (или выше). Кроме того, если выбран тип каталога Azure AD B2C, отчеты о входе доступны через API без каких-либо дополнительных требований к лицензии. 


## <a name="register-an-application"></a>Регистрация приложения

Регистрация необходима, даже если доступ к API отчетов осуществляется с помощью скрипта. Регистрация предоставляет **идентификатор приложения**, который необходим для вызовов авторизации и позволяет коду принимать маркеры.

Чтобы настроить для каталога доступ к API отчетов AAD, необходимо войти на [портал Azure](https://portal.azure.com) с учетной записью администратора Azure, которой также назначена роль участника каталога **глобального администратора** в клиенте AAD.

> [!IMPORTANT]
> В приложениях, запущенных от имени учетных данных с привилегиями администратора, доступно много возможностей. Поэтому храните в надежном месте идентификатор приложения и секретные учетные данные.
> 

**Чтобы зарегистрировать приложение AAD, сделайте следующее:**

1. На [портале Azure](https://portal.azure.com) в области навигации слева щелкните **Azure Active Directory**.
   
    ![Снимок экрана показывает Azure Active Directory, выбранные в меню портал Azure.](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. На странице **Azure Active Directory** щелкните **Регистрация приложений**.

    ![Снимок экрана показывает Регистрация приложений, выбранные в меню "Управление".](./media/howto-configure-prerequisites-for-reporting-api/02.png) 

3. На странице **Регистрация приложений** выберите пункт **Новая регистрация**.

    ![На снимке экрана показана выбранная Новая регистрация.](./media/howto-configure-prerequisites-for-reporting-api/03.png)

4. Страница **регистрации приложения** :

    ![На снимке экрана показана страница Регистрация приложения, на которой можно ввести значения на этом шаге.](./media/howto-configure-prerequisites-for-reporting-api/04.png)

    а. В текстовом поле **Имя** введите `Reporting API application`.

    b. Для **поддерживаемого типа учетных записей** выберите **учетные записи только в этом Организации**.

    c. В **текстовом поле** **URL-адрес перенаправления** выберите введите `https://localhost` .

    d. Выберите **Зарегистрировать**. 


## <a name="grant-permissions"></a>Предоставить разрешения 

В зависимости от API, к которому вы хотите получить доступ, приложению необходимо предоставить следующие разрешения:  

| API | Разрешение |
| --- | --- |
| Microsoft Azure Active Directory | Чтение данных каталога |
| Microsoft Graph | Прочитать все данные журнала аудита |

![На снимке экрана показано, где можно выбрать пункт Добавить разрешение в области разрешений P I.](./media/howto-configure-prerequisites-for-reporting-api/36.png)

В следующем разделе перечислены шаги для получения доступа к обоим API. Если вам не нужен доступ к данным API, вы можете пропустить эти шаги.

**Вот как можно предоставить приложению разрешения на использование API.**


1. Выберите **разрешения API** и **добавьте разрешение**. 

    ![На снимке экрана показана страница разрешений P I, на которой можно выбрать команду Добавить разрешение.](./media/howto-configure-prerequisites-for-reporting-api/05.png)

2. На **странице разрешения API запроса** выберите **Поддержка устаревших API** **Azure Active Directory Graph**. 

    ![На снимке экрана показана страница запроса разрешений P I, на которой можно выбрать Azure Active Directory граф.](./media/howto-configure-prerequisites-for-reporting-api/06.png)

3. На странице **необходимые разрешения** выберите **разрешения приложения**, а затем — **флажок** каталог **. ReadAll**.  Выберите **Добавить разрешения**.

    ![На снимке экрана показана страница запрос разрешений P I, на которой можно выбрать разрешения приложения.](./media/howto-configure-prerequisites-for-reporting-api/07.png)

4. На странице **разрешения API приложения API отчетов** выберите **предоставить согласие администратора**. 

    ![На снимке экрана показана страница отчетов с разрешениями p I, где можно выбрать параметр предоставить согласие администратора.](./media/howto-configure-prerequisites-for-reporting-api/08.png)

5. Примечание. **Microsoft Graph** добавляется по умолчанию во время регистрации API.

    ![На снимке экрана показана страница разрешений P I, на которой можно выбрать команду Добавить разрешение.](./media/howto-configure-prerequisites-for-reporting-api/15.png)

## <a name="gather-configuration-settings"></a>Сбор параметров конфигурации 

В этом разделе показано, как получить из каталога следующие параметры:

- Доменное имя
- Идентификатор клиента
- Секрет клиента

Эти значения необходимы при настройке вызовов API отчетов. 

### <a name="get-your-domain-name"></a>Получение имени домена

**Вот как можно получить доменное имя.**

1. На [портале Azure](https://portal.azure.com) в области навигации слева выберите **Azure Active Directory**.
   
    ![Снимок экрана показывает Azure Active Directory, выбранные в меню портал Azure.](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. На странице **Azure Active Directory** выберите **Имена пользовательских доменов**.

    ![На снимке экрана показаны пользовательские имена доменов, выбранные из Azure Active Directory.](./media/howto-configure-prerequisites-for-reporting-api/09.png) 

3. Скопируйте нужное доменное имя из списка доменов.


### <a name="get-your-applications-client-id"></a>Получение идентификатора клиента приложения

**Вот как можно получить идентификатор клиента приложения.**

1. На [портале Azure](https://portal.azure.com) в области навигации слева щелкните **Azure Active Directory**.
   
    ![Снимок экрана показывает Azure Active Directory, выбранные в меню портал Azure.](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. Выберите свое приложение на странице **Регистрация приложений**.

3. Со страницы приложения перейдите к **идентификатору приложения** и выберите **Щелкните, чтобы скопировать**.

    ![На снимке экрана показана страница отчетов приложение P I, на которой можно скопировать приложение I D.](./media/howto-configure-prerequisites-for-reporting-api/11.png) 


### <a name="get-your-applications-client-secret"></a>Получение секрета клиента приложения
 Избегайте ошибок при попытке доступа к журналам аудита или входу с помощью API.

**Вот как можно получить секрет клиента приложения.**

1. На [портале Azure](https://portal.azure.com) в области навигации слева щелкните **Azure Active Directory**.
   
    ![Снимок экрана показывает Azure Active Directory, выбранные в меню портал Azure.](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2.  Выберите свое приложение на странице **Регистрация приложений**.

3.  На странице **приложение API** выберите **Сертификаты и секреты** , в разделе **секреты клиента** щелкните **+ новый секрет клиента**. 

    ![На снимке экрана показана страница сертификаты & секреты, на которой можно добавить секрет клиента.](./media/howto-configure-prerequisites-for-reporting-api/12.png)

5. На странице **Добавление секрета клиента** добавьте:

    а. В текстовом поле **Описание** введите `Reporting API`.

    b. Для параметра **Срок действия истекает** выберите значение **Через 2 года**.

    c. Выберите команду **Сохранить**.

    d. Скопируйте значение ключа.

## <a name="troubleshoot-errors-in-the-reporting-api"></a>Устранение ошибок в API отчетов

В этом разделе перечислены распространенные сообщения об ошибках, с которыми можно столкнуться при доступе к отчетам о действиях с помощью Microsoft Graph API и действиях по их устранению.

### <a name="error-failed-to-get-user-roles-from-microsoft-graph"></a>Ошибка: не удалось получить роли пользователя из Microsoft Graph

 Войдите в свою учетную запись, используя кнопки входа в пользовательском интерфейсе проводника Graph, чтобы избежать ошибки при попытке входа с помощью Graph Explorer. 

![Песочница Graph](./media/troubleshoot-graph-api/graph-explorer.png)

### <a name="error-failed-to-do-premium-license-check-from-microsoft-graph"></a>Ошибка: не удалось проверить лицензию Premium с Microsoft Graph 

Если при попытке обращения ко входам с помощью песочницы Graph появляется это сообщение об ошибке, выберите **Изменить разрешения** под своей учетной записью на левой панели навигации, а затем выберите **Tasks.ReadWrite** и **Directory.Read.All**. 

![Пользовательский интерфейс для изменения разрешений](./media/troubleshoot-graph-api/modify-permissions.png)

### <a name="error-tenant-is-not-b2c-or-tenant-doesnt-have-premium-license"></a>Ошибка: клиент не B2C, или у клиента нет лицензии Premium

Для доступа к отчетам о входе требуется лицензия Azure Active Directory Premium 1 (P1). Если вы видите это сообщение об ошибке при обращении ко входам, убедитесь, что клиент лицензируется по лицензии Azure AD P1.

### <a name="error-the-allowed-roles-does-not-include-user"></a>Ошибка: разрешенные роли не включают пользователя. 

 Избегайте ошибок при попытке доступа к журналам аудита или входу с помощью API. Убедитесь, что ваша учетная запись является частью **средства чтения безопасности** или роли **читателя отчетов** в клиенте Azure Active Directory.

### <a name="error-application-missing-aad-read-directory-data-permission"></a>Error: Application missing AAD 'Read directory data' permission (Ошибка: у приложения нет разрешения "Чтение данных каталога" AAD) 

### <a name="error-application-missing-microsoft-graph-api-read-all-audit-log-data-permission"></a>Ошибка: в приложении отсутствует разрешение на чтение всех данных журнала аудита Microsoft Graph API

Выполните действия, описанные в разделе [необходимые условия для доступа к API отчетов Azure Active Directory](howto-configure-prerequisites-for-reporting-api.md) , чтобы убедиться, что приложение работает с нужным набором разрешений. 

## <a name="next-steps"></a>Дальнейшие действия

* [Получение данных с помощью API отчетов Azure Active Directory с сертификатами](tutorial-access-api-with-certificates.md)
* [Справочник по API аудита](/graph/api/resources/directoryaudit?view=graph-rest-beta) 
* [Справочник по API отчетов о действиях при входе](/graph/api/resources/signin?view=graph-rest-beta)