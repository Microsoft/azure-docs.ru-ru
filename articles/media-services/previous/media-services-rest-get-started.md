---
title: Приступая к работе с доставкой содержимого по запросу с помощью REST | Документация Майкрософт
description: В этом учебнике показаны шаги по реализации приложения доставки содержимого по запросу с помощью служб мультимедиа и API REST
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 88194b59-e479-43ac-b179-af4f295e3780
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: inhenkel
ms.openlocfilehash: ccf4bc18270dcdbcae02f7b45bea2a05ed663227
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103008881"
---
# <a name="get-started-with-delivering-content-on-demand-using-rest"></a>Начало работы с доставкой содержимого по запросу с помощью REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

В этом кратком руководстве приведены пошаговые инструкции по реализации приложения доставки видео по запросу с помощью служб мультимедиа (AMS) Azure с использованием REST API.

В учебнике описывается базовый рабочий процесс служба мультимедиа и наиболее распространенные объекты и задачи программирования, необходимые для разработки с использованием служб мультимедиа. По завершении работы с руководством вы сможете выполнить потоковую передачу и поэтапно загрузить пример файла мультимедиа, который вы отправили, закодировали и скачали.

На следующем рисунке показаны некоторые объекты, которые чаще всего используются при разработке приложений VoD с использованием модели OData служб мультимедиа.

Щелкните изображение, чтобы просмотреть его полноразмерную версию.  

