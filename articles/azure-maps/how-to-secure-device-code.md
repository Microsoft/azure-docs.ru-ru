---
title: Как защитить устройство ввода с ограниченным входом с помощью Azure AD и Azure Maps API-интерфейсов RESTFUL
titleSuffix: Azure Maps
description: Как настроить приложение без браузера, которое поддерживает вход в Azure AD и вызывает Azure Maps интерфейсы API-интерфейса.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 3833cbfd0802f334e482203d269984eb0e299797
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895636"
---
# <a name="secure-an-input-constrained-device-with-azure-ad-and-azure-maps-rest-apis"></a>Защита входного устройства с помощью Azure AD и Azure Maps API-интерфейсов RESTFUL

В этом руководство описано, как защитить общедоступные приложения или устройства, которые не могут безопасно хранить секреты или принимать входные данные браузера. Эти типы приложений относятся к категории Интернета вещей или Интернета вещей. Некоторые примеры таких приложений могут включать в себя: устройства с интеллектуальным ТВ или данные датчика, порожденные приложениями. 

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Создание регистрации приложения в Azure AD

> [!NOTE]
> * **Предварительное чтение:** [сценарий: классическое приложение, вызывающее веб-API](../active-directory/develop/scenario-desktop-overview.md)
> * В следующем сценарии используется поток кода устройства, который не включает в себя веб-браузер для получения маркера.

Создайте приложение на основе устройства в Azure AD, чтобы включить вход в Azure AD. Этому приложению будет предоставлен доступ к Azure Maps интерфейсам API.

1. В портал Azure в списке служб Azure выберите **Azure Active Directory**  >  **Регистрация приложений**  >  **Новая регистрация**.  

    > [!div class="mx-imgBorder"]
    > ![Интеграция приложений с Azure Active Directory](./media/how-to-manage-authentication/app-registration.png)

2. Введите **имя**, выберите **учетные записи в этом каталоге организации только** в качестве **поддерживаемого типа учетной записи**. В списке **URI перенаправления** укажите **общедоступный клиент/машинный код (мобильный & Desktop)** и добавьте `https://login.microsoftonline.com/common/oauth2/nativeclient` к значению. Дополнительные сведения см. в разделе [классическое приложение Azure AD, которое вызывает веб-API: регистрация приложения](../active-directory/develop/scenario-desktop-app-registration.md). Затем **зарегистрируйте** приложение.

    > [!div class="mx-imgBorder"]
    > ![Добавление сведений о регистрации приложения для имени и URI перенаправления](./media/azure-maps-authentication/devicecode-app-registration.png)

3. Перейдите к разделу **Проверка подлинности** и включите параметр **рассматривать приложение как общедоступный клиент**. Это позволит включить проверку подлинности кода устройства в Azure AD.
    
    > [!div class="mx-imgBorder"]
    > ![Включение регистрации приложения в качестве общедоступного клиента](./media/azure-maps-authentication/devicecode-public-client.png)

4.  Чтобы назначить разрешения делегированного API для Azure Maps, перейдите в приложение. Затем выберите **разрешения API**  >  **Добавить разрешение**. В разделе API, которые **использует Моя организация**, найдите и выберите **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Добавить разрешения API приложения](./media/how-to-manage-authentication/app-permissions.png)

5. Установите флажок для **доступа к Azure Maps**, а затем выберите **Добавить разрешения**.

    > [!div class="mx-imgBorder"]
    > ![Выбор разрешений API приложения](./media/how-to-manage-authentication/select-app-permissions.png)

6. Настройте управление доступом на основе ролей Azure (Azure RBAC) для пользователей или групп. См. статью [предоставление пользователям доступа на основе ролей для Azure Maps](#grant-role-based-access-for-users-to-azure-maps).

7. Добавление кода для получения потока токенов в приложении. сведения о реализации см. в разделе [поток кода устройства](../active-directory/develop/scenario-desktop-acquire-token.md#device-code-flow). При получении маркеров сослаться на область, `user_impersonation` которая была выбрана на предыдущих шагах.

> [!Tip]
> Используйте библиотеку проверки подлинности Майкрософт (MSAL) для получения маркеров доступа. См. рекомендации в [классическом приложении, вызывающем веб-API: конфигурация кода](../active-directory/develop/scenario-desktop-app-configuration.md)

8. Составьте HTTP-запрос с полученным маркером из Azure AD и отправил запрос с действительным клиентом HTTP.

### <a name="sample-request"></a>Пример запроса
Ниже приведен пример текста запроса для отправки простого геозоны, представленного в виде круговой геометрии с помощью центральной точки и радиуса.

```http
POST /mapData/upload?api-version=1.0&dataFormat=geojson
Host: atlas.microsoft.com
x-ms-client-id: 30d7cc….9f55
Authorization: Bearer eyJ0e….HNIVN
```

 Образец текста запроса приведен ниже в геоjson:
```json
{
    "type": "FeatureCollection",
    "features": [{
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [-122.126986, 47.639754]
        },
        "properties": {
            "geometryId": "001",
            "radius": 500
        }
    }]
}
```

### <a name="sample-response"></a>Пример ответа:

Headers:
```http
Location: https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0
Access-Control-Expose-Headers: Location
```

Текст:
```json
{
  "operationId": "{operationId}",
  "status": "Succeeded",
  "created": "2020-01-02 1:02:03 AM +00:00",
  "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{resourceId}?api-version=1.0"
}
```

[!INCLUDE [grant role-based access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Дальнейшие действия

Найдите метрики использования API для учетной записи Azure Maps:
> [!div class="nextstepaction"]
> [Просмотр метрик использования](how-to-view-api-usage.md)