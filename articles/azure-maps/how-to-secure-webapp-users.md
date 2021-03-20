---
title: Как защитить веб-приложение с помощью интерактивного единого входа
titleSuffix: Azure Maps
description: Как настроить веб-приложение, которое поддерживает единый вход Azure AD в Azure Maps веб-пакет SDK с помощью протокола OpenID Connect Connect.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: ebdc4b219e0840c18e6bef8ebfe9b8eefa8faf3b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895582"
---
# <a name="secure-a-web-application-with-user-sign-in"></a>Защита веб-приложения с помощью входа пользователя

Следующее руководством относится к приложению, размещенному на веб-серверах, поддерживает несколько бизнес-сценариев и развертывается на веб-серверах. Приложение имеет требование предоставить защищенные ресурсы, защищенные только пользователями Azure AD. Цель сценария заключается в том, чтобы разрешить веб-приложению проходить проверку подлинности в Azure AD и вызывать Azure Maps интерфейсы API-интерфейса RESTFUL от имени пользователя.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Создание регистрации приложения в Azure AD

Чтобы пользователи могли выполнять вход, необходимо создать веб-приложение в Azure AD. Это веб-приложение будет делегировать доступ пользователей к Azure Maps интерфейсам API-интерфейса RESTFUL.

1. В портал Azure в списке служб Azure выберите **Azure Active Directory**  >  **Регистрация приложений**  >  **Новая регистрация**.  

    > [!div class="mx-imgBorder"]
    > ![Интеграция приложений с Azure Active Directory](./media/how-to-manage-authentication/app-registration.png)

2. Введите **имя**, выберите **тип учетной записи поддержки**, укажите универсальный код ресурса (URI) перенаправления, который будет представлять URL-адрес, по которому Azure AD будет выдавать маркер, а — URL-адрес, по которому размещается элемент управления картой. Дополнительные сведения см. в статье сценарий Azure AD [: веб-приложение, которое входит в систему пользователей](../active-directory/develop/scenario-web-app-sign-user-overview.md). Выполните указанные действия в сценарии Azure AD.  

3. После завершения регистрации приложения убедитесь, что вход в приложение работает для пользователей. После входа в систему приложению можно предоставить делегированный доступ к Azure Maps интерфейсам API.
    
4.  Чтобы назначить разрешения делегированного API для Azure Maps, перейдите в приложение. Затем выберите **разрешения API**  >  **Добавить разрешение**. В разделе API, которые **использует Моя организация**, найдите и выберите **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Добавить разрешения API приложения](./media/how-to-manage-authentication/app-permissions.png)

5. Установите флажок для **доступа к Azure Maps**, а затем выберите **Добавить разрешения**.

    > [!div class="mx-imgBorder"]
    > ![Выбор разрешений API приложения](./media/how-to-manage-authentication/select-app-permissions.png)

6. Разрешите веб-приложению вызывать Azure Maps интерфейсы API RESTFUL, настроив регистрацию приложения с помощью секрета приложения. подробные инструкции см. [в веб-приложении, которое вызывает веб-API: регистрация приложения](../active-directory/develop/scenario-web-app-call-api-app-registration.md). Для проверки подлинности в Azure AD от имени пользователя требуется секрет. Сертификат регистрации приложения или секрет должен храниться в безопасном хранилище, чтобы веб-приложение было извлечено для проверки подлинности в Azure AD. 
   
   * Если приложение уже настроило регистрацию приложения Azure AD и секретный код, этот шаг можно пропустить.

> [!Tip]
> Если приложение размещено в среде Azure, мы рекомендуем использовать [управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md) и экземпляр Azure Key Vault для доступа к секретам, [получая маркер доступа](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) для доступа к Azure Key Vault секретам или сертификатам. Чтобы подключиться к Azure Key Vault для получения секретов, см. [руководство по подключению через управляемое удостоверение](../key-vault/general/tutorial-net-create-vault-azure-web-app.md).
   
7. Реализуйте конечную точку безопасного маркера для Azure Maps веб-пакета SDK, чтобы получить доступ к маркеру. 
   
   * Пример контроллера маркеров см. в разделе [Azure Maps примеры Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/blob/master/src/OpenIdConnect/AzureMapsOpenIdConnectv1/AzureMapsOpenIdConnect/Controllers/TokenController.cs). 
   * Сведения о реализации, отличной от AspNetCore, см. в статье [Получение маркера для приложения](../active-directory/develop/scenario-web-app-call-api-acquire-token.md) из документации по Azure AD.
   * Защищенная конечная точка токена отвечает за возврат маркера доступа для пользователя, прошедшего проверку подлинности и уполномоченного, для вызова Azure Maps интерфейсов API.

8. Настройте управление доступом на основе ролей Azure (Azure RBAC) для пользователей или групп. См. раздел [предоставление пользователям доступа на основе ролей](#grant-role-based-access-for-users-to-azure-maps).

9. Настройте страницу веб-приложения с помощью веб-пакета SDK Azure Maps для доступа к конечной точке защищенного токена. 

```javascript
var map = new atlas.Map("map", {
        center: [-122.33, 47.64],
        zoom: 12,
        language: "en-US",
        authOptions: {
            authType: "anonymous",
            clientId: "<insert>",  // azure map account client id
            getToken: function (resolve, reject, map) {
                var xhttp = new XMLHttpRequest();
                xhttp.open("GET", "/api/token", true); // the url path maps to the token endpoint.
                xhttp.onreadystatechange = function () {
                    if (this.readyState === 4 && this.status === 200) {
                        resolve(this.responseText);
                    } else if (this.status !== 200) {
                        reject(this.responseText);
                    }
                };

                xhttp.send();
            }
        }
    });
    map.events.add("tokenacquired", function () {
        console.log("token acquired");
    });
    map.events.add("error", function (err) {
        console.log(JSON.stringify(err.error));
    });
```

[!INCLUDE [grant role-based access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о сценарии веб-приложений:
> [!div class="nextstepaction"]
> [Scenario: Веб-приложение, выполняющее вход пользователей](../active-directory/develop/scenario-web-app-sign-user-overview.md)

Найдите метрики использования API для учетной записи Azure Maps:
> [!div class="nextstepaction"]
> [Просмотр метрик использования](how-to-view-api-usage.md)

Просмотрите примеры, демонстрирующие интеграцию Azure AD с Azure Maps:
> [!div class="nextstepaction"]
> [Примеры веб-приложения Azure Maps Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/OpenIdConnect)