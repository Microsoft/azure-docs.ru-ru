---
title: Общие сведения об Azure Cognitive Services
titleSuffix: Azure Cognitive Services
description: Cognitive Services делает ИИ доступным для всех разработчиков, и опыт работы с машинным обучением и анализом данных не требуется. Необходимо просто выполнить вызов API из приложения, чтобы добавить в свои приложения возможность для просмотра (расширенные поиск и распознавание изображений) и прослушивания данных, использования речи, поиска данных и принятия решений.
services: cognitive-services
author: nitinme
manager: nitinme
keywords: cognitive services, когнитивня аналитика, когнитивные решения, службы ИИ, когнитивное распознавание, когнитивные функции
ms.service: cognitive-services
ms.subservice: ''
ms.topic: overview
ms.date: 10/22/2020
ms.author: nitinme
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: c89131cc34d45ea94f3bb290ac11ec86f4b83be3
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587618"
---
# <a name="what-are-azure-cognitive-services"></a>Общие сведения об Azure Cognitive Services

Azure Cognitive Services — это облачные службы с REST API и пакетами SDK клиентской библиотеки, которые помогают интегрировать когнитивные средства искусственного интеллекта в свои приложения. Вы можете добавлять когнитивные функции в свои приложения, не обладая опытом работы с искусственным интеллектом (ИИ) или навыками обработки и анализа данных. Azure Cognitive Services содержит различные службы искусственного интеллекта, которые позволяют создавать когнитивные решения, использующие функции просмотра и прослушивания данных, функцию речи, анализа данных и даже принятия решений.

## <a name="categories-of-cognitive-services"></a>Категории Cognitive Services

Каталог служб Cognitive Services, которые обеспечивают когнитивное распознавание, состоит из пяти основных категорий:

* Зрение
* Речь
* Язык
* Решение
* Поиск

В следующих разделах этой статьи представлен список служб, которые являются частью этих пяти основных категорий.

## <a name="vision-apis"></a>API, связанные со зрением

|Имя службы|Описание службы|
|:-----------|:------------------|
|[Компьютерное зрение](./computer-vision/index.yml "API Компьютерного зрения")|Служба Компьютерного зрения предоставляет доступ к передовым когнитивным алгоритмам обработки изображений и возврата данных. Чтобы приступить к работе со службой, см. [Краткое руководство по службе "Компьютерное зрение"](./computer-vision/quickstarts-sdk/client-library.md).|
|[Пользовательское визуальное распознавание](./custom-vision-service/index.yml "Служба "Пользовательское визуальное распознавание"")|Служба "Пользовательское визуальное распознавание" позволяет создавать, развертывать и улучшать пользовательские классификаторы изображений. Классификатор изображений — это служба ИИ, которая присваивает изображениям метки на основе их визуальных характеристик. |
|[Распознавание лиц](./face/index.yml "Распознавание лиц")| Служба "Распознавание лиц" обеспечивает доступ к расширенным алгоритмам, позволяя определять и распознавать лица на основе атрибутов. Чтобы приступить к работе со службой, ознакомьтесь с [кратким руководством по службе "Распознавание лиц"](./face/quickstarts/client-libraries.md).|
|[Распознаватель документов](./form-recognizer/index.yml "Распознаватель документов")|Распознаватель документов идентифицирует и извлекает пары "ключ —значение" и данные таблиц из документов, а затем выводит структурированные данные, включая отношения в исходном файле. Чтобы приступить к работе, см. [Краткое руководство по службе "Распознаватель документов"](./form-recognizer/quickstarts/client-library.md).|
|[Индексатор видео](../media-services/video-indexer/video-indexer-overview.md "Индексатор видео")|Индексатор видео позволяет извлекать аналитические сведения из видео. Чтобы приступить к работе, см. [Краткое руководство по службе "Индексатор видео"](/azure/media-services/video-indexer/video-indexer-get-started).|

## <a name="speech-apis"></a>API, связанные с речью

