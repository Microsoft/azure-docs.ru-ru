---
title: Создание ключей содержимого с помощью REST | Документация Майкрософт
description: В этой статье показано, как создавать ключи содержимого, обеспечивающие безопасный доступ к ресурсам.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 95e9322b-168e-4a9d-8d5d-d7c946103745
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: a9960ece05b7cab4ddd26fdac7516f4adbccf5b0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103017262"
---
# <a name="create-content-keys-with-rest"></a>Создание ключей содержимого с помощью REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [REST](media-services-rest-create-contentkey.md)
> * [.NET](media-services-dotnet-create-contentkey.md)
> 
> 

Службы мультимедиа позволяют доставлять зашифрованные ресурсы. **ContentKey** обеспечивает безопасный доступ к вашим **ресурсам-контейнерам**. 

При создании нового ресурса-контейнера (например, перед [передачей файлов](media-services-rest-upload-files.md)) можно указать следующие параметры шифрования: **StorageEncrypted**, **CommonEncryptionProtected** или **EnvelopeEncryptionProtected**. 

При доставке ресурсов-контейнеров в клиенты можно [настроить динамическое шифрование таких ресурсов](media-services-rest-configure-asset-delivery-policy.md), используя один из двух следующих типов шифрования: **DynamicEnvelopeEncryption** или **DynamicCommonEncryption**.

Зашифрованные ресурсы-контейнеры должны быть связаны с сущностями **ContentKey**. В этой статье описано, как создать ключ содержимого.

Ниже приведены общие инструкции по созданию ключей содержимого, которые нужно связать с ресурсами, подлежащими шифрованию. 

1. Создайте случайный 16-разрядный ключ AES (для общего и конвертного шифрования) или 32-разрядного ключа AES (для шифрования в хранилище). 
   
    Это ключ содержимого ресурса, то есть всех файлов, связанных с этим ресурсом. При расшифровке используется один и тот же ключ содержимого. 
