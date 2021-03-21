---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 10/20/2020
ms.author: trbye
ms.openlocfilehash: 502729e4c90a6d6ffa424ea7f379e2b426b3e3e0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102445600"
---
## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Сначала необходимо установить <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">пакет SDK службы "Речь" для JavaScript </a>. В зависимости от используемой платформы выполните следующие действия:

- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=nodejs#get-the-speech-sdk" target="_blank">Node.js <span 
class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=browser#get-the-speech-sdk" target="_blank">Веб-браузер </a>

## <a name="create-voice-signatures"></a>Создание подписок

Первым шагом является создание голосовых подписей для участников беседы, чтобы их можно было идентифицировать в качестве уникальных докладчиков. Входной `.wav` звуковой файл для создания подписок должен иметь 16-разрядную частоту дискретизации, 16 кГц и одноканальный (Mono) формат. Рекомендуемая длина для каждого звукового образца — от тридцати секунд до двух минут. Этот `.wav` файл должен представлять собой образец голоса **одного пользователя** , чтобы создать уникальный профиль голоса.

В следующем примере показано, как создать голосовую подпись с [помощью REST API](https://aka.ms/cts/signaturegenservice) в JavaScript. Обратите внимание, что необходимо заменить реальную информацию для `subscriptionKey` , `region` и путь к образцу `.wav` файла.

```javascript
const fs = require('fs');
const axios = require('axios');
const formData = require('form-data');
 
const subscriptionKey = 'your-subscription-key';
const region = 'your-region';
 
async function createProfile() {
    let form = new formData();
    form.append('file', fs.createReadStream('path-to-voice-sample.wav'));
    let headers = form.getHeaders();
    headers['Ocp-Apim-Subscription-Key'] = subscriptionKey;
 
    let url = `https://signature.${region}.cts.speech.microsoft.com/api/v1/Signature/GenerateVoiceSignatureFromFormData`;
    let response = await axios.post(url, form, { headers: headers });
    
    // get signature from response, serialize to json string
    return JSON.stringify(response.data.Signature);
}
 
async function main() {
    // use this voiceSignature string with conversation transcription calls below
    let voiceSignatureString = await createProfile();
    console.log(voiceSignatureString);
}
main();
```

Выполнение этого скрипта возвращает строку подписи голоса в переменной `voiceSignatureString` . Дважды запустите функцию, чтобы в качестве входных переменных использовались две строки `voiceSignatureStringUser1` `voiceSignatureStringUser2` .

> [!NOTE]
> Подписи можно создавать **только** с помощью REST API.

## <a name="transcribe-conversations"></a>Транскрипция беседы

В следующем примере кода показано, как транскрипция беседы в режиме реального времени для двух динамиков. Предполагается, что вы уже создали строки голосовой подписи для каждого динамика, как показано выше. Замените реальную информацию для `subscriptionKey` , `region` и путь к `filepath` звуковому файлу, который вы хотите транскрипция.

Этот пример кода выполняет следующие действия.

* Создает поток push-уведомлений, используемый для записи, и записывает в `.wav` него образец файла.
* Создает `Conversation` с помощью `createConversationAsync()` .
* Создает `ConversationTranscriber` с помощью конструктора.
* Добавляет участников в диалог. Строки `voiceSignatureStringUser1` и `voiceSignatureStringUser2` должны поступать в качестве выходных данных шагов, описанных выше.
* Регистрируется в событиях и начинает транскрипцию.

```javascript
(function() {
    "use strict";
    var sdk = require("microsoft-cognitiveservices-speech-sdk");
    var fs = require("fs");
    
    var subscriptionKey = "your-subscription-key";
    var region = "your-region";
    var filepath = "audio-file-to-transcribe.wav"; // 8-channel audio
    
    // create the push stream and write file to it
    var pushStream = sdk.AudioInputStream.createPushStream();
    fs.createReadStream(filepath).on('data', function(arrayBuffer) {
        pushStream.write(arrayBuffer.slice());
    }).on('end', function() {
        pushStream.close();
    });
    
    var speechTranslationConfig = sdk.SpeechTranslationConfig.fromSubscription(subscriptionKey, region);
    var audioConfig = sdk.AudioConfig.fromStreamInput(pushStream);
    speechTranslationConfig.speechRecognitionLanguage = "en-US";
    
    // create conversation and transcriber
    var conversation = sdk.Conversation.createConversationAsync(speechTranslationConfig, "myConversation");
    var transcriber = new sdk.ConversationTranscriber(audioConfig);
    
    // attach the transcriber to the conversation
    transcriber.joinConversationAsync(conversation,
    function () {
        // add first participant using voiceSignature created in enrollment step
        var user1 = sdk.Participant.From("user1@example.com", "en-us", voiceSignatureStringUser1);
        conversation.addParticipantAsync(user1,
        function () {
            // add second participant using voiceSignature created in enrollment step
            var user2 = sdk.Participant.From("user2@example.com", "en-us", voiceSignatureStringUser2);
            conversation.addParticipantAsync(user2,
            function () {
                transcriber.sessionStarted = function(s, e) {
                console.log("(sessionStarted)");
                };
                transcriber.sessionStopped = function(s, e) {
                console.log("(sessionStopped)");
                };
                transcriber.canceled = function(s, e) {
                console.log("(canceled)");
                };
                transcriber.transcribed = function(s, e) {
                console.log("(transcribed) text: " + e.result.text);
                console.log("(transcribed) speakerId: " + e.result.speakerId);
                };
    
                // begin conversation transcription
                transcriber.startTranscribingAsync(
                function () { },
                function (err) {
                    console.trace("err - starting transcription: " + err);
                });
        },
        function (err) {
            console.trace("err - adding user1: " + err);
        });
    },
    function (err) {
        console.trace("err - adding user2: " + err);
    });
    },
    function (err) {
    console.trace("err - " + err);
    });
}()); 
```