---
title: Устранение неполадок речевого пакета SDK — речевая служба
titleSuffix: Azure Cognitive Services
description: В этой статье содержится информация, которая поможет решить неполадки, которые могут возникнуть при использовании пакета SDK для распознавания речи.
services: cognitive-services
author: jhakulin
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/23/2019
ms.author: jhakulin
ms.openlocfilehash: 421b9adf4ae5d2c641484e646bea096716d46cca
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74815411"
---
# <a name="troubleshoot-the-speech-sdk"></a>Устранение неполадок с пакетом SDK для службы "Речь"

В этой статье содержится информация, которая поможет решить неполадки, которые могут возникнуть при использовании пакета SDK для распознавания речи.

## <a name="error-websocket-upgrade-failed-with-an-authentication-error-403"></a>Ошибка: Обновление WebSocket завершилось ошибкой проверки подлинности (403)

Может быть указана неправильная конечная точка для вашего региона или службы. Проверьте универсальный код ресурса (URI), чтобы убедиться в том, что он правильный.

Кроме того, могла возникнуть проблема с вашим ключом подписки или маркером проверки подлинности. Дополнительные сведения см. в следующем разделе.

## <a name="error-http-403-forbidden-or-http-401-unauthorized"></a>Ошибка: HTTP 403 запрещено или HTTP 401 не авторизовано

Эта ошибка часто вызвана проблемами проверки подлинности. Запросы на соединение без допуска `Ocp-Apim-Subscription-Key` или `Authorization` заголовка отклоняются по состоянию 403 или 401.

* Если вы используете ключ подписки для проверки подлинности, может произойти ошибка, потому что:

    - отсутствует или недействителен ключ подписки;
    - вы превысили квоту использования подписки;

* Если вы используете токен авторизации для проверки подлинности, может произойти ошибка, потому что:

    - токен авторизации недопустимый;
    - срок действия токена авторизации истек.

### <a name="validate-your-subscription-key"></a>Проверьте ключ подписки

Вы можете убедиться, что у вас есть допустимый ключ подписки, выполнив одну из следующих команд.

> [!NOTE]
> Замените `YOUR_SUBSCRIPTION_KEY` и `YOUR_REGION` своим собственным ключом подписки и ассоциированной областью.

* PowerShell

    ```Powershell
    $FetchTokenHeader = @{
      'Content-type'='application/x-www-form-urlencoded'
      'Content-Length'= '0'
      'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
    }
    $OAuthToken = Invoke-RestMethod -Method POST -Uri https://YOUR_REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken -Headers $FetchTokenHeader
    $OAuthToken
    ```

* cURL

    ```
    curl -v -X POST "https://YOUR_REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0"
    ```

Если вы ввели действительный ключ подписки, эта команда возвращает маркер авторизации. В противном случае возвращается ошибка.

### <a name="validate-an-authorization-token"></a>Проверьте маркер проверки подлинности

Если используется токен авторизации для аутентификации, выполните одну из приведенных ниже команд, чтобы убедиться, что токен авторизации по-прежнему допустимый. Токены допустимы в течение 10 минут.

> [!NOTE]
> Замените `YOUR_AUDIO_FILE` путем к предварительно записанному звуковому файлу. Замените `YOUR_ACCESS_TOKEN` маркером авторизации, возвращенным на предыдущем шаге. Замените `YOUR_REGION` правильным регионом.

* PowerShell

    ```Powershell
    $SpeechServiceURI =
    'https://YOUR_REGION.stt.speech.microsoft.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US'

    # $OAuthToken is the authorization token returned by the token service.
    $RecoRequestHeader = @{
      'Authorization' = 'Bearer '+ $OAuthToken
      'Transfer-Encoding' = 'chunked'
      'Content-type' = 'audio/wav; codec=audio/pcm; samplerate=16000'
    }

    # Read audio into byte array.
    $audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")

    $RecoResponse = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes

    # Show the result.
    $RecoResponse
    ```

* cURL

    ```
    curl -v -X POST "https://YOUR_REGION.stt.speech.microsoft.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US" -H "Authorization: Bearer YOUR_ACCESS_TOKEN" -H "Transfer-Encoding: chunked" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
    ```

Если вы ввели действительный ключ авторизации, эта команда возвращает расшифровку аудиофайла. В противном случае возвращается ошибка.

---

## <a name="error-http-400-bad-request"></a>Ошибка: HTTP 400 неверный запрос

Обычно эта ошибка возникает, когда текст запроса содержит недопустимые аудиоданные. Поддерживается только формат WAV. Также проверьте заголовки запроса, чтобы убедиться, что вы указываете соответствующие значения для `Content-Type` и `Content-Length`.

## <a name="error-http-408-request-timeout"></a>Ошибка: HTTP 408 Время ожидания запроса истекло

Ошибка возникла, скорее всего, из-за отсутствия аудиоданных в службе. Эта ошибка также может быть вызвана проблемами с сетью.

## <a name="recognitionstatus-in-the-response-is-initialsilencetimeout"></a>"RecognitionStatus" в ответе "InitialSilenceTimeout"

Обычно эта проблема возникает из-за аудиоданных. Это ошибка может появиться, потому что:

* В начале аудиозаписи есть продолжительный участок тишины. В этом случае служба прекращает распознавание через несколько секунд и вернет `InitialSilenceTimeout`.

* Аудиозапись создана с помощью неподдерживаемого кодека, что приводит к тому, что аудиоданные воспринимаются как тишина.

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр заметки о выпуске](releasenotes.md)
