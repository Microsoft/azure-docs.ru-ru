---
title: Краткое руководство по использованию клиентской библиотеки Поиска изображений Bing (Python)
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/04/2020
ms.author: aahi
ms.openlocfilehash: 338e4f0c1c47a7786e21ad98bb3e56a1505f9bb2
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105292"
---
Воспользуйтесь сведениями из краткого руководства и выполните первый поиск изображения с помощью клиентской библиотеки Поиска изображений Bing. Эта библиотека является оболочкой для API и содержит те же функции. Это простое приложение Python отправляет запрос на поиск изображения, анализирует ответ JSON и отображает URL-адрес первого возвращенного изображения.

Исходный код, используемый в данном примере, вместе с дополнительной обработкой ошибок и аннотациями можно получить на [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/image-search-quickstart.py).

## <a name="prerequisites"></a>Предварительные требования

* [Python версии 2.7 или 3.6 и выше.](https://www.python.org/)

* [Клиентская библиотека Поиска изображений Azure](https://pypi.org/project/azure-cognitiveservices-search-imagesearch/) для Python.
    * Установка с помощью `pip install azure-cognitiveservices-search-imagesearch`

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](~/includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Создание и инициализация приложения

1. Создайте сценарий Python в избранной интегрированной среде разработки или редакторе и добавьте следующие строки импорта:

    ```python
    from azure.cognitiveservices.search.imagesearch import ImageSearchClient
    from msrest.authentication import CognitiveServicesCredentials
    ```

2. Создайте переменные для ключа подписки и условия поиска.

    ```python
    subscription_key = "Enter your key here"
    subscription_endpoint = "Enter your endpoint here"
    search_term = "canadian rockies"
    ```

## <a name="create-the-image-search-client"></a>Создание клиента для поиска изображений

1. Создайте экземпляр `CognitiveServicesCredentials` и используйте его для создания клиента:

    ```python
    client = ImageSearchClient(endpoint=subscription_endpoint, credentials=CognitiveServicesCredentials(subscription_key))
    ```
1. Отправьте поисковой запрос к API Bing для поиска изображений:
    ```python
    image_results = client.images.search(query=search_term)
    ```
   ## <a name="process-and-view-the-results"></a>Обработка и просмотр результатов

Выполните синтаксический анализ результатов изображений, возвращенных в ответе.


Если в ответе содержатся результаты поиска, сохраните первый результат и распечатайте такие сведения, как URL-адрес эскиза, исходный URL-адрес и общее количество возвращенных изображений.  

```python
if image_results.value:
    first_image_result = image_results.value[0]
    print("Total number of images returned: {}".format(len(image_results.value)))
    print("First image thumbnail url: {}".format(
        first_image_result.thumbnail_url))
    print("First image content url: {}".format(first_image_result.content_url))
else:
    print("No image results returned!")
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство по одностраничным приложениям для API Bing для поиска изображений](../../tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>См. также раздел

* [Что такое API Bing для поиска изображений?](../../overview.md)  
* [Попробуйте API Bing для поиска изображений](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [Примеры Python для пакета SDK в Azure Cognitive Services](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)  
* [Документация по службам Azure Cognitive Services](../../../index.yml)
* [Справочник по API Bing для поиска изображений](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)