---
title: Использование API Индексатора видео
titleSuffix: Azure Media Services
description: В этой статье описывается, как приступить к работе с API Индексатора видео Служб мультимедиа Azure.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 01/07/2021
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: fcd194e2503610db314f6a975a4afb1d27962f8c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98028218"
---
# <a name="tutorial-use-the-video-indexer-api"></a>Руководство по Использование API Индексатора видео

Индексатор видео — это интегрируемая служба на основе разных технологий искусственного интеллекта (ИИ) для анализа аудио и видео от корпорации Майкрософт, которые упрощает разработку. Интерфейсы API предназначены для того, чтобы разработчики могли сосредоточиться на использовании технологий ИИ для анализа мультимедиа, не беспокоясь о масштабировании, глобальном охвате, доступности и надежности облачной платформы. Эти API можно использовать для передачи файлов, извлечения аналитических сведений из видео, получения URL-адресов внедряемых мини-приложений анализа и воспроизведения, а также для множества других задач.

При создании учетной записи Индексатора видео можно выбрать бесплатную пробную учетную запись (с определенным количеством бесплатных минут индексирования) или платную учетную запись (без ограничивающих квот). В бесплатной пробной учетной записи Индексатора видео предоставляется до 600 минут бесплатной индексации для пользователей веб-сайта и до 2400 минут бесплатной индексации для пользователей API. Платный вариант подразумевает создание учетной записи Индексатора видео, [подключенной к подписке Azure и учетной записи Служб мультимедиа Azure](connect-to-azure.md). Оплата осуществляется за индексированные минуты. См. страницу с [ценами на Службы мультимедиа](https://azure.microsoft.com/pricing/details/media-services/).

В этой статье объясняется, как разработчики могут воспользоваться преимуществами [API Индексатора видео](https://api-portal.videoindexer.ai/).

## <a name="subscribe-to-the-api"></a>Оформление подписки на API

1. Войдите на [портал разработчика Индексатора видео](https://api-portal.videoindexer.ai/).

    Просмотрите Примечание о выпуске в отношении [сведений для входа](release-notes.md#october-2020).
    
     ![Вход на Портал разработчика Индексатора видео](./media/video-indexer-use-apis/sign-in.png)

   > [!Important]
   > * Укажите того же поставщика, который вы использовали при регистрации в Индексаторе видео.
   > * Личные учетные записи Google и Майкрософт (Outlook и Live) можно использовать только в качестве пробных учетных записей. Для учетных записей, подключенных к Azure, требуется Azure AD.
   > * На один адрес электронной почты может быть зарегистрирована только одна активная учетная запись. Если пользователь пытается войти с помощью учетной записи user@gmail.com для LinkedIn, а после этого с помощью учетной записи user@gmail.com для Google, отобразится страница с ошибкой о том, что такой пользователь уже существует.
2. Оформите подписку.

    Выберите вкладку [Продукты](https://api-portal.videoindexer.ai/products). Затем щелкните Authorization и подпишитесь.
    
    ![Вкладка "Продукты" на Портале разработчика Индексатора видео](./media/video-indexer-use-apis/authorization.png)

    > [!NOTE]
    > Новые пользователи автоматически подписываются на API Authorization.
    
    После подписки вы сможете найти свою подписку в разделе   ->  **авторизация** продуктов. На странице Подписка вы найдете первичный и вторичный ключи. Ключи должны быть защищены. Их нужно использовать только в серверном коде. Они не должны быть доступны на стороне клиента (в файлах с расширением JS, HTML и т. д.).

    ![Подписка и ключи на Портале разработчика Индексатора видео](./media/video-indexer-use-apis/subscriptions.png)

> [!TIP]
> Пользователь Индексатора видео может использовать один ключ подписки для доступа к нескольким учетным записям службы. Затем эти учетные записи Индексатора видео можно связать с различными учетными записями Служб мультимедиа.

## <a name="obtain-access-token-using-the-authorization-api"></a>Получение маркера доступа с помощью API авторизации

Как только вы подпишитесь на API авторизации, вы сможете получать маркеры доступа. Эти маркеры доступа используются для аутентификации в API операций.

Каждый вызов к API операций должен быть связан с маркером доступа, соответствующим области авторизации вызова.

- Уровень пользователя. Маркеры доступа этого уровня позволяют выполнять операции на уровне **пользователя**. Например, получить сведения о соответствующих учетных записях.
- Уровень учетной записи. Маркеры доступа этого уровня позволяют выполнять операции на уровне **учетной записи** или **видео**. Например, передать видео, вывести список всех видео, получить аналитические сведения о видео и т. д.
- Уровень видео. Маркеры доступа этого уровня позволяют выполнять операции с определенным **видео**. Например, получить аналитические сведения о видео, скачать субтитры, получить мини-приложения и т. д.

Вы можете выбрать, предоставляют ли эти маркеры доступ только на чтение или изменение, указав для параметра **allowEdit** значение true или false.

Для большинства сценариев взаимодействия между серверами, скорее всего, потребуется один маркер **учетной записи**, так как он распространяется на операции с **учетной записью** и **видео**. Но если вы планируете выполнять клиентские вызовы к Индексатору видео (например, из JavaScript), следует использовать маркер доступа уровня **видео**, чтобы запретить клиентам доступ к учетной записи в целом. Также его следует использовать, так как при внедрении клиентского кода Индексатора видео в свой клиент (например, с помощью мини-приложения **Get Insights Widget** или **Get Player Widget**) необходимо предоставить маркер доступа уровня **видео**.

Чтобы облегчить задачу, выберите API **Authorization** > **GetAccounts**, чтобы сначала получить список учетных записей без маркера доступа уровня пользователя. Также можно отправить запрос на получение учетных записей с действующими маркерами, чтобы не выполнять дополнительный вызов на получение токена учетной записи.

Срок действия маркеров доступа истекает через 1 час. Убедитесь, что маркер доступа действителен, прежде чем использовать API операций. Если срок действия истек, вызовите API авторизации еще раз, чтобы получить новый маркер доступа.

Все готово к началу интеграции с API. См. [подробное описание каждого API REST Индексатора видео](https://api-portal.videoindexer.ai/).

## <a name="account-id"></a>Идентификатор учетной записи

Параметр идентификатора учетной записи является обязательным во всех вызовах к API для операций. Идентификатор учетной записи — это GUID, который можно получить так:

* Воспользуйтесь веб-сайтом **Индексатора видео**, чтобы получить идентификатор учетной записи:

    1. Откройте веб-сайт [Индексатора видео](https://www.videoindexer.ai/) и выполните вход.
    2. Перейдите на страницу **Параметры**.
    3. Скопируйте идентификатор учетной записи.

        ![Параметры Индексатора видео и идентификатор учетной записи](./media/video-indexer-use-apis/account-id.png)

* Получите идентификатор учетной записи с помощью программных средств на **портале разработка Индексатора видео**.

    Используйте API [получения учетных записей](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Get-Account?).

    > [!TIP]
    > Вы можете создать маркеры доступа для учетных записей, определив `generateAccessTokens=true`.

* Получите идентификатор учетной записи из URL-адреса страницы проигрывателя в своей учетной записи.

    При просмотре видео идентификатор отображается после раздела `accounts` и перед разделом `videos`.

    ```
    https://www.videoindexer.ai/accounts/00000000-f324-4385-b142-f77dacb0a368/videos/d45bf160b5/
    ```

## <a name="recommendations"></a>Рекомендации

В этом разделе приводятся некоторые рекомендации по использованию API Индексатора видео.

- Если вы планируете отправить видео, рекомендуется поместить файл в какое-либо общедоступное сетевое расположение (например, учетную запись хранилища BLOB-объектов Azure). Получите ссылку на видео и укажите URL-адрес в качестве значения для параметра отправки файла.

    URL-адрес, предоставляемый Индексатору видео, должен указывать на файл мультимедиа (аудио или видео). Простая проверка URL-адреса (или URL-адреса SAS) заключается в том, чтобы вставить его в браузер, если файл начнет воспроизводиться или загружаться, скорее всего, это хороший URL-адрес. Если в браузере отображается визуализация, скорее всего, это ссылка не на файл, а на HTML-страницу.

- Если вы вызываете API, чтобы получить аналитические сведения для выбранного видео, вы получите подробные выходные данные в формате JSON в виде содержимого ответа. См. дополнительные сведения о [возвращаемом содержимом JSON](video-indexer-output-json-v2.md).

## <a name="code-sample"></a>Пример кода

В следующем фрагменте кода C# показано, как использовать все API Индексатора видео.

```csharp
var apiUrl = "https://api.videoindexer.ai";
var accountId = "..."; 
var location = "westus2"; // replace with the account's location, or with “trial” if this is a trial account
var apiKey = "..."; 

System.Net.ServicePointManager.SecurityProtocol = System.Net.ServicePointManager.SecurityProtocol | System.Net.SecurityProtocolType.Tls12;

// create the http client
var handler = new HttpClientHandler(); 
handler.AllowAutoRedirect = false; 
var client = new HttpClient(handler);
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey); 

// obtain account access token
var accountAccessTokenRequestResult = client.GetAsync($"{apiUrl}/auth/{location}/Accounts/{accountId}/AccessToken?allowEdit=true").Result;
var accountAccessToken = accountAccessTokenRequestResult.Content.ReadAsStringAsync().Result.Replace("\"", "");

client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

// upload a video
var content = new MultipartFormDataContent();
Debug.WriteLine("Uploading...");
// get the video from URL
var videoUrl = "VIDEO_URL"; // replace with the video URL

// as an alternative to specifying video URL, you can upload a file.
// remove the videoUrl parameter from the query string below and add the following lines:
  //FileStream video =File.OpenRead(Globals.VIDEOFILE_PATH);
  //byte[] buffer = new byte[video.Length];
  //video.Read(buffer, 0, buffer.Length);
  //content.Add(new ByteArrayContent(buffer));

var uploadRequestResult = client.PostAsync($"{apiUrl}/{location}/Accounts/{accountId}/Videos?accessToken={accountAccessToken}&name=some_name&description=some_description&privacy=private&partition=some_partition&videoUrl={videoUrl}", content).Result;
var uploadResult = uploadRequestResult.Content.ReadAsStringAsync().Result;

// get the video id from the upload result
var videoId = JsonConvert.DeserializeObject<dynamic>(uploadResult)["id"];
Debug.WriteLine("Uploaded");
Debug.WriteLine("Video ID: " + videoId);           

// obtain video access token
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);
var videoTokenRequestResult = client.GetAsync($"{apiUrl}/auth/{location}/Accounts/{accountId}/Videos/{videoId}/AccessToken?allowEdit=true").Result;
var videoAccessToken = videoTokenRequestResult.Content.ReadAsStringAsync().Result.Replace("\"", "");

client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

// wait for the video index to finish
while (true)
{
  Thread.Sleep(10000);

  var videoGetIndexRequestResult = client.GetAsync($"{apiUrl}/{location}/Accounts/{accountId}/Videos/{videoId}/Index?accessToken={videoAccessToken}&language=English").Result;
  var videoGetIndexResult = videoGetIndexRequestResult.Content.ReadAsStringAsync().Result;

  var processingState = JsonConvert.DeserializeObject<dynamic>(videoGetIndexResult)["state"];

  Debug.WriteLine("");
  Debug.WriteLine("State:");
  Debug.WriteLine(processingState);

  // job is finished
  if (processingState != "Uploaded" && processingState != "Processing")
  {
      Debug.WriteLine("");
      Debug.WriteLine("Full JSON:");
      Debug.WriteLine(videoGetIndexResult);
      break;
  }
}

// search for the video
var searchRequestResult = client.GetAsync($"{apiUrl}/{location}/Accounts/{accountId}/Videos/Search?accessToken={accountAccessToken}&id={videoId}").Result;
var searchResult = searchRequestResult.Content.ReadAsStringAsync().Result;
Debug.WriteLine("");
Debug.WriteLine("Search:");
Debug.WriteLine(searchResult);

// get insights widget url
var insightsWidgetRequestResult = client.GetAsync($"{apiUrl}/{location}/Accounts/{accountId}/Videos/{videoId}/InsightsWidget?accessToken={videoAccessToken}&widgetType=Keywords&allowEdit=true").Result;
var insightsWidgetLink = insightsWidgetRequestResult.Headers.Location;
Debug.WriteLine("Insights Widget url:");
Debug.WriteLine(insightsWidgetLink);

// get player widget url
var playerWidgetRequestResult = client.GetAsync($"{apiUrl}/{location}/Accounts/{accountId}/Videos/{videoId}/PlayerWidget?accessToken={videoAccessToken}").Result;
var playerWidgetLink = playerWidgetRequestResult.Headers.Location;
Debug.WriteLine("");
Debug.WriteLine("Player Widget url:");
Debug.WriteLine(playerWidgetLink);

```

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим руководством удалите ресурсы, которые вы не планируете использовать.

## <a name="see-also"></a>См. также раздел

- [Общие сведения об Индексаторе видео](video-indexer-overview.md)
- [Регионы](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services)

## <a name="next-steps"></a>Дальнейшие действия

- [Анализ сведений о возвращаемом содержимом JSON](video-indexer-output-json-v2.md)
- Ознакомьтесь с [примером кода](https://github.com/Azure-Samples/media-services-video-indexer), демонстрирующим важные аспекты передачи и индексирования видео. Следуя коду, вы получите хорошее представление об использовании нашего API для базовых функций. Обязательно ознакомьтесь со встроенными комментариями и обратите внимание на рекомендации.

