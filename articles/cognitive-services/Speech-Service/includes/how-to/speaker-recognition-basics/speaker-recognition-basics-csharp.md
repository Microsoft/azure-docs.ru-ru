---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 09/28/2020
ms.author: v-jawe
ms.custom: references_regions
ms.openlocfilehash: ad7f6f600082a407a94833fe1575136fc360c9ae
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105726"
---
В этом руководстве показаны базовые шаблоны разработки для Распознавания говорящего с помощью пакета SDK службы Речи. В статье рассматриваются следующие темы:

* Проверка, зависящая от текста и не зависящая от текста.
* Идентификация говорящего для идентификации образца голоса в группе голосов.
* Удаление речевых профилей.

Общие сведения о концепциях распознавания речи см. в статье [Обзор](../../../speaker-recognition-overview.md).

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

> [!IMPORTANT]
> Распознавание говорящего в настоящее время поддерживается *только* в ресурсах распознавания речи Azure, созданных в регионе `westus`.

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Прежде чем выполнять какие-либо действия, необходимо установить пакет SDK для службы "Речь". В зависимости от используемой платформы выполните следующие действия:

* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=dotnet" target="_blank">.NET Framework </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=dotnetcore" target="_blank">.NET Core </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=unity" target="_blank">Unity </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=uwps" target="_blank">UWP </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=xaml" target="_blank">Xamarin </a>

## <a name="import-dependencies"></a>Импорт зависимостей