[![Схема, на которой показаны некоторые наиболее часто используемые объекты в модели данных объектов Служб мультимедиа Azure для разработки приложений для видео по запросу.](./media/media-services-rest-get-started/media-services-overview-object-model-small.png)](./media/media-services-rest-get-started/media-services-overview-object-model.png#lightbox)

## <a name="prerequisites"></a>Предварительные требования
Для начала разработки с помощью REST API служб мультимедиа необходимо выполнить следующие предварительные требования.

* Учетная запись Azure. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/).
* Учетная запись служб мультимедиа. Инструкции по созданию учетной записи служб мультимедиа см. в статье [Создание учетной записи служб мультимедиа Azure с помощью портала Azure](media-services-portal-create-account.md).
* Основная информация о разработке с помощью REST API служб мультимедиа. Дополнительные сведения см. в статье [Обзор REST API служб мультимедиа](media-services-rest-how-to-use.md).
* Выбранное вами приложение, способное отправлять HTTP-запросы и ответы. В этом учебнике используется [Fiddler](https://www.telerik.com/download/fiddler).

В этом кратком руководстве показаны следующие задачи.

1. Запуск конечных точек потоковой передачи с помощью портала Azure
2. Подключение к учетной записи служб мультимедиа с помощью REST API.
3. Создание нового ресурса и отправка видеофайла с помощью REST API.
4. Кодирование исходного файла в набор MP4-файлов с адаптивной скоростью с помощью REST API.
5. Публикация актива и получение URL-адресов потоковой передачи и поэтапного скачивания с помощью REST API.
6. Воспроизведение содержимого.

>[!NOTE]
>Действует ограничение в 1 000 000 записей для разных политик AMS (например, для политики Locator или ContentKeyAuthorizationPolicy). Указывайте один и тот же идентификатор политики, если вы используете те же дни, разрешения доступа и т. д., например политики для указателей, которые должны оставаться на месте в течение длительного времени (политики запрета передачи). Дополнительные сведения см. в [этой статье](media-services-dotnet-manage-entities.md#limit-access-policies).

Дополнительные сведения о сущностях AMS REST, используемых в этой статье, см. в [справочнике по REST API служб мультимедиа Azure](/rest/api/media/operations/azure-media-services-rest-api-reference). См. также статью [Основные понятия служб мультимедиа Azure](media-services-concepts.md).

>[!NOTE]
>При доступе к сущностям в службах мультимедиа необходимо задать определенные поля и значения заголовков в HTTP-запросах. Дополнительную информацию см. в статье [Обзор интерфейса REST API служб мультимедиа](media-services-rest-how-to-use.md).

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>Запуск конечных точек потоковой передачи с помощью портала Azure

При работе со службами мультимедиа Azure один из самых популярных сценариев — потоковая передача видео с переменной скоростью. Службы мультимедиа обеспечивают динамическую упаковку, которая позволяет своевременно доставлять закодированное содержимое MP4-файлов с переменной скоростью в форматах потоковой передачи, которые поддерживаются службами мультимедиа (MPEG DASH, HLS, Smooth Streaming), без необходимости хранения предварительно упакованных версий каждого из этих форматов потоковой передачи.

>[!NOTE]
>При создании учетной записи AMS в нее добавляется конечная точка потоковой передачи **по умолчанию** в состоянии **Остановлена**. Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи, из которой необходимо выполнять потоковую передачу содержимого, должна находиться в состоянии **Выполняется**.

Чтобы запустить конечную точку потоковой передачи, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В окне "Параметры" щелкните элемент "Потоковые конечные точки".
3. Щелкните конечную точку потоковой передачи по умолчанию.

    Появится окно сведений о конечной точке потоковой передачи по умолчанию.

4. Щелкните значок "Пуск".
5. Нажмите кнопку "Сохранить", чтобы сохранить изменения.

## <a name="connect-to-the-media-services-account-with-rest-api"></a><a id="connect"></a>Подключение к учетной записи служб мультимедиа с помощью REST API

Сведения о подключении к API AMS см. в разделе [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-a-new-asset-and-upload-a-video-file-with-rest-api"></a><a id="upload"></a>Создание нового актива и отправка видеофайла с помощью REST API

В службах мультимедиа цифровые файлы отправляются в актив. Сущность **Asset** может содержать видео, аудио, изображения, коллекции эскизов, текстовые дорожки и файлы скрытых субтитров (а также метаданные этих файлов).  После отправки этих файлов в ресурс содержимое сохраняется в безопасном расположении в облаке для дальнейшей обработки и потоковой передачи.

При создании ресурса нужно, в частности, указать параметры его создания. Значение свойства **Options** — это значение перечисления, описывающее параметры шифрования, которые могут использоваться при создании сущности Asset. Допустимые значения перечислены ниже (допустимы только сами значения, а не их сочетания).

* **None** = **0**. Шифрование не используется. При использовании этого параметра содержимое не защищено при передаче и хранении.
    Используйте этот параметр, если MP4-файл планируется доставить с помощью поэтапного скачивания.
* **StorageEncrypted** = **1**. Незашифрованное содержимое шифруется локально с помощью 256-разрядного алгоритма шифрования AES-256, а затем отправляется в службу хранилища Azure, где оно хранится в зашифрованном виде. Активы, защищенные с помощью шифрования хранилища, автоматически расшифровываются и помещаются в систему зашифрованных файлов до кодирования и при необходимости повторно кодируются до отправки в виде нового выходного актива. Шифрование хранилища чаще всего используется, если нужно защитить входные файлы мультимедиа высокого качества с помощью стойкого шифрования при хранении на диске.
* **CommonEncryptionProtected** = **2**. Используйте этот параметр при отправке содержимого, которое уже зашифровано и защищено с помощью стандартного шифрования или PlayReady DRM (например, Smooth Streaming с защитой PlayReady DRM).
* **EnvelopeEncryptionProtected** = **4**. Используйте этот параметр при отправке HLS с шифрованием AES. Файлы должны быть закодированы и зашифрованы с помощью Transform Manager.

### <a name="create-an-asset"></a>Создание ресурса
Ресурс — это контейнер, состоящий из нескольких наборов объектов или объектов разного типа в службах мультимедиа, в том числе видео, аудио, изображения, коллекции отпечатков, текстовые каналы и файлы скрытых субтитров. Для создания ресурса в REST API нужно отправить запрос POST службам мультимедиа и разместить любую информацию о свойствах ресурса в тексте запроса.

В следующем примере показано, как создать ресурс.

**HTTP-запрос**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Assets HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 45

{"Name":"BigBuckBunny.mp4", "Options":"0"}
```

**HTTP-ответ**

При успешном выполнении возвращается следующий результат:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 452
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
Server: Microsoft-IIS/8.5
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
request-id: e98be122-ae09-473a-8072-0ccd234a0657
x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:06:40 GMT

{  
    odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "State":0,
   "Created":"2015-01-18T22:06:40.6010903Z",
   "LastModified":"2015-01-18T22:06:40.6010903Z",
   "AlternateId":null,
   "Name":"BigBuckBunny2.mp4",
   "Options":0,
   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StorageAccountName":"storagetestaccount001"
}
```

### <a name="create-an-assetfile"></a>Создание сущности AssetFile
Сущность [AssetFile](/rest/api/media/operations/assetfile) представляет собой аудио- или видеофайл, который хранится в контейнере больших двоичных объектов. Файл ресурса всегда связан с ресурсом, который, в свою очередь, может содержать одну или несколько сущностей AssetFile. Задача кодировщика служб мультимедиа завершится с ошибкой, если объект файла ресурса не связан с цифровым файлом в контейнере больших двоичных объектов.

После отправки цифрового файла мультимедиа в контейнер больших двоичных объектов используйте HTTP-запрос **MERGE** , чтобы обновить сущность AssetFile информацией о файле мультимедиа (как показано далее в этом разделе).

**HTTP-запрос**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Files HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 164

{  
   "IsEncrypted":"false",
   "IsPrimary":"false",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**HTTP-ответ**

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 535
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
Server: Microsoft-IIS/8.5
request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 00:34:07 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "Name":"BigBuckBunny.mp4",
   "ContentFileSize":"0",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "EncryptionVersion":null,
   "EncryptionScheme":null,
   "IsEncrypted":false,
   "EncryptionKeyId":null,
   "InitializationVector":null,
   "IsPrimary":false,
   "LastModified":"2015-01-19T00:34:08.1934137Z",
   "Created":"2015-01-19T00:34:08.1934137Z",
   "MimeType":"video/mp4",
   "ContentChecksum":null
}
```

### <a name="creating-the-accesspolicy-with-write-permission"></a>Создание сущности AccessPolicy с разрешением на запись
Перед отправкой файлов в хранилище больших двоичных объектов настройте для политики доступа права на запись в ресурс. Для этого отправьте запрос HTTP POST в набор сущностей AccessPolicy. При создании сущности необходимо задать значение DurationInMinutes. В противном случае вы получите сообщение 500 (внутренняя ошибка сервера). Дополнительную информацию о сущностях AccessPolicy см. в разделе [AccessPolicy](/rest/api/media/operations/accesspolicy).

В следующем примере показано, как создать AccessPolicy:

**HTTP-запрос**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 74

{"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"}
```

**HTTP-ответ**

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 312
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
Server: Microsoft-IIS/8.5
request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:18:06 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AccessPolicies/@Element",
   "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "Created":"2015-01-18T22:18:06.6370575Z",
   "LastModified":"2015-01-18T22:18:06.6370575Z",
   "Name":"NewUploadPolicy",
   "DurationInMinutes":440.0,
   "Permissions":2
}
```

### <a name="get-the-upload-url"></a>Получение URL-адреса отправки

Чтобы получить фактический URL-адрес отправки, создайте указатель SAS. Указатели определяют время начала и тип конечной точки подключения для клиентов, которым требуется доступ к файлам в ресурсе. Для обработки разных запросов клиентов и удовлетворения их потребностей можно создать несколько сущностей Locator для заданной пары AccessPolicy и Asset. Каждый из этих указателей использует значения StartTime и DurationInMinutes сущности AccessPolicy, чтобы определить время, в течение которого можно использовать URL-адрес. Дополнительную информацию см. в разделе [Указатель](/rest/api/media/operations/locator).

Ниже приведен формат URL-адреса SAS:

`{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}`

Важные особенности

* С определенным ресурсом нельзя связать более пяти уникальных указателей одновременно. 
* Если вам необходимо незамедлительно передать файлы, следует задать для StartTime значение, на пять минут предшествующее текущему моменту времени. Это необходимо из-за возможного расхождения в показаниях часов на клиентском компьютере и в службах мультимедиа. Кроме того, значение StartTime должно быть указано в следующем формате DateTime: ГГГГ-ММ-ДДTЧЧ:мм:ссZ (например, "2014-05-23T17:53:50Z").    
* Задержка между моментом создания сущности Locator и моментом, когда ее можно начинать использовать, может составлять 30-40 секунд. Это касается [URL-адресов SAS](../../storage/common/storage-sas-overview.md) и исходных указателей.

В следующем примере показано, как создать указатель URL-адреса SAS в соответствии со значением свойства Type в тексте запроса (1 для указателя SAS и 2 для исходного указателя по требованию). Возвращенное свойство **Path** содержит URL-адрес, который необходимо использовать для отправки файла.

**HTTP-запрос**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Locators HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 178

{  
   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StartTime":"2015-02-18T16:45:53",
   "Type":1
}
```

**HTTP-ответ**

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 949
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
Server: Microsoft-IIS/8.5
request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 03:01:29 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Locators/@Element",
   "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
   "ExpirationDateTime":"2015-02-19T00:05:53",
   "Type":1,
   "Path":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
   "BaseUri":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247",
   "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StartTime":"2015-02-18T16:45:53",
   "Name":null
}
```

### <a name="upload-a-file-into-a-blob-storage-container"></a>Отправка файла в контейнер хранилища больших двоичных объектов
Когда сущности AccessPolicy и Locator будут заданы, фактические файлы отправляются в контейнер хранилища BLOB-объектов Azure с помощью интерфейсов REST API службы хранилища Azure. Файлы необходимо отправить в виде блочных BLOB-объектов. Страничные BLOB-объекты не поддерживаются службами мультимедиа Azure.  

> [!NOTE]
> Нужно добавить имя файла для файла, который необходимо передать, в значение **Path** сущности Locator, полученное в предыдущем разделе. Например, `https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4?`.
>
>

Дополнительную информацию о работе с большими двоичными объектами службы хранилища Azure см. в статье [API-интерфейс REST службы BLOB-объектов](/rest/api/storageservices/blob-service-rest-api).

### <a name="update-the-assetfile"></a>Обновление AssetFile
После отправки файла обновите информацию о размере сущности FileAsset (и другую информацию). Пример:

```console
MERGE https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net

{  
   "ContentFileSize":"1186540",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**HTTP-ответ**

При успешном выполнении возвращается следующий результат:

```console
HTTP/1.1 204 No Content
...
```

## <a name="delete-the-locator-and-accesspolicy"></a>Удаление Locator и AccessPolicy
**HTTP-запрос**

```console
DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**HTTP-ответ**

При успешном выполнении возвращается следующий результат:

```console
HTTP/1.1 204 No Content
...
```

**HTTP-запрос**

```console
DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**HTTP-ответ**

При успешном выполнении возвращается следующий результат:

```console
HTTP/1.1 204 No Content
...
```

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a><a id="encode"></a>Кодирование исходного файла в набор MP4-файлов с адаптивным битрейтом

После приема активов в службы мультимедиа файлы мультимедиа можно кодировать, менять их формат, добавлять водяные знаки и т. д. до их доставки клиентам. Эти действия запланированы и выполняются в нескольких экземплярах фоновых ролей для обеспечения высокой производительности и доступности. Эти действия называются заданиями, и каждое задание состоит из атомарных задач, которые выполняют фактическую работу с файлом ресурса-контейнера (дополнительные сведения см. в описаниях [заданий](/rest/api/media/operations/job) и [задач](/rest/api/media/operations/task)).

Как было упомянуто ранее, при работе со службами мультимедиа Azure один из наиболее распространенных сценариев — доставка потоковой передачи с адаптивным битрейтом клиентам. Службы мультимедиа могут динамически упаковывать набор MP4-файлов с адаптивной скоростью в один из следующих форматов. HTTP Live Streaming (HLS), Smooth Streaming, DASH MPEG.

Ниже показано, как создать задание, содержащее одну задачу кодирования. Эта задача задает перекодирование мезонинного файла в набор MP4-файлов с адаптивной скоростью с помощью **Media Encoder Standard**. В данном разделе также показано, как отслеживать ход выполнения задания. После завершения задания можно будет создавать указатели, которые необходимы для получения доступа к ресурсам-контейнерам.

### <a name="get-a-media-processor"></a>Получение обработчика мультимедиа
В службах мультимедиа обработчик мультимедиа — это компонент, который работает со специфическими задачами обработки, такими как кодирование, преобразование формата, шифрование или расшифровка мультимедийного содержимого. Для задачи кодирования, продемонстрированной в данном руководстве, будет использоваться Media Encoder Standard.

Следующий код запрашивает идентификатор кодировщика.

**HTTP-запрос**

```console
    GET https://wamsbayclus001rest-hs.cloudapp.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.19
    Host: wamsbayclus001rest-hs.cloudapp.net
```

**HTTP-ответ**

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 273
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
x-ms-request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 07:54:09 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#MediaProcessors",
   "value":[  
      {  
         "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "Description":"Media Encoder Standard",
         "Name":"Media Encoder Standard",
         "Sku":"",
         "Vendor":"Microsoft",
         "Version":"1.1"
      }
   ]
}
```

### <a name="create-a-job"></a>Создание задания
Каждое задание может состоять из одной или нескольких задач в зависимости от типа обработки, которую необходимо выполнить. REST API позволяет создавать задания и связанные с ними задачи одним из двух способов: Задачи можно определить как встроенные с помощью свойств навигации задач в сущностях Job или с помощью пакетной обработки OData. Пакет SDK для служб мультимедиа использует пакетную обработку. Тем не менее для удобства чтения в этой статье задачи определены прямо в тексте примеров кода. Чтобы узнать больше о пакетной обработке, ознакомьтесь с [пакетной обработкой посредством протокола OData](https://www.odata.org/documentation/odata-version-3-0/batch-processing/).

В следующем примере показано, как создать и опубликовать задание с одной задачей, предназначенной для кодирования видео с определенным разрешением и качеством. Следующий раздел документации содержит список всех [предустановок задач](./media-services-mes-presets-overview.md) , поддерживаемых обработчиком Media Encoder Standard.  

**HTTP-запрос**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 482

{  
   "Name":"NewTestJob",
   "InputMediaAssets":[  
      {  
         "__metadata":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')"
         }
      }
   ],
   "Tasks":[  
      {  
         "Configuration":"Adaptive Streaming",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset>
            <outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
      }
   ]
}
```

**HTTP-ответ**

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 1215
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')
Server: Microsoft-IIS/8.5
request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
x-ms-request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 08:04:35 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
         "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Job"
      },
      "Tasks":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Tasks"
         }
      },
      "OutputMediaAssets":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets"
         }
      },
      "InputMediaAssets":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')/InputMediaAssets"
         }
      },
      "Id":"nb:jid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
      "Name":"NewTestJob",
      "Created":"2015-01-19T08:04:34.3287057Z",
      "LastModified":"2015-01-19T08:04:34.3287057Z",
      "EndTime":null,
      "Priority":0,
      "RunningDuration":0,
      "StartTime":null,
      "State":0,
      "TemplateId":null,
      "JobNotificationSubscriptions":{  
         "__metadata":{  
            "type":"Collection(Microsoft.Cloud.Media.Vod.Rest.Data.Models.JobNotificationSubscription)"
         },
         "results":[  

         ]
      }
   }
}
```

При создании любого запроса задания необходимо учитывать несколько важных моментов.

* Чтобы определить количество входных или выходных ресурсов, используемых задачей, в свойствах TaskBody НЕОБХОДИМО использовать XML-литерал. В статье о задаче приведено определение схемы XML.
* В определении TaskBody каждое внутреннее значение `<inputAsset>` и `<outputAsset>` необходимо установить как JobInputAsset(value) или JobOutputAsset(value).
* В задаче может содержаться несколько выходных ресурсов. Значение JobOutputAsset(x) может использоваться только один раз в качестве выходных данных задачи в задании.
* В качестве входного ресурса задачи можно указать JobInputAsset или JobOutputAsset.
* Задачи не должны образовывать цикл.
* Значение параметра, которое передается в JobInputAsset или JobOutputAsset, — это значение индекса для ресурса. Фактические ресурсы определяются в свойствах навигации ресурсов InputMediaAsset и OutputMediaAsset в определении сущности Job.

> [!NOTE]
> Так как службы мультимедиа созданы на платформе OData версии 3, ссылки на отдельные ресурсы в коллекциях свойств навигации ресурсов InputMediaAsset и OutputMediaAsset добавляются в виде пары "имя-значение" __metadata: uri.
>
>

* Ресурсы InputMediaAsset сопоставляются с одним или несколькими ресурсами, созданными в службах мультимедиа. Ресурсы OutputMediaAsset создаются системой. Они не ссылаются на существующий ресурс.
* Ресурсам OutputMediaAsset можно присвоить имя с помощью атрибута assetName. Если этот атрибут отсутствует, то именем ресурса OutputMediaAsset будет внутреннее текстовое значение элемента `<outputAsset>` с суффиксом, которым может быть значение параметра имени задания или параметра идентификатора задания (если свойство Name не определено). Например, если задать для атрибута assetName значение Sample, для свойства Name ресурса OutputMediaAsset будет задано значение Sample. Тем не менее, если значение атрибута assetName не задано, но в качестве имени задания задано NewJob, имя OutputMediaAsset будет выглядеть следующим образом: JobOutputAsset(значение)_NewJob.

    В следующем примере показано, как установить атрибут assetName:

    ```console
    "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"
    ```
* Чтобы включить создание цепных задач:

  * в задании должно быть по крайней мере две задачи;
  * должна существовать хотя бы одна задача, входные данные которой являются выходными данными другой задачи в задании.

Дополнительную информацию см. в статье [Создание задания кодирования с помощью REST API служб мультимедиа](media-services-rest-encode-asset.md).

### <a name="monitor-processing-progress"></a>Отслеживание хода обработки
Состояние задания можно получить с помощью свойства State, как показано в следующем примере:

**HTTP-запрос**

```console
GET https://wamsbayclus001rest-hs.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/State HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 0
```

**HTTP-ответ**

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 17
Content-Type: application/json;odata=verbose;charset=utf-8
Server: Microsoft-IIS/7.5
x-ms-request-id: 01324d81-18e2-4493-a3e5-c6186209f0c8
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Sun, 13 May 2012 05:16:53 GMT

{"d":{"State":2}}
```