2. Вызовите методы [GetProtectionKeyId](/rest/api/media/operations/rest-api-functions#getprotectionkeyid) и [GetProtectionKey](/rest/api/media/operations/rest-api-functions#getprotectionkey), чтобы получить правильный сертификат X.509, который должен использоваться для шифрования ключа содержимого.
3. Зашифруйте ключ содержимого с помощью открытого ключа сертификата X.509. 
   
   Пакет SDK служб мультимедиа для .NET использует при выполнении шифрования RSA с OAEP.  Примером может служить [функция EncryptSymmetricKeyData](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs).
4. Создайте значение контрольной суммы (на основе алгоритма вычисления контрольной суммы ключа AES для PlayReady), которая вычисляется по идентификатору ключа и ключу содержимого. Дополнительную информацию см. в разделе "Алгоритм вычисления контрольной суммы ключа AES для PlayReady" [здесь](https://www.microsoft.com/playready/documents/).
   
    В следующем примере .NET контрольная сумма вычисляется с помощью части "GUID" идентификатора ключа и незащищенного ключа содержимого.

    ```console
    public static string CalculateChecksum(byte[] contentKey, Guid keyId)
     {
       byte[] array = null;
         using (AesCryptoServiceProvider aesCryptoServiceProvider = new AesCryptoServiceProvider())
         {
             aesCryptoServiceProvider.Mode = CipherMode.ECB;
             aesCryptoServiceProvider.Key = contentKey;
             aesCryptoServiceProvider.Padding = PaddingMode.None;
             ICryptoTransform cryptoTransform = aesCryptoServiceProvider.CreateEncryptor();
             array = new byte[16];
             cryptoTransform.TransformBlock(keyId.ToByteArray(), 0, 16, array, 0);
         }
         byte[] array2 = new byte[8];
         Array.Copy(array, array2, 8);
         return Convert.ToBase64String(array2);
     }
    ```

5. Создайте ключ содержимого, используя значения **EncryptedContentKey** (преобразуется в строку с кодировкой base64), **ProtectionKeyId**, **ProtectionKeyType**, **ContentKeyType** и **Checksum**, полученные на предыдущих шагах.
6. Свяжите сущность **ContentKey** с сущностью **Asset** с помощью операции $links.

Обратите внимание, что в этой статье не объясняется, как сгенерировать ключ AES, зашифровать его и вычислить контрольную сумму. 

> [!NOTE]
> 
> При доступе к сущностям в службах мультимедиа необходимо задать определенные поля и значения заголовков в HTTP-запросах. Дополнительную информацию см. в статье [Обзор интерфейса REST API служб мультимедиа](media-services-rest-how-to-use.md).

## <a name="connect-to-media-services"></a>Подключение к службам мультимедиа

Сведения о подключении к API AMS см. в разделе [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="retrieve-the-protectionkeyid"></a>Получение ProtectionKeyId
В следующем примере показано, как получить ProtectionKeyId (отпечаток сертификата) для сертификата, который необходимо использовать при шифровании ключа содержимого. Выполните этот шаг, чтобы проверить наличие соответствующего сертификата на компьютере.

Запрос:

```console
GET https://media.windows.net/api/GetProtectionKeyId?contentKeyType=0 HTTP/1.1
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
Host: media.windows.net
```

Ответ:

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 139
charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
x-ms-request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 04 Feb 2015 02:42:52 GMT

$metadata#Edm.String","value":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C"}
```

## <a name="retrieve-the-protectionkey-for-the-protectionkeyid"></a>Получение ProtectionKey для ProtectionKeyId
В следующем примере показано, как получить сертификат X.509, используя значение ProtectionKeyId, полученное на предыдущем шаге.

Запрос:

```console
GET https://media.windows.net/api/GetProtectionKey?ProtectionKeyId='7D9BB04D9D0A4A24800CADBFEF232689E048F69C' HTTP/1.1
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
Host: media.windows.net
```


Ответ:

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 1227
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
x-ms-request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Thu, 05 Feb 2015 07:52:30 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String", "value":"MIIDSTCCAjGgAwIBAgIQqf92wku/HLJGCbMAU8GEnDANBgkqhkiG9w0BAQQFADAuMSwwKgYDVQQDEyN3YW1zYmx1cmVnMDAxZW5jcnlwdGFsbHNlY3JldHMtY2VydDAeFw0xMjA1MjkwNzAwMDBaFw0zMjA1MjkwNzAwMDBaMC4xLDAqBgNVBAMTI3dhbXNibHVyZWcwMDFlbmNyeXB0YWxsc2VjcmV0cy1jZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzR0SEbXefvUjb9wCUfkEiKtGQ5Gc328qFPrhMjSo+YHe0AVviZ9YaxPPb0m1AaaRV4dqWpST2+JtDhLOmGpWmmA60tbATJDdmRzKi2eYAyhhE76MgJgL3myCQLP42jDusWXWSMabui3/tMDQs+zfi1sJ4Ch/lm5EvksYsu6o8sCv29VRwxfDLJPBy2NlbV4GbWz5Qxp2tAmHoROnfaRhwp6WIbquk69tEtu2U50CpPN2goLAqx2PpXAqA+prxCZYGTHqfmFJEKtZHhizVBTFPGS3ncfnQC9QIEwFbPw6E5PO5yNaB68radWsp5uvDg33G1i8IT39GstMW6zaaG7cNQIDAQABo2MwYTBfBgNVHQEEWDBWgBCOGT2hPhsvQioZimw8M+jOoTAwLjEsMCoGA1UEAxMjd2Ftc2JsdXJlZzAwMWVuY3J5cHRhbGxzZWNyZXRzLWNlcnSCEKn/dsJLvxyyRgmzAFPBhJwwDQYJKoZIhvcNAQEEBQADggEBABcrQPma2ekNS3Wc5wGXL/aHyQaQRwFGymnUJ+VR8jVUZaC/U/f6lR98eTlwycjVwRL7D15BfClGEHw66QdHejaViJCjbEIJJ3p2c9fzBKhjLhzB3VVNiLIaH6RSI1bMPd2eddSCqhDIn3VBN605GcYXMzhYp+YA6g9+YMNeS1b+LxX3fqixMQIxSHOLFZ1G/H2xfNawv0VikH3djNui3EKT1w/8aRkUv/AAV0b3rYkP/jA1I0CPn0XFk7STYoiJ3gJoKq9EMXhit+Iwfz0sMkfhWG12/XO+TAWqsK1ZxEjuC9OzrY7pFnNxs4Mu4S8iinehduSpY+9mDd3dHynNwT4="}
```

## <a name="create-the-contentkey"></a>Создание ContentKey
После получения сертификата X.509 и использования его открытого ключа для шифрования ключа содержимого создайте сущность **ContentKey** и задайте для нее соответствующие свойства.

Одно из значений, которые необходимо задать при создания ключа содержимого — это тип. Выберите одно из следующих значений:

```console
public enum ContentKeyType
{
    /// <summary>
    /// Specifies a content key for common encryption.
    /// </summary>
    /// <remarks>This is the default value.</remarks>
    CommonEncryption = 0,

    /// <summary>
    /// Specifies a content key for storage encryption.
    /// </summary>
    StorageEncryption = 1,

    /// <summary>
    /// Specifies a content key for configuration encryption.
    /// </summary>
    ConfigurationEncryption = 2,

    /// <summary>
    /// Specifies a content key for Envelope encryption.  Only used internally.
    /// </summary>
    EnvelopeEncryption = 4
    }
```

В следующем примере показано, как создать **ContentKey** с набором **ContentKeyType** для шифрования хранилища ("1"), а **параметра protectionkeytype —** — значение "0", чтобы указать, что идентификатором ключа защиты является отпечаток сертификата X. 509.  

Запрос

```console
POST https://media.windows.net/api/ContentKeys HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
Host: media.windows.net
{
"Name":"ContentKey",
"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C", 
"ContentKeyType":"1", 
"ProtectionKeyType":"0",
"EncryptedContentKey":"your encrypted content key",
"Checksum":"calculated checksum"
}
```

Ответ:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 777
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://media.windows.net/api/ContentKeys('nb%3Akid%3AUUID%3A9c8ea9c6-52bd-4232-8a43-8e43d8564a99')
Server: Microsoft-IIS/8.5
request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
x-ms-request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 04 Feb 2015 02:37:46 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeys/@Element",
"Id":"nb:kid:UUID:9c8ea9c6-52bd-4232-8a43-8e43d8564a99","Created":"2015-02-04T02:37:46.9684379Z",
"LastModified":"2015-02-04T02:37:46.9684379Z",
"ContentKeyType":1,
"EncryptedContentKey":"your encrypted content key",
"Name":"ContentKey",
"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C",
"ProtectionKeyType":0,
"Checksum":"calculated checksum"}
```

## <a name="associate-the-contentkey-with-an-asset"></a>Связывание сущности ContentKey с сущностью Asset
После создания сущности ContentKey свяжите ее с сущностью Asset, используя операцию $links, как показано в следующем примере:

Запрос:

```console
POST https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Afbd7ce05-1087-401b-aaae-29f16383c801')/$links/ContentKeys HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Content-Type: application/json
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
Host: media.windows.net

{"uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeys('nb%3Akid%3AUUID%3A01e6ea36-2285-4562-91f1-82c45736047c')"}
```

Ответ:

```console
HTTP/1.1 204 No Content 
```

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
