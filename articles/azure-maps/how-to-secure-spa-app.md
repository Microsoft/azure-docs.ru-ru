---
title: Как защитить одностраничное приложение с помощью неинтерактивного входа
titleSuffix: Azure Maps
description: Как настроить одностраничное приложение с неинтерактивным управлением доступом на основе ролей Azure (Azure RBAC) и Azure Maps веб-пакетом SDK.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: 9d2af0bf731ab069a8512cb10feccf5ba18d3fa0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101092725"
---
# <a name="how-to-secure-a-single-page-application-with-non-interactive-sign-in"></a>Как защитить одностраничное приложение с помощью неинтерактивного входа

Следующее руководством относится к приложению, которое использует Azure Active Directory (Azure AD) для предоставления маркера доступа для Azure Maps приложений, когда пользователь не может войти в Azure AD. Для этого потока требуется размещение веб-службы, которая должна быть защищена только для доступа к веб-приложению с одной страницей. Существует несколько реализаций, которые могут выполнить проверку подлинности в Azure AD. В этом руководством для получения маркеров доступа используется продукт, функция Azure.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

> [!Tip]
> Карты Azure могут поддерживать маркеры доступа от пользователей, которые входят в интерактивные потоки. Интерактивные потоки обеспечивают более ограниченную область доступа и управление секретами.

## <a name="create-azure-function"></a>Создание Функций Azure

Создайте защищенное приложение веб-службы, которое отвечает за проверку подлинности в Azure AD. 

1. Создайте функцию в портал Azure. Дополнительные сведения см. в статье [Создание функции Azure](../azure-functions/functions-get-started.md).

2. Настройте политику CORS для функции Azure, чтобы она была доступна для веб-приложения с одной страницей. Это обеспечит защиту клиентов браузера от разрешенных источников веб-приложения. См. раздел [Добавление функции CORS](../app-service/app-service-web-tutorial-rest-api.md#add-cors-functionality).

3. Добавьте в функцию Azure [удостоверение, назначенное системой](../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity) , чтобы разрешить создание субъекта-службы для проверки подлинности в Azure AD.  

4. Предоставить учетной записи Azure Maps доступ на основе ролей для назначенного системой удостоверения. Дополнительные сведения см. в разделе [предоставление доступа на основе ролей](#grant-role-based-access) .

5. Напишите код для функции Azure, чтобы получить Azure Maps маркеры доступа, используя назначенное системой удостоверение с одним из поддерживаемых механизмов или протоколом RESTFUL. См. статью [получение маркеров для ресурсов Azure](../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity) .

    Пример протокола RESTFUL:

    ```http
    GET /MSI/token?resource=https://atlas.microsoft.com/&api-version=2019-08-01 HTTP/1.1
    Host: localhost:4141
    ```

    Пример ответа:

    ```http
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "access_token": "eyJ0eXAi…",
        "expires_on": "1586984735",
        "resource": "https://atlas.microsoft.com/",
        "token_type": "Bearer",
        "client_id": "..."
    }
    ```

6. Настройка безопасности для функции Azure HttpTrigger

   * [Создание ключа доступа функции](../azure-functions/functions-bindings-http-webhook-trigger.md?tabs=csharp#authorization-keys)
   * [Защищенная конечная точка HTTP](../azure-functions/functions-bindings-http-webhook-trigger.md?tabs=csharp#secure-an-http-endpoint-in-production) для функции Azure в рабочей среде.
   
7. Настройте веб-пакет SDK для Azure Maps Web Application. 

    ```javascript
    //URL to custom endpoint to fetch Access token
    var url = 'https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?code=<API_KEY>';

    var map = new atlas.Map('myMap', {
                center: [-122.33, 47.6],
                zoom: 12,
                language: 'en-US',
                view: "Auto",
            authOptions: {
                authType: "anonymous",
                clientId: "<insert>", // azure map account client id
                getToken: function(resolve, reject, map) {
                    fetch(url).then(function(response) {
                        return response.text();
                    }).then(function(token) {
                        resolve(token);
                    });
                }
            }
        });

        // use the following events to debug, you can remove them at any time.
        map.events.add("tokenacquired", function () {
            console.log("token acquired");
        });
        map.events.add("error", function (err) {
            console.log(JSON.stringify(err.error));
        });
    ```

## <a name="grant-role-based-access"></a>Предоставление доступа на основе ролей

Вы предоставляете доступ к *управлению доступом на основе ролей Azure (Azure RBAC)* , назначив назначенное системой удостоверение одному или нескольким определениям ролей Azure. Чтобы просмотреть определения ролей Azure, доступные для Azure Maps, перейдите в раздел **Управление доступом (IAM)**. Выберите **роли**, а затем найдите роли, которые начинаются с *Azure Maps*.

1. Перейдите к **учетной записи Azure Maps**. Выберите назначение ролей **управления доступом (IAM)**  >  .

    > [!div class="mx-imgBorder"]
    > ![Предоставление доступа с помощью Azure RBAC](./media/how-to-manage-authentication/how-to-grant-rbac.png)

2. На вкладке **назначения ролей** в разделе **роль** выберите встроенное определение роли Azure Maps, например **Azure Maps модуль чтения данных** или **участник данных Azure Maps**. В разделе **назначение доступа к** выберите **приложение-функция**. Выберите участника по имени. Затем нажмите кнопку **Save** (Сохранить).

   * См. Дополнительные сведения о [назначении ролей Azure](../role-based-access-control/role-assignments-portal.md).

> [!WARNING]
> Azure Maps определения встроенных ролей предоставляют очень большие разрешения на доступ к множеству Azure Maps интерфейсов API. Чтобы ограничить доступ через API к минимуму, см. раздел [Создание пользовательского определения роли и назначение назначенного системой удостоверения](../role-based-access-control/custom-roles.md) пользовательскому определению роли. Это обеспечит минимальный уровень привилегий, необходимых приложению для доступа к Azure Maps.

## <a name="next-steps"></a>Дальнейшие действия

Дальнейшее понимание сценария одностраничного приложения:
> [!div class="nextstepaction"]
> [Одностраничное приложение](../active-directory/develop/scenario-spa-overview.md)

Найдите метрики использования API для учетной записи Azure Maps:
> [!div class="nextstepaction"]
> [Просмотр метрик использования](how-to-view-api-usage.md)

Ознакомьтесь с другими примерами, в которых показано, как интегрировать Azure AD с Azure Maps.
> [!div class="nextstepaction"]
> [Примеры Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ClientGrant)