### <a name="cancel-a-job"></a>Отмена задания
Службы мультимедиа позволяют отменить выполнение заданий с помощью функции CancelJob. Этот вызов вернет код ошибки 400 при попытке отменить задание, если оно находится в состоянии "Отменено", "Отменяется", "Ошибка" или "Завершено".

В следующем примере показано, как вызвать CancelJob.

**HTTP-запрос**

```console
GET https://wamsbayclus001rest-hs.net/API/CancelJob?jobid='nb%3ajid%3aUUID%3a71d2dd33-efdf-ec43-8ea1-136a110bd42c' HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
```

В случае успешного выполнения возвращается код ответа 204 без текста сообщения.

> [!NOTE]
> При передаче идентификатора задания в качестве параметра CancelJob его необходимо закодировать идентификатор задания в URL-адресе (обычно в виде nb:jid:UUID: somevalue).
>
>

### <a name="get-the-output-asset"></a>Получение выходного ресурса
В следующем коде показано, как запросить идентификатор выходного ресурса.

**HTTP-запрос**

```console
GET https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets() HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**HTTP-ответ**

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 445
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
x-ms-request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 08:28:13 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets",
   "value":[  
      {  
         "Id":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
         "State":0,
         "Created":"2015-01-19T07:52:15.603",
         "LastModified":"2015-01-19T07:52:15.603",
         "AlternateId":null,
         "Name":"Multibitrate MP4s",
         "Options":0,
         "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c",
         "StorageAccountName":"storagetestaccount001"
      }
   ]
}
```

