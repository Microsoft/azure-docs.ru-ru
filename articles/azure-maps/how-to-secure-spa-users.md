---
title: Как защитить одностраничное приложение с помощью входа пользователя
titleSuffix: Azure Maps
description: Как настроить одностраничное приложение, которое поддерживает единый вход Azure AD в Azure Maps веб-пакет SDK.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: 60d8dc45fb26ea210b1827a6938716474faa0304
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895619"
---
# <a name="secure-a-single-page-application-with-user-sign-in"></a>Защита одностраничного приложения с помощью входа пользователя

Следующее руководством относится к приложению, которое размещено на сервере содержимого или имеет минимальные зависимости веб-сервера. Приложение предоставляет защищенные ресурсы, защищенные только пользователями Azure AD. Цель сценария заключается в том, чтобы разрешить веб-приложению проходить проверку подлинности в Azure AD и вызывать Azure Maps интерфейсы API-интерфейса RESTFUL от имени пользователя.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Создание регистрации приложения в Azure AD

Создайте веб-приложение в Azure AD, чтобы пользователи могли выполнять вход. Веб-приложение делегирует доступ пользователей к Azure Maps интерфейсам API-интерфейса RESTFUL.

1. В портал Azure в списке служб Azure выберите **Azure Active Directory**  >  **Регистрация приложений**  >  **Новая регистрация**.  

    > [!div class="mx-imgBorder"]
    > ![Интеграция приложений с Azure Active Directory](./media/how-to-manage-authentication/app-registration.png)

2. Введите **имя**, выберите **тип учетной записи поддержки**, укажите универсальный код ресурса (URI) перенаправления, который будет представлять URL-адрес, по которому Azure AD будет выдавать маркер, а — URL-адрес, по которому размещается элемент управления картой. Подробный пример см. в [Azure Maps примерах Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ImplicitGrant). Нажмите кнопку **Зарегистрировать**.  

3. Чтобы назначить разрешения делегированного API для Azure Maps, перейдите в приложение. Затем в разделе **Регистрация приложений** выберите **разрешения API**  >  **Добавить разрешение**. В разделе API, которые **использует Моя организация**, найдите и выберите **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Добавить разрешения API приложения](./media/how-to-manage-authentication/app-permissions.png)

4. Установите флажок для **доступа к Azure Maps**, а затем выберите **Добавить разрешения**.

    > [!div class="mx-imgBorder"]
    > ![Выбор разрешений API приложения](./media/how-to-manage-authentication/select-app-permissions.png)

5. Включите `oauth2AllowImplicitFlow`. Чтобы включить его, в разделе **манифеста** регистрации приложения задайте `oauth2AllowImplicitFlow` для значение `true` .

6. Скопируйте идентификатор приложения Azure AD и идентификатор клиента Azure AD из регистрации приложения для использования в веб-пакете SDK. Добавьте сведения о регистрации приложения Azure AD и `x-ms-client-id` из учетной записи сопоставлений Azure в веб-пакет SDK.

    ```javascript
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js" />
        <script>
            var map = new atlas.Map("map", {
                center: [-122.33, 47.64],
                zoom: 12,
                language: "en-US",
                authOptions: {
                    authType: "aad",
                    clientId: "<insert>",  // azure map account client id
                    aadAppId: "<insert>",  // azure ad app registration id
                    aadTenant: "<insert>", // azure ad tenant id
                    aadInstance: "https://login.microsoftonline.com/"
                }
            });
        </script>   
    ```

7. Настройте управление доступом на основе ролей Azure (Azure RBAC) для пользователей или групп. Сведения о [включении Azure RBAC](#grant-role-based-access-for-users-to-azure-maps)см. в следующих разделах.
   
[!INCLUDE [grant role access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дальнейшее понимание сценария одностраничного приложения:
> [!div class="nextstepaction"]
> [Одностраничное приложение](../active-directory/develop/scenario-spa-overview.md)

Найдите метрики использования API для учетной записи Azure Maps:
> [!div class="nextstepaction"]
> [Просмотр метрик использования](how-to-view-api-usage.md)

Просмотрите примеры, демонстрирующие интеграцию Azure AD с Azure Maps:
> [!div class="nextstepaction"]
> [Примеры Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ImplicitGrant)