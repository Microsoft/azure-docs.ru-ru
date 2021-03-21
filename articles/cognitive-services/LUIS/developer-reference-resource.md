---
title: Ресурсы для разработчиков — Распознавание речи
description: Пакеты SDK, API-интерфейсы RESTFUL, CLI, помогают разрабатывать приложения Распознавание речи (LUIS) на языке программирования. Управляйте ресурсами Azure и прогнозами LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 01/12/2021
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 08a1965aadf8f50625e54d7c5e3b3f6a44c0bbd4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946181"
---
# <a name="sdk-rest-and-cli-developer-resources-for-language-understanding-luis"></a>Ресурсы для разработчиков SDK, RESTFUL и CLI для Распознавание речи (LUIS)

Пакеты SDK, API-интерфейсы RESTFUL, CLI, помогают разрабатывать приложения Распознавание речи (LUIS) на языке программирования. Управляйте ресурсами Azure и прогнозами LUIS.

## <a name="azure-resource-management"></a>Управление ресурсами Azure

Используйте уровень управления Cognitive Services Azure, чтобы создать, изменить, перечислить и удалить ресурс службы Распознавание речи или "перекрытие".

Найдите справочную документацию на основе средства:

* [Azure CLI](/cli/azure/cognitiveservices#az-cognitiveservices-list)

* [Azure RM PowerShell](/powershell/module/azurerm.cognitiveservices/#cognitive_services)


## <a name="language-understanding-authoring-and-prediction-requests"></a>Распознавание речи создания и прогнозирования запросов

Доступ к Распознавание речиной службе осуществляется из ресурса Azure, который необходимо создать. Существует два ресурса:

* Используйте ресурс **разработки** для обучения, чтобы создавать, изменять, обучать и публиковать их.
* Используйте **Прогноз** для среды выполнения для отправки текста пользователя и получения прогноза.

Дополнительные сведения о [конечной точке прогнозирования v3](luis-migration-api-v3.md).

Используйте [Cognitive Services пример кода](https://github.com/Azure-Samples/cognitive-services-quickstart-code) для изучения и использования наиболее распространенных задач.

### <a name="rest-specifications"></a>Спецификации RESTFUL

[Спецификации Luis RESTful](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/cognitiveservices/data-plane/LUIS), а также все [спецификации Azure для других](https://github.com/Azure/azure-rest-api-specs)пользователей доступны в GitHub.

### <a name="rest-apis"></a>Интерфейсы REST API

Интерфейсы API для конечных точек создания и прогнозирования доступны в API-интерфейсах:

|Type|Версия|
|--|--|
|Разработка|[Шаблон](https://go.microsoft.com/fwlink/?linkid=2092087)<br>[Предварительный просмотр v3](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview)|
|Прогнозирование|[Шаблон](https://go.microsoft.com/fwlink/?linkid=2092356)<br>[Том](https://westcentralus.dev.cognitive.microsoft.com/docs/services/luis-endpoint-api-v3-0/)|

### <a name="rest-endpoints"></a>Конечные точки RESTFUL

В настоящее время в LUIS есть 2 типа конечных точек:

* **Разработка** в конечной точке обучения
* **Прогнозирование** запросов в конечной точке среды выполнения.

|Назначение|URL-адрес|
|--|--|
|Создание версии 2 для конечной точки обучения|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/api/v2.0/apps/{appID}/`|
|3. Разработка для конечной точки обучения|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/authoring/v3.0-preview/apps/{appID}/`|
|Прогнозирование v2 — все прогнозы на конечной точке среды выполнения|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q={q}[&timezoneOffset][&verbose][&spellCheck][&staging][&bing-spell-check-subscription-key][&log]`|
|V3 прогноз — прогнозирование версий на конечной точке времени выполнения|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/{appId}/versions/{versionId}/predict?query={query}[&verbose][&log][&show-all-intents]`|
|3 Прогноз — прогнозирование слотов на конечной точке времени выполнения|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/{appId}/slots/{slotName}/predict?query={query}[&verbose][&log][&show-all-intents]`|

В следующей таблице описаны параметры, обозначенные фигурными скобками `{}` в предыдущей таблице.

|Параметр|Назначение|
|--|--|
|`your-resource-name`|Имя ресурса Azure|
|`q` или `query`|Текст фразы, отправляемый из клиентского приложения, такого как чат-бот|
|`version`|имя версии с 10 символами|
|`slot`| `production` или `staging`|

### <a name="rest-query-string-parameters"></a>Параметры строки запроса RESTFUL

[!INCLUDE [V3 query params](./includes/v3-prediction-query-params.md)]

## <a name="app-schema"></a>Схема приложения

[Схема приложения](app-schema-definition.md) импортируется и экспортируется в `.json` формате или `.lu` .

### <a name="language-based-sdks"></a>Пакеты SDK на основе языка

|Язык |Справочная документация|Пакет|Краткие руководства|
|--|--|--|--|
|C#|[Разработка](/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring)</br>[Прогнозирование](/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime)|[Создание NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/)<br>[Прогнозирование NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime/)|[Разработка](./client-libraries-rest-api.md?pivots=rest-api)<br>[Прогнозирование запросов](./client-libraries-rest-api.md?pivots=rest-api)|
|Go|[Создание и прогнозирование](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.0/luis)|[Пакет SDK](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.0/luis)||
|Java|[Создание и прогнозирование](/java/api/overview/azure/cognitiveservices/client/languageunderstanding)|[Создание Maven](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-luis-authoring)<br>[Прогноз Maven](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-luis-runtime)|
|JavaScript|[Разработка](/javascript/api/@azure/cognitiveservices-luis-authoring/)<br>[Прогнозирование](/javascript/api/@azure/cognitiveservices-luis-runtime/)|[Создание NPM](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring)<br>[Прогноз NPM](https://www.npmjs.com/package/@azure/cognitiveservices-luis-runtime)|[Разработка](./client-libraries-rest-api.md?pivots=rest-api)<br>[Прогнозирование](./client-libraries-rest-api.md?pivots=rest-api)|
|Python|[Создание и прогнозирование](./client-libraries-rest-api.md?pivots=rest-api)|[PIP](https://pypi.org/project/azure-cognitiveservices-language-luis/)|[Разработка](./client-libraries-rest-api.md?pivots=rest-api)<br>[Прогнозирование](./client-libraries-rest-api.md?pivots=rest-api)|


### <a name="containers"></a>Контейнеры

Распознавание речи (LUIS) предоставляет [контейнер](luis-container-howto.md) для предоставления локальных и автономных версий приложения.

### <a name="export-and-import-formats"></a>Форматы экспорта и импорта

Распознавание речи предоставляет возможность управлять приложением и его моделями в формате JSON, `.LU` формате ([лудовн](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown)) и сжатом пакете для контейнера распознавание речи.

Импорт и экспорт этих форматов можно получить из API-интерфейсов и с портала LUIS. Портал предоставляет импорт и экспорт как часть списка приложений и версий.

## <a name="workshops"></a>Семинары

* GitHub: (семинар) [общение — AI: НЛУ с помощью Luis](https://github.com/GlobalAICommunity/Workshop-Conversational-AI)

## <a name="continuous-integration-tools"></a>средства непрерывной интеграции;

* GitHub: (Предварительная версия) [Разработка приложения Luis с использованием методик DevOps](https://github.com/Azure-Samples/LUIS-DevOps-Template)
* GitHub: [НЛУ. DevOps](https://github.com/microsoft/NLU.DevOps) — средства, поддерживающие непрерывную интеграцию и развертывание для служб НЛУ.

## <a name="bot-framework-tools"></a>Инструменты платформы Bot

Платформа Bot доступна в виде [пакета SDK](https://github.com/Microsoft/botframework) на различных языках и в качестве службы с помощью [службы Azure Bot](https://dev.botframework.com/).

Платформа Bot предоставляет [несколько средств](https://github.com/microsoft/botbuilder-tools) , помогающих распознавание речи, в том числе:
* [Эмулятор Bot Framework](https://github.com/Microsoft/BotFramework-Emulator/releases) — классическое приложение, позволяющее Bot-разработчикам тестировать и отлаживать программы-роботы, созданные с помощью пакета SDK для Bot Framework.
* [Композитор Framework Composer](https://github.com/microsoft/BotFramework-Composer/blob/stable/README.md) — интегрированное средство разработки для разработчиков и групп могут, предназначенное для создания программы-роботы и общения с помощью платформы Microsoft Bot Framework
* [Примеры для платформы Bot](https://github.com/microsoft/botbuilder-samples) в #C, JavaScript, TypeScript и Python

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об общих [кодах ошибок HTTP](luis-reference-response-codes.md)
* [Справочная документация](../../index.yml) по всем API и пакетам SDK
* [Платформа Bot](https://github.com/Microsoft/botbuilder-dotnet) и [служба Azure Bot](https://dev.botframework.com/)
* [LUDown](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown)
* [Восприятные контейнеры](../cognitive-services-container-support.md)