Чтобы выполнить примеры из этой статьи, добавьте в начале скрипта следующие инструкции `using`:

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
```

## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Чтобы вызвать службу "Речь" с помощью пакета SDK для службы "Речь", необходимо создать [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig). В этом примере создается [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig) с использованием ключа и региона подписки. Для оставшейся части этой статьи также создается стандартный код, который необходимо будет изменить, чтобы внести различные настройки.

Обратите внимание, что для региона задано значение `westus`, так как это единственный поддерживаемый регион для этой службы.

```csharp
public class Program 
{
    static async Task Main(string[] args)
    {
        // replace with your own subscription key 
        string subscriptionKey = "YourSubscriptionKey";
        string region = "westus";
        var config = SpeechConfig.FromSubscription(subscriptionKey, region);
    }
}
```

## <a name="text-dependent-verification"></a>Проверка, зависящая от текста

Проверка говорящего является подтверждением того, что говорящий соответствует известному или **зарегистрированному** голосу. В первую очередь необходимо **зарегистрировать** речевой профиль, чтобы службе было с чем сравнивать будущие примеры голоса. В этом примере профиль регистрируется с помощью стратегии **зависимости от текста**, которая требует произнесения определенной парольной фразы для регистрации и проверки. Список поддерживаемых парольных фраз см. в [справочной документации](/rest/api/speakerrecognition/).

Начните с создания следующей функции в классе `Program`, чтобы зарегистрировать речевой профиль.

```csharp
public static async Task VerificationEnroll(SpeechConfig config, Dictionary<string, string> profileMapping)
{
    using (var client = new VoiceProfileClient(config))
    using (var profile = await client.CreateProfileAsync(VoiceProfileType.TextDependentVerification, "en-us"))
    {
        using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
        {
            Console.WriteLine($"Enrolling profile id {profile.Id}.");
            // give the profile a human-readable display name
            profileMapping.Add(profile.Id, "Your Name");

            VoiceProfileEnrollmentResult result = null;
            while (result is null || result.RemainingEnrollmentsCount > 0)
            {
                Console.WriteLine("Speak the passphrase, \"My voice is my passport, verify me.\"");
                result = await client.EnrollProfileAsync(profile, audioInput);
                Console.WriteLine($"Remaining enrollments needed: {result.RemainingEnrollmentsCount}");
                Console.WriteLine("");
            }
            
            if (result.Reason == ResultReason.EnrolledVoiceProfile)
            {
                await SpeakerVerify(config, profile, profileMapping);
            }
            else if (result.Reason == ResultReason.Canceled)
            {
                var cancellation = VoiceProfileEnrollmentCancellationDetails.FromResult(result);
                Console.WriteLine($"CANCELED {profile.Id}: ErrorCode={cancellation.ErrorCode} ErrorDetails={cancellation.ErrorDetails}");
            }
        }
    }
}
```

В этой функции `await client.CreateProfileAsync()` — то, что фактически создает новый речевой профиль. После ее создания вы указываете, как будут вводиться звуковые образцы. В данном примере для записи звука с устройства ввода по умолчанию используется `AudioConfig.FromDefaultMicrophoneInput()`. Затем вы регистрируете звуковые образцы в цикле `while`, отслеживающем количество оставшихся образцов, которые должны быть зарегистрированы. В каждой итерации `client.EnrollProfileAsync(profile, audioInput)` будет предлагать произнести парольную фразу в микрофон и добавлять этот образец в речевой профиль.

После завершения регистрации вызывается `await SpeakerVerify(config, profile, profileMapping)` для проверки только что созданного профиля. Добавьте еще одну функцию для определения `SpeakerVerify`.

```csharp
public static async Task SpeakerVerify(SpeechConfig config, VoiceProfile profile, Dictionary<string, string> profileMapping)
{
    var speakerRecognizer = new SpeakerRecognizer(config, AudioConfig.FromDefaultMicrophoneInput());
    var model = SpeakerVerificationModel.FromProfile(profile);

    Console.WriteLine("Speak the passphrase to verify: \"My voice is my passport, please verify me.\"");
    var result = await speakerRecognizer.RecognizeOnceAsync(model);
    Console.WriteLine($"Verified voice profile for speaker {profileMapping[result.ProfileId]}, score is {result.Score}");
}
```

В этой функции вы передаете только что созданный объект `VoiceProfile`, чтобы инициализировать модель для проверки. Затем `await speakerRecognizer.RecognizeOnceAsync(model)` предложит снова произнести парольную фразу, но на этот раз она будет проверяться на соответствие речевому профилю, после чего будет возвращаться показатель подобия от 0,0 до 1,0. Объект `result` также возвращает `Accept` или `Reject` в зависимости от того, соответствует ли парольная фраза.

Затем измените функцию `Main()` для вызова новых созданных функций. Кроме того, обратите внимание, что вы создаете `Dictionary<string, string>` для передачи по ссылке через вызовы функции. Это связано с тем, что для обеспечения конфиденциальности служба не позволяет хранить удобное для восприятия человеком имя с созданным `VoiceProfile` и сохраняет только идентификационный номер. В функции `VerificationEnroll` вы добавляете в этот словарь запись с только что созданным идентификатором и текстовым именем. В сценариях разработки приложений, где необходимо отображать удобное для восприятия имя, **необходимо хранить это сопоставление где-нибудь еще, так как служба не может его хранить**.

```csharp
static async Task Main(string[] args)
{
    string subscriptionKey = "YourSubscriptionKey";
    string region = "westus";
    var config = SpeechConfig.FromSubscription(subscriptionKey, region);

    // persist profileMapping if you want to store a record of who the profile is
    var profileMapping = new Dictionary<string, string>();
    await VerificationEnroll(config, profileMapping);

    Console.ReadLine();
}
```

Запустите скрипт, и вам будет предложено произнести фразу *Мой голос — мой паспорт, проверьте меня* три раза для регистрации и еще один раз для проверки. Возвращаемый результат является показателем подобия, который можно использовать для создания собственных настраиваемых порогов для проверки.

```shell
Enrolling profile id 87-2cef-4dff-995b-dcefb64e203f.
Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 2

Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 1

Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 0

