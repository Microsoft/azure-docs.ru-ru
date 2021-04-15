---
title: Краткое руководство по использованию клиентских библиотек Поиска в Интернете Bing (Python)
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/05/2020
ms.author: aahi
ms.openlocfilehash: 5d5aaf84482dae6786ac7fd9f9ee837efca71b34
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105377"
---
Клиентская библиотека Поиска в Интернете Bing позволяет интегрировать поиск Bing в любое приложение Python. В этом кратком руководстве описано, как отправлять запрос, получать ответ в формате JSON, фильтровать и анализировать результаты.

Хотите увидеть код прямо сейчас? Примеры для [клиентских библиотек Поиска Bing для Python](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/tree/master/samples/search) доступны на сайте GitHub.


## <a name="prerequisites"></a>Предварительные требования
Пакет SDK для поиска в Интернете Bing совместим с Python версии 2.7 или 3.6 и выше. Мы рекомендуем использовать для этого руководства виртуальное окружение.

* Python версии 2.7 или 3.6 и выше
* [virtualenv](https://docs.python.org/3/tutorial/venv.html) для Python 2.7
* [venv](https://pypi.python.org/pypi/virtualenv) для Python 3.x

[!INCLUDE [bing-web-search-quickstart-signup](~/includes/bing-web-search-quickstart-signup.md)]

## <a name="create-and-configure-your-virtual-environment"></a>Создание и настройка виртуального окружения

Инструкции по установке и настройке виртуального окружения будут разными для версий Python 2.x и Python 3.x. Выполните описанные ниже шаги, чтобы создать и инициализировать виртуальное окружение.

### <a name="python-2x"></a>Python 2.x

Создайте виртуальное окружение с `virtualenv` для Python 2.7:

```console
virtualenv mytestenv
```

Активируйте созданное окружение:

```console
cd mytestenv
source bin/activate
```

Установите зависимости пакета SDK Bing для поиска в Интернете:

```console
python -m pip install azure-cognitiveservices-search-websearch
```

### <a name="python-3x"></a>Python 3.x

Создайте виртуальное окружение с `venv` для Python 3.x:

```console
python -m venv mytestenv
```

Активируйте созданное окружение:

```console
mytestenv\Scripts\activate.bat
```

Установите зависимости пакета SDK Bing для поиска в Интернете:

```console
cd mytestenv
python -m pip install azure-cognitiveservices-search-websearch
```

## <a name="create-a-client-and-print-your-first-results"></a>Создание клиента и вывод первых результатов

Итак, вы настроили виртуальное окружение и установили зависимости. Теперь давайте создадим клиент. Этот клиент будет обрабатывать запросы и ответы для API Поиска в Интернете Bing.

Если ответ содержит веб-страницы, изображения, новости или видео, будет выведен первый результат в каждой категории.

1. Создайте проект Python, используя любую IDE или любой текстовый редактор.

1. Скопируйте в проект приведенный ниже пример кода. В качестве `endpoint` может быть глобальная конечная точка, приведенная ниже, или конечная точка [пользовательского поддомена](~/articles/cognitive-services/cognitive-services-custom-subdomains.md), отображаемая на портале Azure для вашего ресурса.  

    ```python
    # Import required modules.
    from azure.cognitiveservices.search.websearch import WebSearchClient
    from azure.cognitiveservices.search.websearch.models import SafeSearch
    from msrest.authentication import CognitiveServicesCredentials

    # Replace with your subscription key.
    subscription_key = "YOUR_SUBSCRIPTION_KEY"

    # Instantiate the client and replace with your endpoint.
    client = WebSearchClient(endpoint="YOUR_ENDPOINT", credentials=CognitiveServicesCredentials(subscription_key))

    # Make a request. Replace Yosemite if you'd like.
    web_data = client.web.search(query="Yosemite")
    print("\r\nSearched for Query# \" Yosemite \"")

    '''
    Web pages
    If the search response contains web pages, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.web_pages, 'value'):

        print("\r\nWebpage Results#{}".format(len(web_data.web_pages.value)))

        first_web_page = web_data.web_pages.value[0]
        print("First web page name: {} ".format(first_web_page.name))
        print("First web page URL: {} ".format(first_web_page.url))

    else:
        print("Didn't find any web pages...")

    '''
    Images
    If the search response contains images, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.images, 'value'):

        print("\r\nImage Results#{}".format(len(web_data.images.value)))

        first_image = web_data.images.value[0]
        print("First Image name: {} ".format(first_image.name))
        print("First Image URL: {} ".format(first_image.url))

    else:
        print("Didn't find any images...")

    '''
    News
    If the search response contains news, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.news, 'value'):

        print("\r\nNews Results#{}".format(len(web_data.news.value)))

        first_news = web_data.news.value[0]
        print("First News name: {} ".format(first_news.name))
        print("First News URL: {} ".format(first_news.url))

    else:
        print("Didn't find any news...")

    '''
    If the search response contains videos, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.videos, 'value'):

        print("\r\nVideos Results#{}".format(len(web_data.videos.value)))

        first_video = web_data.videos.value[0]
        print("First Videos name: {} ".format(first_video.name))
        print("First Videos URL: {} ".format(first_video.url))

    else:
        print("Didn't find any videos...")
    ```

1. Замените значение `SUBSCRIPTION_KEY` действительным ключом подписки.

1. Замените `YOUR_ENDPOINT` URL-адресом конечной точки на портале и удалите фрагмент bing/v7.0 из конечной точки.

1. Запустите программу. Например: `python your_program.py`.

## <a name="define-functions-and-filter-results"></a>Определение функций и фильтрация результатов

Теперь, когда вы выполнили первый вызов к API поиска в Интернете Bing, давайте изучим несколько функций. В следующем разделе демонстрируются возможности пакета SDK по уточнению запросов и фильтрации результатов. Каждую из этих функций можно добавить в программу Python, созданную в предыдущем разделе.

### <a name="limit-the-number-of-results-returned-by-bing"></a>Ограничение числа результатов, возвращаемых Bing

В этом примере используются параметры `count` и `offset`, которые позволяют ограничить число результатов, возвращаемых [методом `search`](/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations) пакета SDK. Для первого результата возвращаются `name` и `url`.

1. Добавьте следующий код в проект Python:

   ```python
    # Declare the function.
    def web_results_with_count_and_offset(subscription_key):
        client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))

        try:
            '''
            Set the query, offset, and count using the SDK's search method. See:
            https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python.
            '''
            web_data = client.web.search(query="Best restaurants in Seattle", offset=10, count=20)
            print("\r\nSearching for \"Best restaurants in Seattle\"")

            if web_data.web_pages.value:
                '''
                If web pages are available, print the # of responses, and the first and second
                web pages returned.
                '''
                print("Webpage Results#{}".format(len(web_data.web_pages.value)))

                first_web_page = web_data.web_pages.value[0]
                print("First web page name: {} ".format(first_web_page.name))
                print("First web page URL: {} ".format(first_web_page.url))

            else:
                print("Didn't find any web pages...")

        except Exception as err:
            print("Encountered exception. {}".format(err))
    ```

1. Запустите программу.

### <a name="filter-for-news-and-freshness"></a>Фильтрация по новостям и актуальности

В этом примере используются параметры `response_filter` и `freshness` для фильтрации результатов поиска в [методе `search`](/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations) пакета SDK. Будут возвращаться только те результаты поиска, которые соответствуют новостям и страницам, обнаруженным Bing за последние 24 часа. Для первого результата возвращаются `name` и `url`.

1. Добавьте следующий код в проект Python:

    ```python
    # Declare the function.
    def web_search_with_response_filter(subscription_key):
        client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))
        try:
            '''
            Set the query, response_filter, and freshness using the SDK's search method. See:
            https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python.
            '''
            web_data = client.web.search(query="xbox",
                response_filter=["News"],
                freshness="Day")
            print("\r\nSearching for \"xbox\" with the response filter set to \"News\" and freshness filter set to \"Day\".")

            '''
            If news articles are available, print the # of responses, and the first and second
            articles returned.
            '''
            if web_data.news.value:

                print("# of news results: {}".format(len(web_data.news.value)))

                first_web_page = web_data.news.value[0]
                print("First article name: {} ".format(first_web_page.name))
                print("First article URL: {} ".format(first_web_page.url))

                print("")

                second_web_page = web_data.news.value[1]
                print("\nSecond article name: {} ".format(second_web_page.name))
                print("Second article URL: {} ".format(second_web_page.url))

            else:
                print("Didn't find any news articles...")

        except Exception as err:
            print("Encountered exception. {}".format(err))

    # Call the function.
    web_search_with_response_filter(subscription_key)
    ```

1. Запустите программу.

### <a name="use-safe-search-answer-count-and-the-promote-filter"></a>Использование безопасного поиска, счетчика ответов и фильтра повышения уровня

В этом примере используются параметры `answer_count`, `promote` и `safe_search` для фильтрации результатов поиска в [методе `search`](/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations) пакета SDK. Для первого результата возвращаются `name` и `url`.

1. Добавьте следующий код в проект Python:

    ```python
    # Declare the function.
    def web_search_with_answer_count_promote_and_safe_search(subscription_key):

        client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))

        try:
            '''
            Set the query, answer_count, promote, and safe_search parameters using the SDK's search method. See:
            https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python.
            '''
            web_data = client.web.search(
                query="Niagara Falls",
                answer_count=2,
                promote=["videos"],
                safe_search=SafeSearch.strict  # or directly "Strict"
            )
            print("\r\nSearching for \"Niagara Falls\"")

            '''
            If results are available, print the # of responses, and the first result returned.
            '''
            if web_data.web_pages.value:

                print("Webpage Results#{}".format(len(web_data.web_pages.value)))

                first_web_page = web_data.web_pages.value[0]
                print("First web page name: {} ".format(first_web_page.name))
                print("First web page URL: {} ".format(first_web_page.url))

            else:
                print("Didn't see any Web data..")

        except Exception as err:
            print("Encountered exception. {}".format(err))
    ```

1. Запустите программу.

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ключ подписки из кода программы и отключите виртуальное окружение, когда завершите работу с этим проектом.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Примеры пакета SDK для Python в Cognitive Services](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)

## <a name="see-also"></a>См. также раздел

* [Справочник по пакету SDK Azure для Python](/python/api/azure-cognitiveservices-search-websearch/)
