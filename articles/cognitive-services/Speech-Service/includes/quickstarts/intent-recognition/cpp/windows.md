---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.date: 04/04/2020
ms.topic: include
ms.author: trbye
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: b4c6591911137441bec7411dc697ed63212fbbc2
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102445787"
---
## <a name="prerequisites"></a>Предварительные требования

Необходимые условия:

* <a href="~/articles/cognitive-services/Speech-Service/quickstarts/setup-platform.md?tabs=windows&pivots=programming-language-cpp" target="_blank">Установите пакет SDK службы "Речь" для среды разработки и создайте пустой пример проекта</a>.

## <a name="create-a-luis-app-for-intent-recognition"></a>Создание приложения LUIS для распознавания намерений

[!INCLUDE [Create a LUIS app for intent recognition](../luis-sign-up.md)]

## <a name="open-your-project-in-visual-studio"></a>Откройте проект в Visual Studio.

Затем откройте проект в Visual Studio.

1. Запустите Visual Studio 2019.
2. Загрузите проект и откройте `helloworld.cpp`.

## <a name="start-with-some-boilerplate-code"></a>Добавление стандартного кода

Добавим код, который выступает в качестве основы для нашего проекта. Имейте в виду, что вы создали асинхронный метод `recognizeIntent()`.

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=6-16,72-80)]

## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Прежде чем инициализировать объект `IntentRecognizer`, необходимо создать конфигурацию, использующую ключ и расположение ресурса прогнозирования LUIS.

> [!IMPORTANT]
> Ключ для начала работы и ключ разработки не будут работать. Необходимо использовать ранее созданные ключ прогнозирования и расположение. Дополнительные сведения см. в разделе [Создание приложения LUIS для распознавания намерений](#create-a-luis-app-for-intent-recognition).

Вставьте код в метод `recognizeIntent()`. Обязательно обновите следующие значения:

* Замените `"YourLanguageUnderstandingSubscriptionKey"` ключом прогнозирования LUIS.
* Замените `"YourLanguageUnderstandingServiceRegion"` расположением LUIS.  Использование **идентификатор региона** из [региона](../../../../regions.md)

>[!TIP]
> Если вам нужна помощь с поиском этих значений, перейдите к разделу [Создание приложения LUIS для распознавания намерений](#create-a-luis-app-for-intent-recognition).

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=25)]

В этом примере для создания `SpeechConfig` используется метод `FromSubscription()`. Полный список доступных методов см. в статье [SpeechConfig Class](/cpp/cognitive-services/speech/speechconfig) (Класс SpeechConfig).

Пакет SDK для распознавания речи по умолчанию распознает использование языкового стандарта en-us. Дополнительные сведения о выборе исходного языка см. в разделе [Specify source language for speech to text](../../../../how-to-specify-source-language.md) (Указание исходного языка для преобразования речи в текст).

## <a name="initialize-an-intentrecognizer"></a>Инициализация объекта IntentRecognizer

Теперь создадим объект `IntentRecognizer`. Вставьте этот код в метод `recognizeIntent()` непосредственно под конфигурацией службы "Речь".

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=28)]

## <a name="add-a-languageunderstandingmodel-and-intents"></a>Добавление модели LanguageUnderstandingModel и намерений

Необходимо сопоставить объект `LanguageUnderstandingModel` с распознавателем намерений и добавить намерения, которые необходимо распознать. Мы будем использовать намерения из предварительно созданной предметной области для системы домашней автоматики.

Вставьте приведенный ниже код под строкой вашего `IntentRecognizer`. Обязательно замените `"YourLanguageUnderstandingAppId"` идентификатором приложения LUIS.

>[!TIP]
> Если вам нужна помощь с поиском этих значений, перейдите к разделу [Создание приложения LUIS для распознавания намерений](#create-a-luis-app-for-intent-recognition).

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=31-33)]

В этом примере функция `AddIntent()` используется для индивидуального добавления намерений. Если требуется добавить все намерения из модели, используйте `AddAllIntents(model)` и передайте модель.

> [!NOTE]
> Пакет SDK службы "Речь" поддерживает только конечные точки LUIS версии 2.0.
> Нужно вручную изменить URL-адрес конечной точки версии 3.0, который можно найти в поле примера запроса, на шаблон URL-адреса версии 2.0.
> Для конечных точек LUIS версии 2.0 всегда используется один из двух следующих шаблонов:
> * `https://{AzureResourceName}.cognitiveservices.azure.com/luis/v2.0/apps/{app-id}?subscription-key={subkey}&verbose=true&q=`
> * `https://{Region}.api.cognitive.microsoft.com/luis/v2.0/apps/{app-id}?subscription-key={subkey}&verbose=true&q=`

## <a name="recognize-an-intent"></a>Распознавание намерения

В объекте `IntentRecognizer` необходимо вызвать метод `RecognizeOnceAsync()`. С помощью этого метода служба "Речь" узнает, что для распознавания отправляется одна фраза и что после идентификации фразы необходимо остановить распознавание речи. Для удобства мы дождемся завершения возврата.

Вставьте приведенный ниже код под строкой модели:

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=43)]

## <a name="display-the-recognition-results-or-errors"></a>Отображение результатов распознавания (или ошибок)

Когда служба "Речь" возвращает результат распознавания, необходимо с ним что-то сделать. Мы оставим его как есть и выведем результат в консоли.

Вставьте код ниже под строкой `auto result = recognizer->RecognizeOnceAsync().get();`.

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=46-71)]

## <a name="check-your-code"></a>Проверка кода

На этом этапе код должен выглядеть так:

> [!NOTE]
> Мы добавили несколько комментариев к этой версии.

[!code-cpp[](~/samples-cognitive-services-speech-sdk/quickstart/cpp/windows/intent-recognition/helloworld/helloworld.cpp?range=6-79)]

## <a name="build-and-run-your-app"></a>Создание и запуск приложения

Теперь можно приступать к созданию приложения и проверке распознавания речи, используя службу "Речь".

1. **Скомпилируйте код**. В строке меню Visual Studio последовательно выберите **Сборка** > **Собрать решение**.
2. **Запустите приложение**. В строке меню выберите **Отладка** > **Начать отладку** или нажмите клавишу <kbd>F5</kbd>.
3. **Начните распознавание**. Вам будет предложено произнести фразу на английском языке. Речь, записанная в виде текста, отправляется в службу "Речь" и выводится в консоли.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [footer](./footer.md)]