Speak the passphrase to verify: "My voice is my passport, verify me."
Verified voice profile for speaker Your Name, score is 0.915581
```

## <a name="text-independent-verification"></a>Проверка, не зависящая от текста

Проверка, **не зависящая от текста**, отличается от проверки, **зависящей от текста**, в следующих аспектах.

* Не требуется определенная парольная фраза, можно произносить что угодно.
* Не требуются три звуковых образца, но *требуется* 20 секунд звуковых данных.

Внесите несколько простых изменений в функцию `VerificationEnroll`, чтобы переключиться на проверку, **не зависящую от текста**. Сначала измените тип проверки на `VoiceProfileType.TextIndependentVerification`. Затем измените цикл `while`, чтобы он отслеживал `result.RemainingEnrollmentsSpeechLength` и запрос на произнесение речи выводился до тех пор, пока не будут записаны 20 секунд.

```csharp
public static async Task VerificationEnroll(SpeechConfig config, Dictionary<string, string> profileMapping)
{
    using (var client = new VoiceProfileClient(config))
    using (var profile = await client.CreateProfileAsync(VoiceProfileType.TextIndependentVerification, "en-us"))
    {
        using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
        {
            Console.WriteLine($"Enrolling profile id {profile.Id}.");
            // give the profile a human-readable display name
            profileMapping.Add(profile.Id, "Your Name");

            VoiceProfileEnrollmentResult result = null;
            while (result is null || result.RemainingEnrollmentsSpeechLength > TimeSpan.Zero)
            {
                Console.WriteLine("Continue speaking to add to the profile enrollment sample.");
                result = await client.EnrollProfileAsync(profile, audioInput);
                Console.WriteLine($"Remaining enrollment audio time needed: {result.RemainingEnrollmentsSpeechLength}");
                Console.WriteLine("");
            }
            
            if (result.Reason == ResultReason.EnrolledVoiceProfile)
            {
                await SpeakerVerify(config, profile, profileMapping);
            }
            else if (result.Reason == ResultReason.Canceled)
            {
                var cancellation = VoiceProfileEnrollmentCancellationDetails.FromResult(result);
                Console.WriteLine($"CANCELED {profile.Id}: ErrorCode={cancellation.ErrorCode} ErrorDetails={cancellation.ErrorDetails}");
            }
        }
    }
}
```

Запустите программу еще раз и произнесите что-нибудь на этапе проверки, так как парольная фраза не требуется. Снова возвращается показатель подобия.

```shell
Enrolling profile id 4tt87d4-f2d3-44ae-b5b4-f1a8d4036ee9.
Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:15.3200000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:09.8100008

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:05.1900000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:00.8700000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:00

