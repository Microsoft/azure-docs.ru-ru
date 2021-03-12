---
title: Обзор REST API операций служб мультимедиа | Документация Майкрософт
description: REST API операций Служб мультимедиа позволяет создавать задания, ресурсы, динамические каналы и другие компоненты в учетной записи Служб мультимедиа. В этой статье содержатся общие сведения о REST API служб мультимедиа Azure v2.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: a5f1c5e7-ec52-4e26-9a44-d9ea699f68d9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 3/10/2021
ms.author: inhenkel
ms.reviewer: johndeu
ms.openlocfilehash: 9f147e333e4d1b95a14dd3121d7ab304b6166248
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103010054"
---
# <a name="media-services-operations-rest-api-overview"></a>Обзор REST API операций служб мультимедиа

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

**REST API операций Служб мультимедиа** позволяет создавать задания, ресурсы, динамические каналы и другие компоненты в учетной записи Служб мультимедиа. Дополнительные сведения см. в статье [Media Services Operations REST API Reference](/rest/api/media/operations/azure-media-services-rest-api-reference) (Справочник по REST API операций служб мультимедиа).

Службы мультимедиа предоставляют REST API, поддерживающий XML-формат JSON или atom+pub. Этот интерфейс помимо набора дополнительных заголовков требует определенные HTTP-заголовки, которые каждый клиент должен отправлять при подключении к Службам мультимедиа. В следующих разделах описываются заголовки и HTTP-команды, которые можно использовать при создании запросов и получения ответов из служб мультимедиа.

Аутентификация в REST API Служб мультимедиа выполняется через Azure Active Directory. Дополнительные сведения см. в статье [Использование аутентификации Azure AD для доступа к API служб мультимедиа Azure с помощью REST](media-services-rest-connect-with-aad.md).

## <a name="considerations"></a>Рекомендации

При использовании REST следует принимать во внимание следующие соображения.

