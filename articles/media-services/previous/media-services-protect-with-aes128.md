---
title: Использование динамического шифрования AES-128 и службы доставки ключей | Документация Майкрософт
description: В этой статье показано, как динамически шифровать содержимое с помощью алгоритма AES-128 и службы доставки ключей.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 4d2c10af-9ee0-408f-899b-33fa4c1d89b9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 7dacc01b37fb91f02fe67f76fe79c0bfa9375ea7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103009544"
---
# <a name="use-aes-128-dynamic-encryption-and-the-key-delivery-service"></a>Использование динамического шифрования AES-128 и службы доставки ключей

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-protect-with-aes128.md)
> * [Java](https://github.com/rnrneverdies/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
>  

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Службы мультимедиа можно использовать для доставки содержимого HTTP Live Streaming (HLS) и Smooth Streaming, зашифрованного с помощью 128-битных ключей шифрования AES. Они также включают в себя службу доставки ключей, которая доставляет ключи шифрования авторизованным пользователям. Если нужно зашифровать ресурс-контейнер с помощью служб мультимедиа, то свяжите с ним ключ шифрования и настройте политики авторизации для ключа. Когда поток запрашивается проигрывателем, службы мультимедиа используют указанный ключ для динамического шифрования содержимого с помощью AES. Чтобы расшифровать поток, проигрыватель запросит ключ у службы доставки ключей. Чтобы определить, есть ли у пользователя право на получение ключа, служба оценивает политики авторизации, заданные для ключа.

Службы мультимедиа поддерживают несколько способов аутентификации пользователей, которые запрашивают ключи. Для политики авторизации ключа содержимого можно задать одно или несколько ограничений: открытая авторизация или авторизация с помощью маркера. При ограничении с помощью маркера к политике должен прилагаться маркер, выданный службой маркеров безопасности (STS). Службы мультимедиа поддерживают маркеры в форматах [простого веб-маркера](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2) (SWT) и [JSON Web Token](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3) (JWT). Дополнительные сведения см. [в статье Настройка политики авторизации для ключа содержимого](media-services-protect-with-aes128.md#configure_key_auth_policy).

Чтобы воспользоваться преимуществами динамического шифрования, необходимо иметь ресурс-контейнер, содержащий набор многоскоростных MP4-файлов или многоскоростных исходных файлов Smooth Streaming. Вам также потребуется настроить политику доставки для ресурса-контейнера (описывается далее в этой статье). В зависимости от формата, указанного в URL-адресе потоковой передачи, сервер потокового воспроизведения по запросу обеспечивает доставку содержимого по выбранному протоколу. Благодаря этому можно хранить и оплачивать файлы в одном формате хранения. Службы мультимедиа создают и обрабатывают соответствующий ответ с учетом запросов клиента.

Эта статья полезна разработчикам приложений, которые доставляют защищенные файлы мультимедиа. В ней показано, как настроить политики авторизации для службы доставки ключей, чтобы только авторизованные клиенты могли получать ключи шифрования, а также рассматривается использование динамического шифрования.

Дополнительные сведения о том, как шифровать содержимое с помощью Advanced Encryption Standard (AES) для доставки в Safari на macOS, см. в [этой записи блога](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).
Обзор способов защиты содержимого мультимедиа с помощью шифрования AES представлен в [этом видео](https://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-Protecting-your-Media-Content-with-AES-Encryption).


## <a name="aes-128-dynamic-encryption-and-key-delivery-service-workflow"></a>Рабочий процесс динамического шифрования AES-128 и службы доставки ключей

Выполните описанные ниже общие действия для шифрования файлов с помощью AES, используя службу доставки ключей служб мультимедиа и динамическое шифрование.

1. [Создайте ресурс и отправьте файлы в ресурс](media-services-protect-with-aes128.md#create_asset).

2. [Закодируйте ресурс с файлами в набор MP4-файлов с переменной скоростью](media-services-protect-with-aes128.md#encode_asset).

3. [Создайте ключ содержимого и свяжите его с закодированным ресурсом-контейнером](media-services-protect-with-aes128.md#create_contentkey). В службах мультимедиа ключ содержимого содержит ключ шифрования ресурса.

4. [Настройте политику авторизации ключа содержимого](media-services-protect-with-aes128.md#configure_key_auth_policy). Потребуется настроить политику авторизации для ключа содержимого. Прежде чем ключ содержимого будет доставлен в клиент, его нужно привести в соответствие с политикой.

5. [Настройте политику доставки для ресурса-контейнера](media-services-protect-with-aes128.md#configure_asset_delivery_policy). Конфигурация политики доставки включает URL-адрес для получения ключа и вектор инициализации (IV). (Для шифрования и расшифровки алгоритму AES-128 требуется один и тот же вектор инициализации.) Конфигурация также включает протокол доставки (например, MPEG-ТИРЕ, HLS, Smooth Streaming или все) и тип динамического шифрования (например, конверт или без динамического шифрования).

    К разным протоколам можно применять разные политики в отношении одного и того же ресурса. Например, вы можете применить шифрование PlayReady при использовании Smooth или DASH и конвертное шифрование AES при использовании HLS. Потоковая передача по тем протоколам, которые не определены в политике доставки блокируется. (Например, при добавлении одной политики, указывающей только HLS в качестве протокола.) Исключением является отсутствие определенной политики доставки ресурсов. Тогда все протоколы разрешено использовать в незашифрованном виде.

6. [Создайте указатель OnDemand](media-services-protect-with-aes128.md#create_locator), чтобы получить URL-адрес для потоковой передачи.

В статье также показано, [как клиентское приложение может запрашивать ключ у службы доставки ключей](media-services-protect-with-aes128.md#client_request).

В конце статьи вы найдете полный [пример для .NET](media-services-protect-with-aes128.md#example).

На следующем изображении показан описанный выше рабочий процесс. В этом случае для аутентификации используется маркер.

![Защита с помощью AES-128](./media/media-services-content-protection-overview/media-services-content-protection-with-aes.png)

В оставшейся части статьи приводятся объяснения, примеры кода и ссылки на статьи с инструкциями для выполнения описанных выше задач.

## <a name="current-limitations"></a>Текущие ограничения
При добавлении или обновлении политики доставки ресурсов необходимо удалить все существующие указатели и создать новый.

## <a name="create-an-asset-and-upload-files-into-the-asset"></a><a id="create_asset"></a>Создание ресурса-контейнера и отправка в него файлов
Для кодирования и потоковой передачи видео, а также управления ими сначала необходимо отправить содержимое в службы мультимедиа. Отправленное содержимое будет сохранено в безопасном облачном хранилище для последующей обработки и потоковой передачи. 

Дополнительные сведения см. в статье [Передача файлов в учетную запись служб мультимедиа с помощью .NET](media-services-dotnet-upload-files.md).

## <a name="encode-the-asset-that-contains-the-file-to-the-adaptive-bitrate-mp4-set"></a><a id="encode_asset"></a>Кодирование ресурса, содержащего файл, в набор MP4 с адаптивной скоростью
При использовании динамического шифрования создается ресурс, содержащий набор MP4-файлов или исходных файлов Smooth Streaming (оба типа относятся к файлам с несколькими скоростями). Затем с учетом формата, указанного в манифесте или запросе фрагмента, сервер потоковой передачи по запросу организует передачу содержимого по выбранному протоколу. Затем нужно только хранить и оплачивать файлы в одном формате хранения. Службы мультимедиа создают и обрабатывают соответствующий ответ с учетом запросов клиента. Чтобы узнать больше, ознакомьтесь с [общими сведениями о динамической упаковке](media-services-dynamic-packaging-overview.md).

>[!NOTE]
>При создании учетной записи служб мультимедиа в нее добавляется конечная точка потоковой передачи по умолчанию в состоянии "Остановлена". Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи, из которой необходимо выполнять потоковую передачу содержимого, должна находиться в состоянии "Выполняется". 
>
>Кроме того, используйте динамическую упаковку и динамическое шифрование, ресурс должен содержать набор файлов формата MP4 или потоковой передачи Smooth Streaming с переменной скоростью.

Инструкции по кодированию см. в статье [Кодирование ресурса-контейнера с помощью Media Encoder Standard и .NET](media-services-dotnet-encode-with-media-encoder-standard.md).

## <a name="create-a-content-key-and-associate-it-with-the-encoded-asset"></a><a id="create_contentkey"></a>Создание ключа содержимого и связывание его с закодированным ресурсом-контейнером
В службах мультимедиа ключ содержимого содержит ключ, который используется для шифрования ресурса-контейнера.

Дополнительные сведения см. в руководстве по [созданию ключей содержимого](media-services-dotnet-create-contentkey.md).

## <a name="configure-the-content-keys-authorization-policy"></a><a id="configure_key_auth_policy"></a>Настройка политики авторизации для ключа содержимого
Службы мультимедиа поддерживают несколько способов аутентификации пользователей, которые запрашивают ключи. Потребуется настроить политику авторизации для ключа содержимого. Прежде чем ключ будет доставлен клиенту (проигрыватель), он должен быть приведен в соответствие с политикой. В политике авторизации ключа содержимого может быть одно или несколько ограничений авторизации: можно использовать открытую авторизацию, с ограничением по маркеру или с ограничением по IP-адресу.

Дополнительную информацию см. в статье о [настройке политики авторизации для ключа содержимого](media-services-dotnet-configure-content-key-auth-policy.md).

## <a name="configure-an-asset-delivery-policy"></a><a id="configure_asset_delivery_policy"></a>Настройка политики доставки для ресурса
Настройте политику доставки для ресурса-контейнера. Вот некоторые элементы, входящие в конфигурацию политики доставки:

* URL-адрес для получения ключа. 
* Вектор инициализации, используемый для конвертного шифрования. Для шифрования и расшифровки с помощью AES-128 требуется один и тот же вектор инициализации. 
* Протокол доставки ресурсов-контейнеров (например, MPEG-DASH, HLS, Smooth Streaming или все перечисленные).
* Тип динамического шифрования (например, конвертное шифрование с помощью AES), если оно используется. 

Дополнительные сведения см. в статье о [настройке политики доставки для ресурсов-контейнеров](media-services-dotnet-configure-asset-delivery-policy.md).

## <a name="create-an-ondemand-streaming-locator-to-get-a-streaming-url"></a><a id="create_locator"></a>Создание указателя потоковой передачи по запросу для получения URL-адреса для потоковой передачи
При использовании протоколов Smooth Streaming, DASH или HLS вам необходимо предоставить пользователю URL-адрес для потоковой передачи.

> [!NOTE]
> При добавлении или обновлении политики доставки ресурсов необходимо удалить все существующие указатели и создать новый.
> 
> 

Указания по публикации ресурса и созданию URL-адреса потоковой передачи см. в статье [Создание URL-адреса потоковой передачи](media-services-deliver-streaming-content.md).

## <a name="get-a-test-token"></a>Получение маркера тестирования
Получите маркер тестирования в зависимости от ограничения по маркеру, заданного в политике авторизации ключа.

```csharp
    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate = 
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the data in the given TokenRestrictionTemplate.
    //The GenerateTestToken method returns the token without the word "Bearer" in front
    //so you have to add it in front of the token string. 
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
```

Для проверки потока можно использовать [проигрыватель служб мультимедиа Azure](https://aka.ms/azuremediaplayer).

## <a name="how-can-your-client-request-a-key-from-the-key-delivery-service"></a><a id="client_request"></a>Отправка запроса клиента на получение ключа в службе доставки ключей
На предыдущем этапе вы создали URL-адрес, указывающий на файл манифеста. Чтобы отправить запрос в службу доставки ключей, клиент должен извлечь необходимые сведения из файлов манифеста потоковой передачи.

### <a name="manifest-files"></a>Файлы манифестов
Клиенту необходимо извлечь из файла манифеста значение URL-адреса (который также содержит идентификатор ключа "kid"). Затем клиент пытается получить ключ шифрования в службе доставки ключей. Клиенту также необходимо извлечь значение вектора инициализации и использовать его для расшифровки потока. В следующем фрагменте кода показан элемент `<Protection>` манифеста Smooth Streaming.

```xml
    <Protection>
      <ProtectionHeader SystemID="B47B251A-2409-4B42-958E-08DBAE7B4EE9">
        <ContentProtection xmlns:sea="urn:mpeg:dash:schema:sea:2012" schemeIdUri="urn:mpeg:dash:sea:2012">
          <sea:SegmentEncryption schemeIdUri="urn:mpeg:dash:sea:aes128-cbc:2013"/>
          <sea:KeySystem keySystemUri="urn:mpeg:dash:sea:keysys:http:2013"/>
          <sea:CryptoPeriod IV="0xD7D7D7D7D7D7D7D7D7D7D7D7D7D7D7D7" 
                            keyUriTemplate="https://wamsbayclus001kd-hs.cloudapp.net/HlsHandler.ashx?
                                            kid=da3813af-55e6-48e7-aa9f-a4d6031f7b4d"/>
        </ContentProtection>
      </ProtectionHeader>
    </Protection>
```

При использовании протокола HLS корневой манифест разбивается на файлы сегментов. 

Например, корневой манифест: http: \/ /test001.Origin.mediaservices.Windows.NET/8bfe7d6f-34e3-4d1a-b289-3e48a8762490/BigBuckBunny.ISM/MANIFEST (Format = m3u8-AAPL). Он содержит список имен файлов сегментов.

```text
. . . 
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=630133,RESOLUTION=424x240,CODECS="avc1.4d4015,mp4a.40.2",AUDIO="audio"
QualityLevels(514369)/Manifest(video,format=m3u8-aapl)
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=965441,RESOLUTION=636x356,CODECS="avc1.4d401e,mp4a.40.2",AUDIO="audio"
QualityLevels(842459)/Manifest(video,format=m3u8-aapl)
…
```

Если вы откроете один из файлов сегментов в текстовом редакторе (например, http: \/ /test001.Origin.mediaservices.Windows.NET/8bfe7d6f-34e3-4d1a-b289-3e48a8762490/BigBuckBunny.ISM/QualityLevels (514369)/MANIFEST (Video, Format = m3u8-AAPL), он содержит #EXT-X-Key, что означает, что файл зашифрован.

```text
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-ALLOW-CACHE:NO
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:9
#EXT-X-KEY:METHOD=AES-128,
URI="https://wamsbayclus001kd-hs.cloudapp.net/HlsHandler.ashx?
        kid=da3813af-55e6-48e7-aa9f-a4d6031f7b4d",
        IV=0XD7D7D7D7D7D7D7D7D7D7D7D7D7D7D7D7
#EXT-X-PROGRAM-DATE-TIME:1970-01-01T00:00:00.000+00:00
#EXTINF:8.425708,no-desc
Fragments(video=0,format=m3u8-aapl)
#EXT-X-ENDLIST
```

>[!NOTE] 
>Если вы планируете воспроизводить HLS с шифрованием AES в Safari, см. [этот блог](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).

### <a name="request-the-key-from-the-key-delivery-service"></a>Запрос ключа в службе доставки ключей

В следующем коде показано, как отправить запрос в службу доставки ключей служб мультимедиа, используя код URI доставки ключа (извлеченный из манифеста) и маркер. Получение простых веб-маркеров из службы маркеров безопасности в этой статье не рассматривается.

```csharp
    private byte[] GetDeliveryKey(Uri keyDeliveryUri, string token)
    {
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(keyDeliveryUri);

        request.Method = "POST";
        request.ContentType = "text/xml";
        if (!string.IsNullOrEmpty(token))
        {
            request.Headers[AuthorizationHeader] = token;
        }
        request.ContentLength = 0;

        var response = request.GetResponse();

        var stream = response.GetResponseStream();
        if (stream == null)
        {
            throw new NullReferenceException("Response stream is null");
        }

        var buffer = new byte[256];
        var length = 0;
        while (stream.CanRead && length <= buffer.Length)
        {
            var nexByte = stream.ReadByte();
            if (nexByte == -1)
            {
                break;
            }
            buffer[length] = (byte)nexByte;
            length++;
        }
        response.Close();

        // AES keys must be exactly 16 bytes (128 bits).
        var key = new byte[length];
        Array.Copy(buffer, key, length);
        return key;
    }
```

## <a name="protect-your-content-with-aes-128-by-using-net"></a>Защита содержимого с помощью AES-128 и .NET

### <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

1. Настройте среду разработки и укажите в файле app.config сведения о подключении, как описано в статье [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md).

2. Добавьте следующие элементы в "appSettings", определенные в файле app.config:

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

### <a name="example"></a><a id="example"></a>Пример

Замените код в файле Program.cs кодом, приведенным в этом разделе.
 
>[!NOTE]
>Действует ограничение в 1 000 000 записей для разных политик служб мультимедиа (например, для политики Locator или ContentKeyAuthorizationPolicy). Указывайте один и тот же идентификатор политики, если вы всегда используете одни и те же даты и разрешения доступа. Например, это политики для указателей, которые должны оставаться на месте в течение длительного времени (не политики передачи). Дополнительные сведения см. в разделе "Ограничение политик доступа" статьи [Управление активами и связанными сущностями с помощью пакета SDK служб мультимедиа для .NET](media-services-dotnet-manage-entities.md#limit-access-policies).

Обязательно обновите переменные, чтобы они указывали на папки, в которых находятся входные файлы.

[!code-csharp[Main](../../../samples-mediaservices-encryptionaes/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs)]

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