Speak the passphrase to verify: "My voice is my passport, please verify me."
Verified voice profile for speaker Your Name, score is 0.849409
```

## <a name="speaker-identification"></a>Идентификация говорящего

Идентификация говорящего используется для определения, **кто** именно говорит из определенной группы зарегистрированных голосов. Этот процесс очень похож на проверку, **не зависящую от текста**. Основное отличие заключается в том, что можно сравнивать не с одним, а сразу с несколькими речевыми профилями.

Создайте функцию `IdentificationEnroll` для регистрации нескольких речевых профилей. Процесс регистрации каждого профиля аналогичен процессу регистрации при **проверке, не зависящей от текста**, и для каждого профиля требуется 20 секунд речи. Эта функция принимает список строк `profileNames` и создает новый речевой профиль для каждого имени в списке. Функция возвращает список объектов `VoiceProfile`, которые используются в следующей функции для определения говорящего.

```csharp
public static async Task<List<VoiceProfile>> IdentificationEnroll(SpeechConfig config, List<string> profileNames, Dictionary<string, string> profileMapping)
{
    List<VoiceProfile> voiceProfiles = new List<VoiceProfile>();
    using (var client = new VoiceProfileClient(config))
    {
        foreach (string name in profileNames)
        {
            using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
            {
                var profile = await client.CreateProfileAsync(VoiceProfileType.TextIndependentIdentification, "en-us");
                Console.WriteLine($"Creating voice profile for {name}.");
                profileMapping.Add(profile.Id, name);

                VoiceProfileEnrollmentResult result = null;
                while (result is null || result.RemainingEnrollmentsSpeechLength > TimeSpan.Zero)
                {
                    Console.WriteLine($"Continue speaking to add to the profile enrollment sample for {name}.");
                    result = await client.EnrollProfileAsync(profile, audioInput);
                    Console.WriteLine($"Remaining enrollment audio time needed: {result.RemainingEnrollmentsSpeechLength}");
                    Console.WriteLine("");
                }
                voiceProfiles.Add(profile);
            }
        }
    }
    return voiceProfiles;
}
```

Создайте следующую функцию `SpeakerIdentification`, чтобы отправить запрос на идентификацию. Основное отличие этой функции от запроса **проверки говорящего** заключается в использовании метода `SpeakerIdentificationModel.FromProfiles()`, который принимает список объектов `VoiceProfile`. 

```csharp
public static async Task SpeakerIdentification(SpeechConfig config, List<VoiceProfile> voiceProfiles, Dictionary<string, string> profileMapping) 
{
    var speakerRecognizer = new SpeakerRecognizer(config, AudioConfig.FromDefaultMicrophoneInput());
    var model = SpeakerIdentificationModel.FromProfiles(voiceProfiles);

    Console.WriteLine("Speak some text to identify who it is from your list of enrolled speakers.");
    var result = await speakerRecognizer.RecognizeOnceAsync(model);
    Console.WriteLine($"The most similar voice profile is {profileMapping[result.ProfileId]} with similarity score {result.Score}");
}
```

Измените функцию `Main()` следующим образом. Вы создаете список строк `profileNames`, который передаете в функцию `IdentificationEnroll()`. Появится запрос на создание нового речевого профиля для каждого имени в этом списке, поэтому можно добавить дополнительные имена для создания дополнительных профилей для друзей или коллег.

```csharp
static async Task Main(string[] args)
{
    // replace with your own subscription key 
    string subscriptionKey = "YourSubscriptionKey";
    string region = "westus";
    var config = SpeechConfig.FromSubscription(subscriptionKey, region);

    // persist profileMapping if you want to store a record of who the profile is
    var profileMapping = new Dictionary<string, string>();
    var profileNames = new List<string>() { "Your name", "A friend's name" };
    
    var enrolledProfiles = await IdentificationEnroll(config, profileNames, profileMapping);
    await SpeakerIdentification(config, enrolledProfiles, profileMapping);

    foreach (var profile in enrolledProfiles)
    {
        profile.Dispose();
    }
    Console.ReadLine();
}
```

Запустите скрипт, и появится предложение произнести что-нибудь для регистрации образцов голоса для первого профиля. После завершения регистрации вам будет предложено повторить эту процедуру для каждого имени в списке `profileNames`. После завершения всех регистраций будет предложено, чтобы **кто-нибудь** что-нибудь сказал, и служба попытается опознать этого пользователя на основе зарегистрированных речевых профилей.

В этом примере возвращается только ближайшее соответствие и показатель подобия, но можно получить полный ответ, включающий пять главных показателей подобия, добавив `string json = result.Properties.GetProperty(PropertyId.SpeechServiceResponse_JsonResult)` в вашу функцию `SpeakerIdentification`.

## <a name="changing-audio-input-type"></a>Изменение типа входных звуковых данных

В примерах, приведенных в этой статье, для записи образцов звука используется микрофон устройства по умолчанию. Если же необходимо использовать звуковые файлы вместо ввода с микрофона, просто измените любой экземпляр `AudioConfig.FromDefaultMicrophoneInput()` на `AudioConfig.FromWavFileInput(path/to/your/file.wav)`, чтобы переключиться на ввод файлов. Можно также использовать смешанные входные данные, например микрофон для регистрации и файлы для проверки.

## <a name="deleting-voice-profile-enrollments"></a>Удаление регистраций речевых профилей

Чтобы удалить зарегистрированный профиль, используйте функцию `DeleteProfileAsync()` в объекте `VoiceProfileClient`. В следующем примере функции показано, как удалить речевой профиль для известного идентификатора речевого профиля.

```csharp
public static async Task DeleteProfile(SpeechConfig config, string profileId) 
{
    using (var client = new VoiceProfileClient(config))
    {
        var profile = new VoiceProfile(profileId);
        await client.DeleteProfileAsync(profile);
    }
}
```