## <a name="publish-the-asset-and-get-streaming-and-progressive-download-urls-with-rest-api"></a><a id="publish_get_urls"></a>Публикация актива и получение URL-адресов потоковой передачи и поэтапного скачивания с помощью REST API

Чтобы выполнить потоковую передачу ресурса-контейнера или скачать его, сначала необходимо "опубликовать" его, создав указатель. Указатели предоставляют доступ к файлам, содержащимся в активе. Службы мультимедиа поддерживают два типа указателей: Указатели OnDemandOrigin, использующиеся для потоковой передачи мультимедиа (например, MPEG DASH, HLS или Smooth Streaming), и указатели подписанного URL-адреса, использующиеся для скачивания файлов мультимедиа. 

После создания указателей можно создать URL-адреса, которые используются для потоковой передачи или скачивания файлов.

>[!NOTE]
>При создании учетной записи AMS в нее добавляется конечная точка потоковой передачи **по умолчанию** в состоянии **Остановлена**. Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи, из которой необходимо выполнять потоковую передачу содержимого, должна находиться в состоянии **Выполняется**.

URL-адрес потоковой передачи для Smooth Streaming имеет следующий формат:

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest
```

URL-адрес потоковой передачи для HLS имеет следующий формат:

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)
```

URL-адрес потоковой передачи для MPEG DASH имеет следующий формат:

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)
```

URL-адрес SAS, используемый для скачивания файлов, имеет следующий формат:

```console
{blob container name}/{asset name}/{file name}/{SAS signature}
```

В этом разделе показано, как выполнять следующие задачи, необходимые для "публикации" ресурсов.  

* Создание сущности AccessPolicy с разрешением на чтение
* Создание URL-адреса SAS для скачивания содержимого
* Создание исходного URL-адреса для потоковой передачи содержимого

### <a name="creating-the-accesspolicy-with-read-permission"></a>Создание сущности AccessPolicy с разрешением на чтение
Перед скачиванием или потоковой передачей любого медиасодержимого сначала определите сущность AccessPolicy с разрешениями на чтение и создайте соответствующую сущность указателя, которая задает тип механизма доставки, который нужно включить для клиентов. Дополнительную информацию о доступных свойствах см. в статье [Свойства сущности AccessPolicy](/rest/api/media/operations/accesspolicy#accesspolicy_properties).

В следующем примере показано, как задать сущность AccessPolicy для разрешений на чтение для данного ресурса.

```console
POST https://wamsbayclus001rest-hs.net/API/AccessPolicies HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 74
Expect: 100-continue

