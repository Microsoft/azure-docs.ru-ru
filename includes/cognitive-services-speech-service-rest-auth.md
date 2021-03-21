---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 01/08/2021
ms.author: erhopf
ms.openlocfilehash: 22127f81d871fe333750020196540db17e7544f7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98033469"
---
## <a name="authentication"></a>Аутентификация

Для каждого запроса требуется заголовок Authorization. В этой таблице показано, какие заголовки поддерживаются для каждой службы:

| Поддерживаемые заголовки авторизации | Преобразование речи в текст | Преобразование текста в речь |
|------------------------|----------------|----------------|
| Ocp-Apim-Subscription-Key | Да | Да |
| Authorization: Bearer | Да | Да |

При использовании заголовка `Ocp-Apim-Subscription-Key` необходимо предоставить только ключ подписки. Пример:

```http
'Ocp-Apim-Subscription-Key': 'YOUR_SUBSCRIPTION_KEY'
```

При использовании заголовка `Authorization: Bearer` необходимо выполнять запрос к конечной точке `issueToken`. В этом запросе ключ подписки обменивается на маркер доступа, действительный в течение 10 минут. В следующих нескольких разделах вы узнаете, как получить маркер и использовать маркер.

### <a name="how-to-get-an-access-token"></a>Как получить маркер доступа

Чтобы получить маркер доступа, необходимо отправить запрос в конечную точку `issueToken`, используя заголовок `Ocp-Apim-Subscription-Key` и ключ подписки.

`issueToken`Конечная точка имеет следующий формат:

```http
https://<REGION_IDENTIFIER>.api.cognitive.microsoft.com/sts/v1.0/issueToken
```

Замените `<REGION_IDENTIFIER>` идентификатором, соответствующим региону подписки, из следующей таблицы:

[!INCLUDE [](cognitive-services-speech-service-region-identifier.md)]

Используйте эти примеры, чтобы создать запрос на получение маркера доступа.

#### <a name="http-sample"></a>Пример HTTP

Это простой HTTP-запрос для получения маркера. Замените `YOUR_SUBSCRIPTION_KEY` своим ключом подписки на службу распознавания речи. Если ваша подписка не находится в регионе "Западная часть США", замените заголовок `Host` на имя узла в вашем регионе.

```http
POST /sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
```

Тело ответа содержит маркер доступа в формате JSON Web Token (JWT).

#### <a name="powershell-sample"></a>Пример для PowerShell

Это простой сценарий PowerShell для получения маркера доступа. Замените `YOUR_SUBSCRIPTION_KEY` своим ключом подписки на службу распознавания речи. Обязательно используйте правильную конечную точку для региона, который соответствует вашей подписке. В этом примере используется конечная точка для западной части США.

```powershell
$FetchTokenHeader = @{
  'Content-type'='application/x-www-form-urlencoded';
  'Content-Length'= '0';
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
}

$OAuthToken = Invoke-RestMethod -Method POST -Uri https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken
 -Headers $FetchTokenHeader

# show the token received
$OAuthToken

```

#### <a name="curl-sample"></a>Пример cURL

cURL — это программа командной строки, доступная в Linux (и в подсистеме Windows для Linux). Эта команда cURL показывает, как получить маркер доступа. Замените `YOUR_SUBSCRIPTION_KEY` своим ключом подписки на службу распознавания речи. Обязательно используйте правильную конечную точку для региона, который соответствует вашей подписке. В этом примере используется конечная точка для западной части США.

```console
curl -v -X POST \
 "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken" \
 -H "Content-type: application/x-www-form-urlencoded" \
 -H "Content-Length: 0" \
 -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

#### <a name="c-sample"></a>Пример на языке C#

Этот класс C# показывает, как получить маркер доступа. Передайте ключ подписки на службу распознавания речи при создании экземпляра класса. Если регион вашей подписки — не западная часть США, измените значение `FetchTokenUri` в соответствии с регионом своей подписки.

```csharp
public class Authentication
{
    public static readonly string FetchTokenUri =
        "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
    private string subscriptionKey;
    private string token;

    public Authentication(string subscriptionKey)
    {
        this.subscriptionKey = subscriptionKey;
        this.token = FetchTokenAsync(FetchTokenUri, subscriptionKey).Result;
    }

    public string GetAccessToken()
    {
        return this.token;
    }

    private async Task<string> FetchTokenAsync(string fetchUri, string subscriptionKey)
    {
        using (var client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
            UriBuilder uriBuilder = new UriBuilder(fetchUri);

            var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
            Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
            return await result.Content.ReadAsStringAsync();
        }
    }
}
```

#### <a name="python-sample"></a>Пример на языке Python

```python
# Request module must be installed.
# Run pip install requests if necessary.
import requests

subscription_key = 'REPLACE_WITH_YOUR_KEY'


def get_token(subscription_key):
    fetch_token_url = 'https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken'
    headers = {
        'Ocp-Apim-Subscription-Key': subscription_key
    }
    response = requests.post(fetch_token_url, headers=headers)
    access_token = str(response.text)
    print(access_token)
```

### <a name="how-to-use-an-access-token"></a>Как использовать маркер доступа

Маркер доступа должен отправляться в службу в виде заголовка `Authorization: Bearer <TOKEN>`. Каждый маркер доступа действителен в течение 10 минут. Вы можете получить новый маркер в любое время, но чтобы уменьшить сетевой трафик и задержку, мы рекомендуем использовать один маркер в течение девяти минут.

Ниже приведен пример HTTP-запроса к REST API преобразования речи в текст для коротких аудио:

```http
POST /cognitiveservices/v1 HTTP/1.1
Authorization: Bearer YOUR_ACCESS_TOKEN
Host: westus.stt.speech.microsoft.com
Content-type: application/ssml+xml
Content-Length: 199
Connection: Keep-Alive

// Message body here...
```
