---
title: Установка и запуск контейнеров DOCKER для API анализа текста
titleSuffix: Azure Cognitive Services
description: Используйте контейнеры DOCKER для API анализа текста для выполнения обработки естественного языка, например для анализа тональности в локальной среде.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: aahi
keywords: локальный, Docker, контейнер, анализ тональности, обработка на естественном языке
ms.openlocfilehash: ac82781ed4a05fbbca7f5b16edb0b5349dfc400a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102432080"
---
# <a name="install-and-run-text-analytics-containers"></a>Установка и запуск контейнеров API анализа текста

> [!NOTE]
> * Контейнер для анализ тональности и обнаружения языков теперь общедоступен. Контейнер извлечения ключевых фраз доступен в виде общедоступной предварительной версии.
> * Связывание сущностей и NER в настоящее время недоступны в качестве контейнера.
> * Для доступа к Анализ текста контейнеру работоспособности требуется [Форма запроса](https://aka.ms/csgate). В настоящее время плата за использование не взимается.
> * Расположения образов контейнеров могли быть недавно изменены. Ознакомьтесь с этой статьей, чтобы увидеть обновленное расположение для этого контейнера.

Контейнеры позволяют запускать API-интерфейсы Анализа текста в собственной среде и прекрасно подходят для различных требований к безопасности и управлению данными. Контейнеры Анализ текста обеспечивают расширенную обработку естественного языка по необработанному тексту и включают три основные функции: анализ тональностиности, извлечение ключевых фраз и определение языка. 

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

> [!IMPORTANT]
> Бесплатная учетная запись ограничена 5 000 транзакциями в месяц, а для контейнеров допустимы только <a href="https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics" target="_blank">ценовые категории</a> " **бесплатный** " и " **стандартный** ". Дополнительные сведения о тарифах запросов транзакций см. в разделе [ограничения данных](../overview.md#data-limits).

## <a name="prerequisites"></a>Предварительные условия

Чтобы запустить любой из Анализ текста контейнеров, необходимо иметь главный компьютер и среды контейнеров.

## <a name="preparation"></a>Подготовка

Прежде чем использовать контейнеры Анализа текста, необходимо выполнить следующие условия:

|Обязательно|Назначение|
|--|--|
|Модуль Docker| На [главном компьютере](#the-host-computer) должен быть установлен модуль Docker. Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).<br><br> Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure. <br><br> **В ОС Windows** для Docker нужно также настроить поддержку контейнеров Linux.<br><br>|
|Опыт работы с Docker | Требуется базовое представление о понятиях Docker, включая реестры, репозитории, контейнеры и образы контейнеров, а также знание основных команд `docker`.| 
|Ресурс API анализа текста |Для использования контейнера необходимо следующее:<br><br>Ресурс Azure [анализ текста](../../cognitive-services-apis-create-account.md) с [ценовой категорией](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/)Free (F0) или Standard (S). Необходимо получить соответствующий ключ API и URI конечной точки, перейдя на страницу **ключа и конечной точки** ресурса в портал Azure. <br><br>**{API_KEY}**: один из двух доступных ключей ресурсов. <br><br>**{ENDPOINT_URI}**: конечная точка для ресурса. |

[!INCLUDE [Gathering required parameters](../../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>Главный компьютер

[!INCLUDE [Host Computer requirements](../../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Требования к контейнеру и рекомендации

В следующей таблице описаны минимальные и Рекомендуемые спецификации для контейнеров Анализ текста. Требуется по крайней мере 2 гигабайта (ГБ) памяти, и каждое ядро ЦП должно быть не менее 2,6 ГГц или быстрее. Также перечислены допустимые транзакции на раздел (TPS).

|  | Минимальные спецификации узла | Рекомендуемые спецификации узла | Минимальная Техническая спецификация | Максимальная техническая спецификация|
|---|---------|-------------|--|--|
| **Определение языка, извлечение ключевых фраз**   | 1 ядро, 2 ГБ памяти | 1 ядро, 4 ГБ памяти |15 | 30|
| **анализ тональности**;   | 1 ядро, 2 ГБ памяти | 4 ядра, память 8 ГБ |15 | 30|
| **Анализ текста для документа/запроса о работоспособности 1**   |  4 ядра, 10 ГБ памяти | 6 ядер, 12 ГБ памяти |15 | 30|
| **Анализ текста для документов и запросов Health-10**   |  6 ядер, 16 ГБ памяти | 8 ядер, память 20 ГБ |15 | 30|

Ядро ЦП и память соответствуют `--cpus` `--memory` параметрам и, которые используются как часть `docker run` команды.

## <a name="get-the-container-image-with-docker-pull"></a>Получение образа контейнера с помощью `docker pull`

[!INCLUDE [Tip for using docker list](../../../../includes/cognitive-services-containers-docker-list-tip.md)]

Образы контейнеров для Анализ текста доступны в реестре контейнеров Microsoft.

# <a name="sentiment-analysis"></a>[анализ тональности ](#tab/sentiment)

[!INCLUDE [docker-pull-sentiment-analysis-container](../includes/docker-pull-sentiment-analysis-container.md)]

# <a name="key-phrase-extraction-preview"></a>[Извлечение ключевых фраз (Предварительная версия)](#tab/keyphrase)

[!INCLUDE [docker-pull-key-phrase-extraction-container](../includes/docker-pull-key-phrase-extraction-container.md)]

# <a name="language-detection"></a>[распознавание языка](#tab/language);

[!INCLUDE [docker-pull-language-detection-container](../includes/docker-pull-language-detection-container.md)]

# <a name="text-analytics-for-health-preview"></a>[Анализ текста для здравоохранения (предварительная версия).](#tab/healthcare)

[!INCLUDE [docker-pull-health-container](../includes/docker-pull-health-container.md)]

***

## <a name="how-to-use-the-container"></a>Использование контейнера

После размещения контейнера на [главном компьютере](#the-host-computer) воспользуйтесь следующей процедурой для работы с ним.

1. [Запустите контейнер](#run-the-container-with-docker-run) с необходимыми настройками выставления счетов.
1. [Запросите конечную точку прогнозирования контейнера](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Запуск контейнера с помощью команды `docker run`

Используйте команду [DOCKER Run](https://docs.docker.com/engine/reference/commandline/run/) для запуска контейнеров. Контейнер продолжит работу, пока вы не завершите его работу.

> [!IMPORTANT]
> * В командах Docker в следующих разделах используется обратная косая черта (`\`) как символ продолжения строки. Замените или удалите ее в соответствии с требованиями вашей операционной системы. 
> * Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится.  Дополнительные сведения см. в [разделе о выставлении счетов](#billing).
> * Контейнеры "Анализ тональности" и "язык обнаружения" являются общедоступными. Контейнер извлечения ключевых фраз использует v2 API и находится в режиме предварительной версии.

# <a name="sentiment-analysis"></a>[анализ тональности](#tab/sentiment);

[!INCLUDE [docker-run-sentiment-analysis-container](../includes/docker-run-sentiment-analysis-container.md)]

# <a name="key-phrase-extraction-preview"></a>[Извлечение ключевых фраз (Предварительная версия)](#tab/keyphrase)

[!INCLUDE [docker-run-key-phrase-extraction-container](../includes/docker-run-key-phrase-extraction-container.md)]

# <a name="language-detection"></a>[распознавание языка](#tab/language);

[!INCLUDE [docker-run-language-detection-container](../includes/docker-run-language-detection-container.md)]

# <a name="text-analytics-for-health-preview"></a>[Анализ текста для здравоохранения (предварительная версия).](#tab/healthcare)

[!INCLUDE [docker-run-health-container](../includes/docker-run-health-container.md)]

***

[!INCLUDE [Running multiple containers on the same host](../../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="query-the-containers-prediction-endpoint"></a>Запрос конечной точки прогнозирования контейнера

Контейнер предоставляет интерфейсы REST API конечной точки прогнозирования запросов.

Используйте узел `http://localhost:5000` для API контейнера.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Остановка контейнера

[!INCLUDE [How to stop the container](../../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Устранение неполадок

Если контейнер запускается с выходным [подключением](../text-analytics-resource-container-config.md#mount-settings) и включенным ведением журнала, контейнер создает файлы журнала, которые удобно использовать для устранения неполадок, возникающих во время запуска или работы контейнера.

[!INCLUDE [Cognitive Services FAQ note](../../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Выставление счетов

Контейнеры API анализа текста отправляют данные для выставления счетов в Azure с помощью ресурса _API анализа текста_ в учетной записи Azure. 

[!INCLUDE [Container's Billing Settings](../../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Дополнительные сведения об этих параметрах см. в статье [Настройка контейнеров](../text-analytics-resource-container-config.md).

## <a name="summary"></a>Итоги

В этой статье вы узнали основные понятия и рабочий процесс для скачивания, установки и выполнения контейнеров Анализа текста. В разделе "Сводка" сделайте следующее.

* Анализ текста предоставляет три контейнера Linux для DOCKER, включая различные возможности:
   * *анализ тональности*;
   * *Извлечение ключевых фраз (Предварительная версия)* 
   * *распознавание языка*;
   * *Анализ текста для здравоохранения (предварительная версия).*
* Образы контейнеров загружаются из реестра контейнеров Майкрософт (мкр) или из репозитория предварительной версии.
* Образы контейнеров выполняются в Docker.
* Указав узел URI контейнера, пакет SDK или REST API можно использовать для вызова операций в контейнерах Анализа текста.
* При создании экземпляра контейнера нужно указать данные для выставления счетов.

> [!IMPORTANT]
> Контейнеры Cognitive Services не лицензируются для запуска без подключения к Azure для отслеживания использования. Клиенты должны разрешить контейнерам непрерывную передачу данных для выставления счетов в службу контроля потребления. Cognitive Services контейнеры не отправляют данные клиента (например, анализируемый текст) в корпорацию Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей о [конфигурации контейнеров](../text-analytics-resource-container-config.md).
* Чтобы решить проблемы, связанные с функциональностью, ознакомьтесь с [часто задаваемыми вопросами](../text-analytics-resource-faq.md).