{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}
```

При успешном выполнении возвращается код успешного завершения 201, описывающий созданную сущность AccessPolicy. Затем идентификатор сущности AccessPolicy будет использоваться вместе с идентификатором сущности Asset ресурса, содержащего файл, который необходимо доставить (например, выходной ресурс), для создания сущности Locator.

> [!NOTE]
> Этот основной рабочий процесс будет таким же, как при передаче файла во время приема ресурса (как было описано ранее в этой статье). Кроме того, если вам (или вашим клиентам) необходимо незамедлительно получить доступ к файлам, как и при передаче файлов, установите для StartTime значение, на пять минут предшествующее текущему моменту времени. Это действие необходимо из-за возможного расхождения в показаниях часов на клиенте и в службах мультимедиа. Значение StartTime должно быть указано в следующем формате даты и времени: ГГГГ-ММ-ДДTЧЧ:мм:ссZ (например, "2014-05-23T17:53:50Z").
>
>

### <a name="creating-a-sas-url-for-downloading-content"></a>Создание URL-адреса SAS для скачивания содержимого
Следующий код показывает, как получить URL-адрес, который может использоваться для скачивания файла мультимедиа, созданного и переданного ранее. У сущности AccessPolicy есть разрешения на чтение, а путь указателя указывает на URL-адрес скачивания SAS.

```console
POST https://wamsbayclus001rest-hs.net/API/Locators HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 182
Expect: 100-continue