|Имя службы|Описание службы|
|:-----------|:------------------|
|[Служба распознавания речи](./speech-service/index.yml "Служба Речь")|Служба "Речь" позволяет добавлять в приложения функции с поддержкой речи. Служба речи включает в себя различные возможности, такие как преобразование речи в текст, преобразование текста в речь и многое другое.|
<!--
|[Speaker Recognition API](./speech-service/speaker-recognition-overview.md "Speaker Recognition API") (Preview)|The Speaker Recognition API provides algorithms for speaker identification and verification.|
|[Bing Speech](./speech-service/how-to-migrate-from-bing-speech.md "Bing Speech") (Retiring)|The Bing Speech API provides you with an easy way to create speech-enabled features in your applications.|
|[Translator Speech](/azure/cognitive-services/translator-speech/ "Translator Speech") (Retiring)|Translator Speech is a machine translation service.|
-->
## <a name="language-apis"></a>API, связанные с языком

|Имя службы|Описание службы|
|:-----------|:------------------|
|[Распознавание речи (LUIS)](./luis/index.yml "Распознавание речи")|Распознавание речи (LUIS) — это облачная служба ИИ, которая применяет пользовательскую аналитику машинного обучения к тексту пользователя в разговорном стиле и на естественном языке, чтобы предсказать общий смысл и извлечь соответствующую подробную информацию. Чтобы приступить к работе со службой, см. [Краткое руководство по службе "Распознавание речи"](./luis/get-started-portal-build-app.md).|
|[QnA Maker](./qnamaker/index.yml "QnA Maker")|Служба QnA Maker позволяет создавать службу для работы с вопросами и ответами на основе частично структурированного содержимого. Чтобы приступить к работе со службой, см. [Краткое руководство по службе QnA Maker](./qnamaker/quickstarts/create-publish-knowledge-base.md).|
|[Анализ текста](./text-analytics/index.yml "Анализ текста")| Служба "Анализ текста" — это служба обработки естественного языка, выполняющая анализ тональности необработанного текста, извлечение ключевых фраз и определение языка. Чтобы приступить к работе со службой, см. [Краткое руководство по службе "Анализ текста"](./text-analytics/quickstarts/client-libraries-rest-api.md).|
|[Переводчик](./translator/index.yml "API перевода")|Переводчик выполняет машинный перевод текстов почти в реальном времени.|
| [Иммерсивное средство чтения](./immersive-reader/index.yml "Иммерсивное средство чтения") | Иммерсивное средство чтения добавляет в приложения возможности чтения и анализа данных экрана. Чтобы приступить к работе со службой, см. [краткое руководство по службе "Иммерсивное средство чтения"](./immersive-reader/quickstarts/client-libraries.md). |

## <a name="decision-apis"></a>API-интерфейсы принятия решений

|Имя службы|Описание службы|
|:-----------|:------------------|
|[Детектор аномалий](./anomaly-detector/index.yml "Детектор аномалий") |Детектор аномалий позволяет отслеживать и обнаруживать отклонения в данных временных рядов. Чтобы приступить к работе со службой, см. [Краткое руководство по службе "Детектор аномалий"](./anomaly-detector/quickstarts/client-libraries.md).|
|[Content Moderator](./content-moderator/overview.md "Content Moderator")|Content Moderator позволяет отслеживать потенциально оскорбительное, нежелательное и представляющее риск содержимое. Чтобы приступить к работе со службой, см. [краткое руководство по службе Content Moderator](./content-moderator/client-libraries.md).|
|[Помощник по метрикам](./metrics-advisor/index.yml) (предварительная версия) | Помощник по метрикам обеспечивает настраиваемое обнаружение аномалий для многовариантных данных временных рядов с использованием полнофункционального веб-портала. Чтобы приступить к работе со службой, см. [Краткое руководство по службе "Помощник по метрикам"](./metrics-advisor/quickstarts/rest-api-and-client-library.md). |
|[Персонализатор](./personalizer/index.yml "Персонализатор")|Персонализатор позволяет выбирать максимально удобный режим работы для своих пользователей, изучая их поведение в реальном времени. Чтобы приступить к работе со службой, см. [Краткое руководство по службе "Персонализатор"](./personalizer/quickstart-personalizer-sdk.md).|

