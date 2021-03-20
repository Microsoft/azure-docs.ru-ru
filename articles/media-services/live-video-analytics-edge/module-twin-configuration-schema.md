---
title: Схема JSON двойникаа модуля — Azure
description: В этом разделе описывается схема JSON двойника модуля Live Video Analytics на IoT Edge.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 8bd86bdc2c8de9ee586e785db2074fa772100420
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87053069"
---
# <a name="module-twin-json-schema"></a>Схема JSON двойника модуля

Двойникови устройств — это документы JSON, в которых хранятся сведения о состоянии устройства, включая метаданные, конфигурации и условия. Центр Интернета вещей Azure сохраняет двойник устройства для каждого устройства, подключаемого к Центру Интернета вещей. Подробное описание см. [в статье изучение и использование модуля двойников в центре Интернета вещей](../../iot-hub/iot-hub-devguide-module-twins.md) .

В этом разделе описывается схема JSON двойника модуля Live Video Analytics на IoT Edge.

> [!NOTE]
> Чтобы получить разрешение для доступа к ресурсам служб мультимедиа и API служб мультимедиа, вы должны пройти аутентификацию. Дополнительные сведения см. [в статье доступ к API служб мультимедиа Azure](../latest/media-services-apis-overview.md#accessing-the-azure-media-services-api).

## <a name="module-twin-properties"></a>Свойства двойника модуля

Интерактивная аналитика видео на IoT Edge предоставляет следующие свойства модуля двойника. 

|Свойство |Обязательно |Динамический |Описание |
|---|---|---|---|
|аппликатиондатадиректори |Да |Нет |Путь к подключенному тому для сохранения конфигурации. |
|азуремедиасервицесармид |Да |Нет |Уникальный идентификатор управления ресурсами Azure для учетной записи служб мультимедиа.|
|aadTenantId |Да |Нет |Идентификатор клиента Azure AD.|
|aadServicePrincipalAppId |Да |Да |Пользователь создал идентификатор Azure AD.|
|аадсервицепринЦипалцертификате |Да<sup>*</sup>  |Да |Клиент создал сертификат AppId Azure AD.|
|аадсервицепринЦипалпассворд |Да<sup>*</sup>  |Да |Пользователь создал пароль для AppId Azure AD.|
|аадендпоинт |Нет |Нет |Конечная точка Azure AD, относящаяся к облаку. <br/>По умолчанию: `https://login.microsoftonline.com` |
|aadResourceId |Нет |Нет |Идентификатор аудитории или ресурса Azure AD, относящийся к облаку <br/>По умолчанию: `https://management.core.windows.net/` |
|armEndpoint |Нет |Нет |Конечная точка управления ресурсами Azure, зависящая от облака. <br/>По умолчанию: `https://management.azure.com/` |
|диагностикслевел |Нет |Да |Уровень детализации событий: <br/>Сведения &#x02758; &#x02758 предупреждений; Ошибка &#x02758; Критические &#x02758; None |
|диагностиксевентсаутпутнаме |Нет |Да |Выходные данные концентратора для событий диагностики. <br/>(Пусто означает, что диагностика не публикуется)|
|оператионалевентсаутпутнаме|Нет|Да|Выходные данные концентратора для рабочих событий.<br/>(Пустое означает, что операционные события не публикуются.)
|LogLevel|Нет|Да|Это может быть: <br/>&#x000B7; Verbose<br/>&#x000B7; Сведения (по умолчанию)<br/>&#x000B7; !<br/>&#x000B7; План<br/>&#x000B7; None|
|логкатегориес|Нет|Да|Разделенный запятыми список следующих элементов: Application, Медиапипелине, Events <br/>По умолчанию: приложение, события|
|дебуглогсдиректори|Нет|Да|Каталог для журналов отладки. Если эти журналы созданы, при отсутствии отладочных журналов отключаются.

<sup>*</sup>НЕОБХОДИМО указать сертификат или пароль субъекта-службы. 

Динамические свойства можно обновлять без перезапуска модуля. Чтобы получить значения для некоторых из этих свойств, выполните инструкции из статьи [Получение доступа к API служб мультимедиа](../latest/access-api-howto.md) . 

Дополнительные сведения о роли необязательных параметров диагностики см. в статье [мониторинг и ведение журнала](monitoring-logging.md) .

```
{ 
    "properties.desired": { 
        // Required 
        "applicationDataDirectory": "/var/lib/azuremediaservices", 
        "azureMediaServicesArmId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<rgname>/providers/microsoft.media/mediaservices/<ams_account>", 
        "aadTenantId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalAppId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalPassword": "{Service principal password}", 

        // Optional API Access 
        "aadEndpoint": "https://<aad-endpoint>", 
        "aadResourceId": "https://management.core.windows.net/", 
        "armEndpoint": "https://management.azure.com/", 
        
        // Optional Diagnostics 
        "diagnosticsEventsOutputName": "lvaEdgeDiagnostics",
        "operationalEventsOutputName": "lvaEdgeOperational",
        "logLevel": "Information",
        "logCategories": "Application,Events"
    } 
} 
```

## <a name="next-steps"></a>Дальнейшие действия

[Прямые методы](direct-methods.md)
