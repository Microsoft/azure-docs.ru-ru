---
title: API отправки Центра партнеров для подключения приложений Azure в коммерческом магазине Майкрософт
description: Ознакомьтесь с предварительными требованиями для использования API отправки Центра партнеров для приложений Azure в коммерческом магазине в Центре партнеров Майкрософт.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 12/10/2019
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: 9863ed24da9e427f885a4794bda7e103b0c1cc8e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96455457"
---
# <a name="partner-center-submission-api-to-onboard-azure-apps-in-partner-center"></a>API отправки Центра партнеров для подключения приложений Azure в Центре партнеров

Используйте *API отправки Центра партнеров* для запроса, создания отправок и публикации предложений Azure программным способом.  Этот API полезен, если вы используете свою учетную запись для управления несколькими предложениями и хотите автоматизировать и оптимизировать процесс отправки для этих предложений.

## <a name="api-prerequisites"></a>Предварительные требования для API

Для использования API Центра партнеров для продуктов Azure требуется несколько программных ресурсов: 

- приложение Azure Active Directory;
- маркер доступа Azure Active Directory (Azure AD).

### <a name="step-1-complete-prerequisites-for-using-the-partner-center-submission-api"></a>Шаг 1. Выполнение предварительных требований для использования API отправки Центра партнеров

Прежде чем приступить к написанию кода для вызова API отправки Центра партнеров, убедитесь, что выполнены следующие предварительные требования.

- У вас (или вашей организации) должен иметься каталог Azure AD, а также у вас должен быть доступ уровня [глобального администратора](../../active-directory/roles/permissions-reference.md) к этому каталогу. Если вы уже используете Microsoft 365 или другие бизнес-службы Майкрософт, у вас уже есть каталог Azure AD. В противном случае можно [создать новую службу Azure AD в Центре партнеров](/windows/uwp/publish/associate-azure-ad-with-partner-center#create-a-brand-new-azure-ad-to-associate-with-your-partner-center-account) совершенно бесплатно.

- Необходимо [связать приложение Azure AD с учетной записью Центра партнеров](/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services#associate-an-azure-ad-application-with-your-windows-partner-center-account) и получить идентификатор арендатора, идентификатор и ключ клиента. Эти значения необходимы для получения маркера доступа Azure AD, который будет использоваться в вызовах к API отправки Microsoft Store.

#### <a name="how-to-associate-an-azure-ad-application-with-your-partner-center-account"></a>Порядок связывания приложения Azure AD с учетной записью Центра партнеров

Чтобы использовать API отправки Microsoft Store, необходимо связать приложение Azure AD с учетной записью Центра партнеров, получить идентификатор арендатора и идентификатор клиента для приложения и создать ключ. Приложение Azure AD представляет приложение или службу, из которых вы хотите вызвать API отправки Центра партнеров. Для получения маркера доступа Azure AD, передаваемого в API, необходимы идентификатор арендатора, идентификатор и ключ клиента.

>[!Note]
>Это действие выполняется только один раз. После получения идентификатора арендатора, идентификатора и ключа клиента их можно повторно использовать в любой момент, когда потребуется создать новый маркер доступа Azure AD.

1. В Центре партнеров [свяжите учетную запись Центра партнеров своей организации с каталогом Azure AD организации](/windows/uwp/publish/associate-azure-ad-with-partner-center).
1. Затем на странице **Пользователи** в разделе **Параметры учетной записи** в Центре партнеров [добавьте приложение Azure AD](/windows/uwp/publish/add-users-groups-and-azure-ad-applications#add-azure-ad-applications-to-your-partner-center-account), представляющее приложение или службу, которые будут использоваться для доступа к отправкам для учетной записи Центра партнеров. Убедитесь, что этому приложению назначена роль **Менеджер**. Если приложение еще не существует в каталоге Azure AD, можно [создать новое приложение Azure AD в Центре партнеров](/windows/uwp/publish/add-users-groups-and-azure-ad-applications#create-a-new-azure-ad-application-account-in-your-organizations-directory-and-add-it-to-your-partner-center-account).
1. Вернитесь на страницу **Пользователи**, щелкните имя приложения Azure AD, чтобы перейти к параметрам приложения, и скопируйте **идентификатор арендатора** и **идентификатор клиента**.
1. Щелкните **Добавить новый ключ**. На следующем экране скопируйте значение в поле **Ключ**. Покинув эту страницу, вы больше не сможете получить доступ к этим сведениям. Дополнительные сведения см. в разделе [Управление ключами для приложения Azure AD](/windows/uwp/publish/add-users-groups-and-azure-ad-applications#manage-keys).

### <a name="step-2-obtain-an-azure-ad-access-token"></a>Шаг 2. Получение маркера доступа Azure AD

Прежде чем вызывать какой-либо метод в API отправки Центра партнеров, сначала необходимо получить маркер доступа Azure AD, который передается в заголовок **авторизации** каждого метода в API. После получения маркера доступа у вас будет 60 минут, чтобы использовать его до истечения срока действия. По истечении срока действия маркер можно обновить, чтобы его можно было использовать в последующих вызовах API.

Чтобы получить маркер доступа, следуйте инструкциям из статьи [Вызовы служб с помощью учетных данных клиента](../../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) для отправки `HTTP POST` в конечную точку `https://login.microsoftonline.com/<tenant_id>/oauth2/token`. Пример запроса:

JSONCopy
```Json
POST https://login.microsoftonline.com/<tenant_id>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

grant_type=client_credentials
&client_id=<your_client_id>
&client_secret=<your_client_secret>
&resource= https://api.partner.microsoft.com
```

Для параметров *tenant_id* в `POST URI` и *client_id* и *client_secret* укажите идентификатор арендатора, идентификатор и ключ клиента для приложения, полученные из Центра партнеров в предыдущем разделе. Для параметра *resource* укажите значение `https://api.partner.microsoft.com`.

### <a name="step-3-use-the-microsoft-store-submission-api"></a>Шаг 3. Использование API отправки в Microsoft Store

После получения маркера доступа Azure AD можно вызвать методы в API отправки Центра партнеров. Для создания или обновления отправок обычно вызывается несколько методов в API отправки Центра партнеров в определенном порядке. Сведения о каждом сценарии и синтаксисе каждого метода см. в разделе, посвященном API приема данных Swagger.

https://apidocs.microsoft.com/services/partneringestion/

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте подробнее о [создании технического ресурса виртуальной машины Azure](../create-azure-container-technical-assets.md)
* Узнайте подробнее о [создании предложения контейнера Azure](../create-azure-container-offer.md)