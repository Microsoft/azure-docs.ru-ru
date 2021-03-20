---
title: Использование контейнеров Cognitive Services Azure в локальной среде
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать контейнеры DOCKER для локального использования Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.topic: article
ms.date: 12/16/2020
ms.author: aahi
keywords: локальная среда, Docker, контейнер, Kubernetes
ms.openlocfilehash: f5caac0a0da3595075dddd5a273a6883a0cca468
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97861835"
---
# <a name="azure-cognitive-services-containers"></a>Контейнеры Azure Cognitive Services

Azure Cognitive Services предоставляет несколько [контейнеров DOCKER](https://www.docker.com/what-container) , которые позволяют использовать те же интерфейсы API, которые доступны в Azure локально. Используя эти контейнеры, вы получаете возможность Cognitive Services ближе к вашим данным для обеспечения соответствия требованиям, безопасности или других эксплуатационных причин. В настоящее время поддержка контейнеров доступна для подмножества Cognitive Services Azure.

> [!VIDEO https://www.youtube.com/embed/hdfbn4Q8jbo]

Контейнеризация — это подход к распространению программного обеспечения, при котором приложение или служба упаковывается в образ контейнера вместе со всеми зависимостями и конфигурациями. Образ контейнера можно развертывать на узле контейнера с незначительной модификацией или без нее. Контейнеры изолированы друг от друга и от базовой операционной системы. При этом они расходуют меньше ресурсов, чем виртуальная машина. Контейнеры для краткосрочных задач можно создавать на основе образов контейнеров и удалять их, когда они станут ненужными.

## <a name="features-and-benefits"></a>Возможности и преимущества:

- **Неизменяемая инфраструктура**. Включите DevOps Teams, чтобы использовать согласованный и надежный набор известных системных параметров, в то время как можно адаптироваться к изменению. Контейнеры обеспечивают гибкость в прогнозировании в прогнозируемой экосистеме и позволяют избежать перенастройки.
- **Управление данными**: выбор места обработки данных с помощью Cognitive Services. Это может быть важно, если не удается отправить данные в облако, но требуется доступ к API-интерфейсы Cognitive Services. Поддержка согласованности в гибридной среде, в том числе данных, управления, идентификатора и безопасности.
- **Управление обновлениями модели**. гибкость в управлении версиями и обновлении моделей, развернутых в их решениях.
- **Переносимая архитектура**. позволяет создавать переносимую архитектуру приложений, которая может быть развернута в Azure, в локальной среде и на границе. Контейнеры можно развертывать непосредственно в [Службе Azure Kubernetes](../aks/index.yml), [Экземплярах контейнеров Azure](../container-instances/index.yml) или в кластере [Kubernetes](https://kubernetes.io/), развернутом в [Azure Stack](/azure-stack/operator). Дополнительные сведения см. в статье [Развертывание Kubernetes в Azure Stack](/azure-stack/user/azure-stack-solution-template-kubernetes-deploy).
- **Высокая пропускная способность и низкая задержка**. Предоставьте клиентам возможность масштабирования для повышения пропускной способности и низкой задержки, позволяя Cognitive Services физически близко к логике приложения и данным. Контейнеры не ограничивают число транзакций в секунду (TPS), и их можно увеличивать и уменьшать для удовлетворения спроса, предоставив необходимые аппаратные ресурсы.
- **Масштабируемость**: благодаря постоянно растущему популярности программного обеспечения для управления контейнерами и управления контейнерами, например Kubernetes; масштабируемость — это в плане развития технологических улучшений. Основываясь на масштабируемой кластерной среде, разработка приложений предусматривает высокий уровень доступности.

## <a name="containers-in-azure-cognitive-services"></a>Контейнеры в Azure Cognitive Services

Контейнеры Cognitive Services Azure предоставляют следующий набор контейнеров DOCKER, каждый из которых содержит подмножество функциональных возможностей служб в Azure Cognitive Services. Инструкции и расположения изображений можно найти в приведенных ниже таблицах. Также доступен список [образов контейнеров](containers/container-image-tags.md) .

### <a name="decision-containers"></a>Контейнеры принятия решений

| Служба |  Контейнер | Описание | Доступность |
|--|--|--|--|
| [Детектор аномалий][ad-containers] | **Детектор аномалий** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-decision-anomaly-detector))  | API Детектор аномалий позволяет отслеживать и обнаруживать отклонения в данных временных рядов с помощью машинного обучения. | Общедоступная версия |

### <a name="language-containers"></a>Контейнеры для работы с языком

| Служба |  Контейнер | Описание | Доступность |
|--|--|--|--|
| [LUIS][lu-containers] |  **LUIS** ([образ](https://go.microsoft.com/fwlink/?linkid=2043204&clcid=0x409)) | Загружает обученную или опубликованную модель Распознавания речи, также известную как приложение LUIS, в контейнер Docker и предоставляет доступ к прогнозам запроса из конечных точек API контейнера. Можно собирать журналы запросов из контейнера и отправлять их обратно на [портал LUIS](https://www.luis.ai) для повышения точности прогнозов приложения. | Общедоступная версия |
| [Анализ текста][ta-containers-keyphrase] | **Извлечение ключевых фраз** ([изображение](https://go.microsoft.com/fwlink/?linkid=2018757&clcid=0x409)) | Извлекает ключевые фразы для определения основных идей текста. Например, для входного текста "Еда была вкусной и персонал был замечательным" API вернет основные тезисы в записи: "еда" и "замечательный персонал". | Preview (Предварительный просмотр) |
| [Анализ текста][ta-containers-language] |  **Распознавание языка текста** ([изображение](https://go.microsoft.com/fwlink/?linkid=2018759&clcid=0x409)) | Решение может определить язык введенного текста (поддерживается более 120 языков) и сообщить один код языка для каждого документа, отправленного в запросе. Код языка сопряжен с показателем, указывающим степень оценки. | Preview (Предварительный просмотр) |
| [Анализ текста][ta-containers-sentiment] | **Анализ тональности v3** ([изображение](https://go.microsoft.com/fwlink/?linkid=2018654&clcid=0x409)) | Анализирует необработанный текст на предмет положительной или отрицательной тональности. Эта версия анализа тональности возвращает метки тональности (например, *положительный* или *отрицательный*) для каждого документа и предложения внутри него. |  Общедоступная версия |
| [Анализ текста][ta-containers-health] |  **Анализ текста для сферы здравоохранения** | Извлеките и пометка медицинских данных из неструктурированного текста клинической практике. | Предварительный просмотр с проверкой. [Запрос доступа][request-access]. |

### <a name="speech-containers"></a>Контейнеры Речи

> [!NOTE]
> Чтобы использовать речевые контейнеры, необходимо выполнить [форму запроса в Интернете](https://aka.ms/csgate).

| Служба |  Контейнер | Описание | Доступность |
|--|--|--|
| [API службы "Речь"][sp-containers-stt] |  Преобразование **речи в текст** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-custom-speech-to-text)) | Расшифровывает непрерывную речь в режиме реального времени в текстовый формат. | Общедоступная версия |
| [API службы "Речь"][sp-containers-cstt] | **Пользовательское распознавание речи к тексту** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-custom-speech-to-text)) | Расшифровывает непрерывной речи в режиме реального времени в текст с помощью пользовательской модели. | Общедоступная версия |
| [API службы "Речь"][sp-containers-tts] | Преобразование **текста в речь** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-text-to-speech)) | Преобразует текст в естественно звучащую речь. | Общедоступная версия |
| [API службы "Речь"][sp-containers-ctts] | **Пользовательский текст в речь** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-custom-text-to-speech)) | Преобразует текст в голосовую речь с помощью пользовательской модели. | Предварительный просмотр с проверкой |
| [API службы "Речь"][sp-containers-ntts] | **Текст нейронной речи** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-neural-text-to-speech)) | Преобразует текст в голосовую речь с помощью технологии глубокой нейронной сети, обеспечивая более естественную синтезированную речь. | Общедоступная версия |
| [API службы "Речь"][sp-containers-lid] | **Определение языка распознавания речи** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-language-detection)) | Определяет язык речевого звука. | Предварительный просмотр с проверкой |