* При запросе сущностей существует ограничение в 1000 сущностей, возвращаемых за один раз, так как в открытой версии 2 REST количество результатов запросов ограничено 1000. Используйте **Skip** и **Take** (.NET) или **top** (REST), как описано в [этом](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities) и [этом](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities) примерах. 
* Если вы используете JSON и указываете в запросе ключевое слово **__metadata** (например, для ссылки на связанный объект), вы должны задать для заголовка **Accept**[подробный (Verbose) формат JSON](https://www.odata.org/documentation/odata-version-3-0/json-verbose-format/) (см. пример ниже). Odata "не поймет" свойство **__metadata** в запросе, если не задать для него подробный формат.  

    ```console
    POST https://media.windows.net/API/Jobs HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.19
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

    {
        "Name" : "NewTestJob", 
        "InputMediaAssets" : 
            [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aba5356eb-30ff-4dc6-9e5a-41e4223540e7')"}}]
    . . . 
   ```

## <a name="standard-http-request-headers-supported-by-media-services"></a>Стандартные заголовки HTTP-запроса, поддерживаемые службами мультимедиа
Для каждого вызова, отправляемого в службы мультимедиа, предусмотрен набор обязательных заголовков, которые необходимо включить в запрос, а также набор дополнительных заголовков, которые можно включить при необходимости. В таблице ниже перечислены обязательные заголовки.

| Заголовок | Тип | Значение |
| --- | --- | --- |
| Авторизация |Bearer |Носитель – единственный принимаемый механизм авторизации. Значение должно содержать маркер доступа, предоставленный Azure Active Directory. |
| x-ms-version |Decimal |2.17 (или последняя версия)|
| DataServiceVersion |Decimal |3.0 |
| MaxDataServiceVersion |Decimal |3.0 |

> [!NOTE]
> Так как Службы мультимедиа предоставляют интерфейсы REST API с использованием протокола OData, заголовки DataServiceVersion и MaxDataServiceVersion должны быть включены во все запросы. Тем не менее, если они отсутствуют, сейчас в Службах мультимедиа предполагается, что для DataServiceVersion используется значение 3.0.
> 
> 

Ниже приведен набор дополнительных заголовков.

| Заголовок | Тип | Значение |
| --- | --- | --- |
| Дата |RFC 1123 date |Метка времени запроса |
| Принять |Тип содержимого |Запрошенный тип содержимого для ответа, например:<p> application/json;odata=verbose<p> application/atom+xml<p> Ответы могут принадлежать к разным типам содержимого, например, при получении больших двоичных объектов полезными данными в успешном ответе считается поток больших двоичных объектов. |
| Accept-Encoding |Gzip, deflate |Кодировка GZIP и, в соответствующих случаях, DEFLATE. Примечание. Для больших ресурсов службы мультимедиа могут пропустить этот заголовок и вернуть несжатые данные. |
| Accept-Language |"en", "ru" и т. д. |Указывает предпочитаемый язык для ответа. |
| Accept-Charset |Тип кодировки, например "UTF-8" |По умолчанию — UTF-8. |
| X-HTTP-Method |Метод HTTP |Позволяет клиентам или брандмауэрам, которые не поддерживают методы HTTP, например PUT или DELETE, использовать эти методы, туннелированные посредством вызова GET. |
| Content-Type |Тип содержимого |Тип содержимого текста запросов PUT или POST. |
| client-request-id |Строка |Определяемое вызывающим объектом значение, по которому можно идентифицировать данный запрос. Если это значение указано, оно будет включено в ответное сообщении в качестве способа сопоставления запроса. <p><p>**Важно**<p>Значения должны превышать 2096 Б (2 КБ). |

## <a name="standard-http-response-headers-supported-by-media-services"></a>Стандартные ответы HTTP-запроса, поддерживаемые службами мультимедиа
Ниже приведен набор заголовков, которые могут быть возвращены в зависимости от запрошенного ресурса и действия, которое необходимо выполнить.

| Заголовок | Тип | Значение |
| --- | --- | --- |
| request-id |Строка |Уникальный идентификатор текущей операции, создаваемый службой. |
| client-request-id |Строка |Идентификатор, указанный вызывающим объектом в исходном запросе, если он имеется. |
| Дата |RFC 1123 date |Дата и время обработки запроса. |
| Content-Type |Различается |Тип содержимого текста ответа. |
| Content-Encoding |Различается |Соответственно, Gzip или Deflate. |

## <a name="standard-http-verbs-supported-by-media-services"></a>Стандартные HTTP-команды, поддерживаемые службами мультимедиа
Ниже приведен полный список HTTP-команд, которые могут использоваться при создании HTTP-запросов:

| Команда | Описание |
| --- | --- |
| GET |Возвращает текущее значение объекта. |
| POST |Создает объект на основе предоставленных данных или отправляет команду. |
| PUT |Заменяет объект или создает именованный объект (в зависимости от ситуации). |
| DELETE |Удаляет объект. |
| MERGE |Вносит в существующий объект изменения в именованном свойстве. |
| HEAD |Возвращает метаданные объекта для ответа GET. |

## <a name="discover-and-browse-the-media-services-entity-model"></a>Обнаружение и просмотр модели сущностей Служб мультимедиа
Чтобы упростить обнаружение сущностей служб мультимедиа, можно использовать операцию $metadata. Она позволяет получать все допустимые типы сущностей, свойства сущностей, ассоциации, функции, действия и т. д. Добавив операцию $metadata в конец конечной точки REST API Служб мультимедиа, вы можете получить доступ к этой службе обнаружения.

 /api/$metadata.

Чтобы метаданные можно было просмотреть в браузере, в конце универсального кода ресурса необходимо добавить выражение "?api-version=2.x" или не включать в запрос заголовок x-ms-version.

## <a name="authenticate-with-media-services-rest-using-azure-active-directory"></a>Аутентификация в REST Служб мультимедиа с помощью Azure Active Directory

Проверка подлинности в REST API осуществляется через Azure Active Directory.
Дополнительные сведения о том, как получить необходимые параметры проверки подлинности учетной записи Служб мультимедиа на портале Azure, см. в статье [Доступ к API Служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Дополнительные сведения о написании кода, который позволяет подключиться к REST API с помощью проверки подлинности Azure AD, см. в статье [Использование аутентификации Azure AD для доступа к API служб мультимедиа Azure с помощью REST](media-services-rest-connect-with-aad.md).

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как использовать проверку подлинности Azure AD в REST API Служб мультимедиа, изучив сведения из [этой статьи](media-services-rest-connect-with-aad.md).

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