## <a name="search-apis"></a>Интерфейсы API для поиска

> [!NOTE]
> Хотите узнать о [Когнитивном поиске Azure](../search/index.yml)? Хотя для некоторых задач используется Cognitive Services, это другая технология поиска, поддерживающая иные сценарии.

|Имя службы|Описание службы|
|:-----------|:------------------|
|[Поиск новостей Bing](/azure/cognitive-services/bing-news-search/ "API Поиска новостей Bing")|Служба "Поиск новостей Bing" выполняет поиск новостных статей релевантных запросу пользователя.|
|[Поиск видео Bing](/azure/cognitive-services/Bing-Video-Search/ "API Поиска видео Bing")|Служба "Поиск видео Bing" выполняет поиск видео, релевантных запросу пользователя.|
|[Поиск в Интернете Bing](./bing-web-search/index.yml "API Поиска в Интернете Bing")|Служба "Поиск в Интернете Bing" выполняет поиск материалов в Интернете, релевантных запросу пользователя.|
|[Автозаполнение Bing](/azure/cognitive-services/Bing-Autosuggest "API Автозаполнения Bing")|Служба "Автозаполнение Bing" предлагает отправлять в Bing часть слова поискового запроса и получать список предлагаемых запросов.|
|[Пользовательский поиск Bing](/azure/cognitive-services/bing-custom-search "Пользовательский поиск Bing")|Служба "Пользовательский поиск Bing" позволяет выполнять пользовательскую настройку функции поиска требуемых материалов.|
|[Поиск сущностей Bing](/azure/cognitive-services/bing-entities-search/ "API Поиска сущностей Bing")|Служба "Поиск сущностей Bing" возвращает сведения о сущностях, которые Bing определяет как релевантные запросу пользователя.|
|[Поиск изображений Bing](/azure/cognitive-services/bing-image-search "API Поиска изображений Bing")|Служба "Поиск изображений Bing" отображает изображения, релевантные запросу пользователя.|
|[Визуальный поиск Bing](/azure/cognitive-services/bing-visual-search "Визуальный поиск Bing")|Служба "Визуальный поиск Bing" возвращает аналитические сведения об изображении, например визуально схожие изображения, ресурсы, на которых можно купить продукты, найденные на изображении, а также связанные результаты поиска.|
|[Поиск местных компаний Bing](/azure/cognitive-services/bing-local-business-search/ "Поиск местных компаний Bing")| С помощью API Поиска местных компаний Bing для приложений можно находить контактную информацию о местных компаниях, а также сведения об их расположении на основе поисковых запросов.|
|[Проверка орфографии Bing](/azure/cognitive-services/bing-spell-check/ "API Проверки орфографии Bing")|Служба "Проверка орфографии Bing" предназначена для контекстной проверки грамматики и орфографии.|

## <a name="get-started-with-cognitive-services"></a>Начало работы с Cognitive Services

Начните с создания ресурса Cognitive Services с помощью практических руководств, используя указанные ниже методы.

* [Портал Azure](cognitive-services-apis-create-account.md?tabs=multiservice%2Cwindows "Портал Azure")
* [Azure CLI](cognitive-services-apis-create-account-cli.md?tabs=windows "Azure CLI")
* [Клиентские библиотеки пакетов Azure SDK](cognitive-services-apis-create-account-cli.md?tabs=windows "cognitive-services-apis-create-account-client-library?pivots=programming-language-csharp")
* [Шаблоны Azure Resource Manager (ARM)](./create-account-resource-manager-template.md?tabs=portal "Шаблоны Azure Resource Manager (ARM)")

## <a name="using-cognitive-services-in-different-development-environments"></a>Использование Cognitive Services в разных окружениях разработки

В Azure и Cognitive Services вы можете использовать несколько вариантов разработки:

* средства автоматизации и интеграции, такие как Logic Apps и Power Automate;
* варианты развертывания, такие как "Функции Azure" и "Служба приложений"; 
* контейнеры Docker Cognitive Services для безопасного доступа;
* такие средства, как Apache Spark, Azure Databricks, Azure Synapse Analytics и "Служба Azure Kubernetes" для сценариев с большими данными. 

