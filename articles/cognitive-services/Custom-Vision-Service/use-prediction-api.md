---
title: использование конечной точки прогноза для программного тестирования изображений с помощью классификатора — Пользовательская служба визуального распознавания
titleSuffix: Azure Cognitive Services
description: Сведения о том, как использовать API для программного тестирования изображений с помощью классификатора Пользовательской службы визуального распознавания.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/02/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 7f1939536e033d2cf964dd2f4ee562e4ee20061b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88934758"
---
# <a name="use-your-model-with-the-prediction-api"></a>Использование модели с API-интерфейсом прогнозирования

После обучения модели можно протестировать изображения программно, отправив их в конечную точку API прогнозирования.

> [!NOTE]
> В этом документе описано, как использовать C# для отправки изображения в API прогнозирования. Дополнительные сведения и примеры см. в [справочнике по API прогнозирования](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c15).

## <a name="publish-your-trained-iteration"></a>Опубликуйте обученную итерацию

На [веб-странице Пользовательской службы визуального распознавания](https://customvision.ai) выберите проект и перейдите на вкладку __Производительность__.

Чтобы отправить изображения в API прогнозирования, сначала необходимо опубликовать итерацию для прогнозирования, что можно сделать, выбрав __опубликовать__ и указав имя опубликованной итерации. Это сделает вашу модель доступной для API-интерфейса прогнозирования Пользовательское визуальное распознавание ресурса Azure.

![На вкладке "производительность" отображается красный прямоугольник, окружающий кнопку "опубликовать".](./media/use-prediction-api/unpublished-iteration.png)

После успешной публикации модели на боковой панели слева появится метка "опубликованная", и ее имя появится в описании итерации.

![Вкладка производительность отображается с красным прямоугольником, окружающим опубликованную метку, и именем опубликованной итерации.](./media/use-prediction-api/published-iteration.png)

## <a name="get-the-url-and-prediction-key"></a>Получение URL-адреса и ключа прогнозирования

После публикации модели можно получить необходимые сведения, выбрав __прогнозирующий URL-адрес__. Откроется диалоговое окно со сведениями об использовании API прогнозирования, включая __прогнозирующий URL-адрес__ и __ключ прогноза__.

![На вкладке "производительность" отображается красный прямоугольник, окружающий кнопку "прогнозирующий URL-адрес".](./media/use-prediction-api/published-iteration-prediction-url.png)

![На вкладке "производительность" отображается красный прямоугольник, окружающий значение прогнозирующего URL-адреса для использования файла изображения и значения Prediction-Key.](./media/use-prediction-api/prediction-api-info.png)


В этом разделе вы будете использовать локальный образ, поэтому скопируйте URL-адрес в разделе **при наличии файла изображения** во временном расположении. Скопируйте также и соответствующее значение __ключа прогноза__ .

## <a name="create-the-application"></a>Создание приложения

1. В Visual Studio создайте новое консольное приложение C#.

1. Используйте следующий код в файле __Program.cs__:

    ```csharp
    using System;
    using System.IO;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Threading.Tasks;

    namespace CVSPredictionSample
    {
        public static class Program
        {
            public static void Main()
            {
                Console.Write("Enter image file path: ");
                string imageFilePath = Console.ReadLine();

                MakePredictionRequest(imageFilePath).Wait();

                Console.WriteLine("\n\nHit ENTER to exit...");
                Console.ReadLine();
            }

            public static async Task MakePredictionRequest(string imageFilePath)
            {
                var client = new HttpClient();

                // Request headers - replace this example key with your valid Prediction-Key.
                client.DefaultRequestHeaders.Add("Prediction-Key", "<Your prediction key>");

                // Prediction URL - replace this example URL with your valid Prediction URL.
                string url = "<Your prediction URL>";

                HttpResponseMessage response;

                // Request body. Try this sample with a locally stored image.
                byte[] byteData = GetImageAsByteArray(imageFilePath);

                using (var content = new ByteArrayContent(byteData))
                {
                    content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                    response = await client.PostAsync(url, content);
                    Console.WriteLine(await response.Content.ReadAsStringAsync());
                }
            }

            private static byte[] GetImageAsByteArray(string imageFilePath)
            {
                FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
                BinaryReader binaryReader = new BinaryReader(fileStream);
                return binaryReader.ReadBytes((int)fileStream.Length);
            }
        }
    }
    ```

1. Измените следующие сведения:
   * Присвойте `namespace` полю имя проекта.
   * Замените заполнитель `<Your prediction key>` значением ключа, полученным ранее.
   * Замените заполнитель `<Your prediction URL>` URL-адресом, полученным ранее.

## <a name="run-the-application"></a>Выполнение приложения

При запуске приложения вам будет предложено ввести путь к файлу изображения в консоли. Затем изображение отправляется в API-интерфейс прогнозирования, и результаты прогноза возвращаются в виде строки в формате JSON. Ниже приведен пример ответа.

```json
{
    "id":"7796df8e-acbc-45fc-90b4-1b0c81b73639",
    "project":"8622c779-471c-4b6e-842c-67a11deffd7b",
    "iteration":"59ec199d-f3fb-443a-b708-4bca79e1b7f7",
    "created":"2019-03-20T16:47:31.322Z",
    "predictions":[
        {"tagId":"d9cb3fa5-1ff3-4e98-8d47-2ef42d7fb373","tagName":"cat", "probability":1.0},
        {"tagId":"9a8d63fb-b6ed-4462-bcff-77ff72084d99","tagName":"dog", "probability":0.1087869}
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как отправлять изображения в классификатор или детектор пользовательского образа, а также как получить ответ программным способом с помощью пакета SDK для C#. Далее вы узнаете, как выполнять комплексные сценарии в C# или приступить к работе с другим языковым пакетом SDK.

* [Краткое руководство. Пользовательское визуальное распознавание SDK](quickstarts/image-classification.md)