### <a name="vision-containers"></a>Контейнеры Компьютерного зрения

> [!WARNING]
> 11 июня 2020 г. корпорация Майкрософт объявила о том, что она не будет продавать технологию распознавания лиц полицейским управлениям в США до тех пор, пока не вступят силу строгие правовые нормы, гарантирующие защиту прав человека. Таким образом, клиенты не смогут использовать возможности по распознаванию лиц или функции, включенные в службы Azure, такие как Распознавание лиц или Индексатор видео, если клиент является сотрудником полицейского управления США или разрешает использование таких служб управлением или для управления.

| Служба |  Контейнер | Описание | Доступность |
|--|--|--|--|
| [Компьютерное зрение][cv-containers] | **Чтение с оптического распознавания текста** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-vision-read)) | Контейнер для чтения и распознавания текста позволяет извлекать печатные и рукописные тексты из изображений и документов с поддержкой форматов JPEG, PNG, BMP, PDF и TIFF. Дополнительные сведения см. в [документации по API чтения](./computer-vision/concept-recognizing-text.md). | Предварительный просмотр с проверкой. [Запрос доступа][request-access]. |
| [Пространственный анализ][spa-containers] | **Пространственный анализ** ([изображение](https://hub.docker.com/_/microsoft-azure-cognitive-services-vision-spatial-analysis)) | Анализирует потоковую передачу видео в реальном времени, чтобы понять пространственные отношения между людьми, их движением и взаимодействием с объектами в физических средах. | Предварительный просмотр с проверкой. [Запрос доступа][request-access]. |
| [Распознавание лиц][fa-containers] | **Распознавание лиц** | Обнаруживает человеческие лица на изображениях и определяет методом машинного прогнозирования их характеристики, например черты лица (нос, глаза и т. д.), пол, возраст и многое другое. Кроме обнаружения лиц, API может сравнивать два лица на одном или на разных изображениях, используя оценку достоверности, а также искать по базе данных похожие или идентичные лица. Схожие лица можно упорядочивать в группы по близким визуальным характеристикам. | Недоступно |
| [Распознаватель форм][fr-containers] | **Распознаватель документов** | Понимание форм применяет технологию машинного обучения для обнаружения и извлечения пар "ключ-значение" и таблиц из форм. | Недоступно | 


<!--
|[Personalizer](./personalizer/what-is-personalizer.md) |F0, S0|**Personalizer** ([image](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409))|Azure Personalizer is a cloud-based API service that allows you to choose the best experience to show to your users, learning from their real-time behavior.|
-->

Кроме того, некоторые контейнеры поддерживаются в предложении Cognitive Services [нескольких сервисных ресурсов](cognitive-services-apis-create-account.md) . Вы можете создать один Cognitive Services ресурс "все в одном" и использовать один и тот же ключ выставления счетов в поддерживаемых службах для следующих служб:

* API Компьютерного зрения
* Распознавание лиц
* LUIS
* Анализ текста

## <a name="prerequisites"></a>Предварительные условия

Прежде чем использовать контейнеры Azure Cognitive Services, необходимо выполнить указанные ниже предварительные требования.

**Модуль Docker.** Модуль Docker нужно установить в локальной среде. Docker предоставляет пакеты для настройки среды с Docker для [macOS](https://docs.docker.com/docker-for-mac/), [Linux](https://docs.docker.com/engine/installation/#supported-platforms) и [Windows](https://docs.docker.com/docker-for-windows/). В Windows Docker нужно настроить для поддержки контейнеров Linux. Контейнеры Docker можно также развертывать непосредственно в [Службе Azure Kubernetes](../aks/index.yml) или [Экземплярах контейнеров Azure](../container-instances/index.yml).

Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure.

**Опыт работы с Реестром контейнеров Майкрософт и Docker.** Требуется базовое представление о Реестре контейнеров Майкрософт и понятиях Docker, таких как реестры, репозитории, контейнеры и образы контейнеров, а также знание основных команд `docker`.

Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).

Для некоторых контейнеров могут действовать дополнительные требования, в том числе к характеристикам сервера и (или) объему памяти.

[!INCLUDE [Cognitive Services container security](containers/includes/cognitive-services-container-security.md)]

## <a name="developer-samples"></a>Примеры для разработчиков

Примеры для разработчиков доступны в нашем [репозитории GitHub](https://github.com/Azure-Samples/cognitive-services-containers-samples).

## <a name="next-steps"></a>Дальнейшие действия

Узнайте о [рецептах контейнеров](containers/container-reuse-recipe.md) , которые можно использовать с Cognitive Services.

Установите и изучите функциональные возможности, предоставляемые контейнерами в Azure Cognitive Services:

* [Контейнеры детекторов аномалий][ad-containers]
* [Контейнеры Компьютерное зрение][cv-containers]
* [Контейнеры лиц][fa-containers]
* [Контейнеры распознавателя форм][fr-containers]
* [Контейнеры Распознавание речи (LUIS)][lu-containers]
* [Контейнеры API речевой службы][sp-containers]
* [Контейнеры Анализ текста][ta-containers]

<!--* [Personalizer containers](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409)
-->

[ad-containers]: anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: face/face-how-to-install-containers.md
[fr-containers]: form-recognizer/form-recognizer-container-howto.md
[lu-containers]: luis/luis-container-howto.md
[sp-containers]: speech-service/speech-container-howto.md
[spa-containers]: ./computer-vision/spatial-analysis-container.md
[sp-containers-lid]: speech-service/speech-container-howto.md?tabs=lid
[sp-containers-stt]: speech-service/speech-container-howto.md?tabs=stt
[sp-containers-cstt]: speech-service/speech-container-howto.md?tabs=cstt
[sp-containers-tts]: speech-service/speech-container-howto.md?tabs=tts
[sp-containers-ctts]: speech-service/speech-container-howto.md?tabs=ctts
[sp-containers-ntts]: speech-service/speech-container-howto.md?tabs=ntts
[ta-containers]: text-analytics/how-tos/text-analytics-how-to-install-containers.md
[ta-containers-keyphrase]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=keyphrase
[ta-containers-language]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=language
[ta-containers-sentiment]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment
[ta-containers-health]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=health
[request-access]: https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyQZ7B8Cg2FEjpibPziwPcZUNlQ4SEVORFVLTjlBSzNLRlo0UzRRVVNPVy4u