Дополнительные сведения см. в статье о [вариантах разработки для Cognitive Services](./cognitive-services-development-options.md).

<!--
## Subscription management

Once you are signed in with your Microsoft Account, you can access [My subscriptions](https://www.microsoft.com/cognitive-services/subscriptions "My subscriptions") to show the products you are using, the quota remaining, and the ability to add additional products to your subscription.

## Upgrade to unlock higher limits

All APIs have a free tier, which has usage and throughput limits.  You can increase these limits by using a paid offering and selecting the appropriate pricing tier option when deploying the service in the Azure portal. [Learn more about the offerings and pricing](https://azure.microsoft.com/pricing/details/cognitive-services/ "offerings and pricing"). You'll need to set up an Azure subscriber account with a credit card and a phone number. If you have a special requirement or simply want to talk to sales, click "Contact us" button at the top the pricing page.
-->

## <a name="using-cognitive-services-securely"></a>Безопасное использование Cognitive Services

Службы Azure Cognitive Services предоставляют многоуровневую модель безопасности, включая [проверки подлинности](authentication.md "проверка подлинности") с помощью учетных данных Azure Active Directory, действительного ключа ресурса и [виртуальных сетей Azure](cognitive-services-virtual-networks.md "Виртуальные сети Azure").

## <a name="containers-for-cognitive-services"></a>Контейнеры для Cognitive Services

 Azure Cognitive Services предоставляет несколько контейнеров Docker, которые позволяют локально использовать те же API интерфейсы, которые доступны в Azure. Использование этих контейнеров предоставляет возможность гибко использовать Cognitive Services с вашими данными для обеспечения безопасности, соответствия требованиям и других эксплуатационных преимуществ. Дополнительные сведения о [контейнерах Cognitive Services](cognitive-services-container-support.md "Контейнеры Cognitive Services").

## <a name="regional-availability"></a>Доступность по регионам

API-интерфейсы в Cognitive Services размещаются в растущей сети центров обработки данных корпорации Майкрософт. Вы можете найти локализацию для каждого API в [списке регионов Azure](https://azure.microsoft.com/regions "Список регионов Azure").

Ищете регион, который пока не поддерживается? Сообщите нам, отправив запрос функции на наш [форум UserVoice](https://cognitive.uservoice.com/ "Форум UserVoice").

## <a name="supported-cultural-languages"></a>Поддерживаемые региональные языки

Cognitive Services поддерживает широкий спектр региональных языков на уровне службы. Дополнительные сведения о доступности языков для каждого API можно найти в [списке поддерживаемых языков](language-support.md "Список поддерживаемых языков").

## <a name="certifications-and-compliance"></a>Соответствие требованиям и сертификаты

Службы Cognitive Services получили такие сертификаты, как сертификация CSA STAR, FedRAMP Moderate и HIPAA BAA. Вы можете [скачать](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942 "скачиваемого файла") сертификаты для собственных аудитов и проверок безопасности.

Чтобы понять, как работает конфиденциальность и управление данными, перейдите в [центр управления безопасностью](https://servicetrust.microsoft.com/ "Центр управления безопасностью").

## <a name="support"></a>Поддержка

Cognitive Services предоставляет несколько вариантов поддержки, которые помогут вам эффективно создавать интеллектуальные приложения. Для пользователей Cognitive Services также обеспечивается поддержка от сообщества разработчиков, которые помогут ответить на конкретные вопросы. Полный список доступных вариантов см. в статье о [возможностях поддержки и справки для Cognitive Services](cognitive-services-support-options.md "Возможности получения поддержки и справки Cognitive Services").

## <a name="next-steps"></a>Дальнейшие действия

* [Создание учетной записи Cognitive Services](cognitive-services-apis-create-account.md "Создание учетной записи Cognitive Services")
* [Новые возможности в документации по Cognitive Services](whats-new-docs.md "Новые возможности в документации по Cognitive Services")
* [Планирование затрат и управление ими для Cognitive Services](plan-manage-costs.md)
