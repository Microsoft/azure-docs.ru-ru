---
title: Краткое руководство. Интеллектуальный анализ текста с помощью клиентской библиотеки Анализа текста
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве описывается, как выполнить анализ тональности и другие действия с помощью API "Анализ текста" служб Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 04/19/2021
ms.author: aahi
keywords: Интеллектуальный анализ текста, анализ тональности, анализ текста
ms.custom: devx-track-python, devx-track-js, devx-track-csharp, cog-serv-seo-aug-2020
zone_pivot_groups: programming-languages-text-analytics
ms.openlocfilehash: 527936e1da385a3622358fe12efa0f87af4b3fd9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765100"
---
# <a name="quickstart-use-the-text-analytics-client-library-and-rest-api"></a>Краткое руководство. Использование клиентской библиотеки и REST API Анализа текста

Изучите эту статью, чтобы начать работу с клиентской библиотекой и REST API Анализа текста. Выполните следующие действия, чтобы опробовать пример кода для интеллектуального анализа текста:

* Анализ тональности
* Интеллектуальный анализ данных
* Определение языка
* Распознавание сущностей
* Распознавание персональных идентификационных сведений
* Извлечение ключевой фразы


::: zone pivot="programming-language-csharp"

> [!IMPORTANT]
> * `3.0` — это последняя стабильная версия API Анализа текста.
>    * Обязательно следуйте указаниям по используемой версии.
> * В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных. Для рабочих сценариев рекомендуется использовать пакетные асинхронные методы для повышения производительности и масштабируемости. См. справочную документацию ниже.
> * Если вы намерены использовать решение "Анализ текста для здравоохранения" или асинхронные операции, ознакомьтесь с примерами на [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics), [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/) или [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics) на сайте GitHub.

[!INCLUDE [C# quickstart](../includes/quickstarts/csharp-sdk.md)]

::: zone-end

::: zone pivot="programming-language-java"

> [!IMPORTANT]
> * `3.0` — это последняя стабильная версия API Анализа текста.
> * В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных. Для рабочих сценариев рекомендуется использовать пакетные асинхронные методы для повышения производительности и масштабируемости. См. справочную документацию ниже.
Если вы намерены использовать решение "Анализ текста для здравоохранения" или асинхронные операции, ознакомьтесь с примерами на [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics), [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/) или [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics) на сайте GitHub.

[!INCLUDE [Java quickstart](../includes/quickstarts/java-sdk.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

> [!IMPORTANT]
> * `3.0` — это последняя стабильная версия API Анализа текста.
>    * Обязательно следуйте указаниям по используемой версии.
> * В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных. Для рабочих сценариев рекомендуется использовать пакетные асинхронные методы для повышения производительности и масштабируемости. См. справочную документацию ниже.
> * Вы также можете запустить эту версию клиентской библиотеки службы "Анализ текста" [в браузере](https://github.com/Azure/azure-sdk-for-js/blob/master/documentation/Bundling.md).

[!INCLUDE [NodeJS quickstart](../includes/quickstarts/nodejs-sdk.md)]

::: zone-end

::: zone pivot="programming-language-python"

> [!IMPORTANT]
> * `3.0` — это последняя стабильная версия API Анализа текста.
>    * Обязательно следуйте указаниям по используемой версии.
> * В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных. Для рабочих сценариев рекомендуется использовать пакетные асинхронные методы для повышения производительности и масштабируемости. См. справочную документацию ниже. Если вы намерены использовать решение "Анализ текста для здравоохранения" или асинхронные операции, ознакомьтесь с примерами на [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics), [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/) или [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics) на сайте GitHub.

[!INCLUDE [Python quickstart](../includes/quickstarts/python-sdk.md)]

::: zone-end

::: zone pivot="rest-api"

> [!IMPORTANT]
> * `3.0` — это последняя стабильная версия API Анализа текста.
>    * Обязательно следуйте указаниям по используемой версии.

[!INCLUDE [REST API quickstart](../includes/quickstarts/rest-api.md)]

::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Изучение решения](../text-analytics-user-scenarios.md#analyze-recorded-inbound-customer-calls)

* [Text Analytics overview](../overview.md) (Общие сведения об анализе текста)
* [Пример. Как определить тональность с помощью Анализа текста](../how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Распознавание сущностей](../how-tos/text-analytics-how-to-entity-linking.md)
* [Пример. Как определить язык с помощью Анализа текста](../how-tos/text-analytics-how-to-keyword-extraction.md)
* [Распознавание языка](../how-tos/text-analytics-how-to-language-detection.md)
