---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.date: 04/04/2020
ms.topic: include
ms.author: trbye
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: f8caa38fa77134ef156360af2fd3d7dc5d84bb3d
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102444552"
---
## <a name="prerequisites"></a>Предварительные требования

Необходимые условия:

* <a href="~/articles/cognitive-services/Speech-Service/quickstarts/setup-platform.md?pivots=programming-language-python" target="_blank">Установите пакет SDK службы "Речь" для среды разработки и создайте пустой пример проекта</a>.

## <a name="create-a-luis-app-for-intent-recognition"></a>Создание приложения LUIS для распознавания намерений

[!INCLUDE [Create a LUIS app for intent recognition](../luis-sign-up.md)]

## <a name="open-your-project"></a>Открытие проекта

1. Откройте предпочтительную интегрированную среду разработки.
2. Создайте проект и файл с именем `quickstart.py`, а затем откройте его.

## <a name="start-with-some-boilerplate-code"></a>Добавление стандартного кода

Добавим код, который выступает в качестве основы для нашего проекта.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=5-7)]

## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Прежде чем инициализировать объект `IntentRecognizer`, необходимо создать конфигурацию, использующую ключ и расположение ресурса прогнозирования LUIS.

Вставьте этот код в `quickstart.py`. Обязательно обновите следующие значения:

* Замените `"YourLanguageUnderstandingSubscriptionKey"` ключом прогнозирования LUIS.
* Замените `"YourLanguageUnderstandingServiceRegion"` расположением LUIS. Используйте **идентификатор региона** из [региона](../../../../regions.md)

>[!TIP]
> Если вам нужна помощь с поиском этих значений, перейдите к разделу [Создание приложения LUIS для распознавания намерений](#create-a-luis-app-for-intent-recognition).

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=12)]

В этом примере создается объект `SpeechConfig` с использованием ключа и региона LUIS. Полный список доступных методов см. в статье [SpeechConfig Class](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig) (Класс SpeechConfig).

Пакет SDK для распознавания речи по умолчанию распознает использование языкового стандарта en-us. Дополнительные сведения о выборе исходного языка см. в разделе [Specify source language for speech to text](../../../../how-to-specify-source-language.md) (Указание исходного языка для преобразования речи в текст).

## <a name="initialize-an-intentrecognizer"></a>Инициализация объекта IntentRecognizer

Теперь создадим объект `IntentRecognizer`. Вставьте этот код непосредственно под конфигурацией службы "Речь".

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=15)]

## <a name="add-a-languageunderstandingmodel-and-intents"></a>Добавление модели LanguageUnderstandingModel и намерений

Необходимо сопоставить объект `LanguageUnderstandingModel` с распознавателем намерений и добавить намерения, которые необходимо распознать. Мы будем использовать намерения из предварительно созданной предметной области для системы домашней автоматики.

Вставьте приведенный ниже код под строкой вашего `IntentRecognizer`. Обязательно замените `"YourLanguageUnderstandingAppId"` идентификатором приложения LUIS. 

>[!TIP]
> Если вам нужна помощь с поиском этих значений, перейдите к разделу [Создание приложения LUIS для распознавания намерений](#create-a-luis-app-for-intent-recognition).

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=19-27)]

В этом примере функция `add_intents()` используется для добавления списка явно определенных намерений. Если требуется добавить все намерения из модели, используйте `add_all_intents(model)` и передайте модель.

> [!NOTE]
> Пакет SDK службы "Речь" поддерживает только конечные точки LUIS версии 2.0.
> Нужно вручную изменить URL-адрес конечной точки версии 3.0, который можно найти в поле примера запроса, на шаблон URL-адреса версии 2.0.
> Для конечных точек LUIS версии 2.0 всегда используется один из двух следующих шаблонов:
> * `https://{AzureResourceName}.cognitiveservices.azure.com/luis/v2.0/apps/{app-id}?subscription-key={subkey}&verbose=true&q=`
> * `https://{Region}.api.cognitive.microsoft.com/luis/v2.0/apps/{app-id}?subscription-key={subkey}&verbose=true&q=`

## <a name="recognize-an-intent"></a>Распознавание намерения

В объекте `IntentRecognizer` необходимо вызвать метод `recognize_once()`. С помощью этого метода служба "Речь" узнает, что для распознавания отправляется одна фраза и что после идентификации фразы необходимо остановить распознавание речи.

Вставьте приведенный ниже код под строкой модели.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=35)]

## <a name="display-the-recognition-results-or-errors"></a>Отображение результатов распознавания (или ошибок)

Когда служба "Речь" возвращает результат распознавания, необходимо с ним что-то сделать. Мы оставим его как есть и выведем результат в консоли.

Под вызовом `recognize_once()` добавьте указанный ниже код.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=38-47)]

## <a name="check-your-code"></a>Проверка кода

На этом этапе код должен выглядеть, как показано ниже.

> [!NOTE]
> Мы добавили несколько комментариев к этой версии.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=5-47)]

## <a name="build-and-run-your-app"></a>Создание и запуск приложения

Запустите пример из консоли или в интегрированной среде разработки:

```
python quickstart.py
```

Слова, произносимые в микрофон, в течение следующих 15 секунд будут распознаны и записаны в окне консоли.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [footer](./footer.md)]