{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c", "StartTime" : "2014-05-17T16:45:53", "Type":1}
```

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 1150
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')
Server: Microsoft-IIS/7.5
x-ms-request-id: 8cfd8556-3064-416a-97f2-67d9e35f0911
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Mon, 14 May 2012 21:41:32 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
      },
      "AccessPolicy":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')/AccessPolicy"
         }
      },
      "Asset":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Asset"
         }
      },
      "Id":"nb:lid:UUID:8e5a821d-2194-4d00-8884-adf979856874",
      "ExpirationDateTime":"\/Date(1337049393000)\/",
      "Type":1,
      "Path":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c?st=2012-05-14T21%3A36%3A33Z&se=2012-05-15T02%3A36%3A33Z&sr=c&si=8e5a821d-2194-4d00-8884-adf979856874&sig=y75dViDpC5V8WutrXM%2B%2FGpR3uOtqmlISiNlHU1YUBOg%3D",
      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
      "AssetId":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
      "StartTime":"\/Date(1337031393000)\/"
   }
}
```

Возвращенное свойство **Path** содержит URL-адрес SAS.

> [!NOTE]
> При скачивании зашифрованного содержимого хранилища необходимо вручную расшифровать его перед его отображением или использовать обработчик мультимедиа для расшифровки хранилища в задаче обработки, чтобы вывести обработанные файлы в незашифрованном виде в OutputAsset, а затем скачать их из этого ресурса. Дополнительную информацию об обработке см. в разделе "Создание задания кодирования с помощью REST API служб мультимедиа". Кроме того, указатели URL-адреса SAS нельзя обновить после их создания. Например, нельзя повторно использовать один и тот же указатель с обновленным значением StartTime. Это связано со способом создания URL-адресов SAS. Если вы хотите получить доступ к ресурсу для скачивания после истечения срока действия указателя, необходимо создать новый указатель с новым значением StartTime.
>
>

### <a name="download-files"></a>Загрузка файлов
Когда сущности AccessPolicy и Locator будут заданы, файлы можно будет скачать с помощью REST API службы хранилища Azure.  

> [!NOTE]
> Необходимо добавить имя файла для файла, который нужно скачать, в значение **Path** сущности Locator, полученное в предыдущем разделе. Например, `https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4`? . . .

Дополнительную информацию о работе с большими двоичными объектами службы хранилища Azure см. в статье [API-интерфейс REST службы BLOB-объектов](/rest/api/storageservices/blob-service-rest-api).

В результате задания кодирования, выполненного ранее (кодирования в набор MP4-файлов с адаптивной скоростью), у вас будет несколько MP4-файлов для поэтапного скачивания. Пример:    

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

### <a name="creating-a-streaming-url-for-streaming-content"></a>Создание URL-адреса потоковой передачи потокового содержимого
В следующем коде показано, как создать указатель URL-адреса потоковой передачи:

```console
POST https://wamsbayclus001rest-hs/API/Locators HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs
Content-Length: 182
Expect: 100-continue

{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3", "StartTime" : "2014-05-17T16:45:53",, "Type":2}
```

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 981
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')
Server: Microsoft-IIS/7.5
x-ms-request-id: 2eac4158-fc78-4fa2-81ee-c9f582dc385f
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Mon, 14 May 2012 21:41:39 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
      },
      "AccessPolicy":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/AccessPolicy"
         }
      },
      "Asset":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/Asset"
         }
      },
      "Id":"nb:lid:UUID:52034bf6-dfae-4d83-aad3-3bd87dcb1a5d",
      "ExpirationDateTime":"\/Date(1337049395000)\/",
      "Type":2,
      "Path":"http://wamsbayclus001rest-hs.net/52034bf6-dfae-4d83-aad3-3bd87dcb1a5d/",
      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
      "AssetId":"nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3",
      "StartTime":"\/Date(1337031395000)\/"
   }
}
```

Для потоковой передачи с исходного URL-адреса Smooth Streaming в проигрывателе потокового мультимедиа необходимо добавить путь свойства с именем файла манифеста Smooth Streaming, за которым следует "/manifest".

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest`

Для потоковой передачи HLS добавьте (format=m3u8-aapl) после "/manifest".

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)`

Для потоковой передачи MPEG DASH добавьте (format=mpd-time-csf) после "/manifest".

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)`


## <a name="play-your-content"></a><a id="play"></a>Воспроизведение содержимого
Чтобы воспроизвести видео, используйте [Проигрыватель служб мультимедиа Azure](https://aka.ms/azuremediaplayer).

Для тестирования поэтапного скачивания вставьте URL-адрес в браузер (например, Internet Explorer, Chrome, Safari).

## <a name="next-steps-media-services-learning-paths"></a>Дальнейшие действия Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]