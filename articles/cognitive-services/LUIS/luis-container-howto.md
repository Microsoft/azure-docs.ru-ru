---
title: Установка и запуск контейнеров DOCKER для LUIS
titleSuffix: Azure Cognitive Services
description: Используйте контейнер LUIS для загрузки обученного или опубликованного приложения и получения доступа к своим прогнозам в локальной среде.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: aahi
keywords: локальная среда, Docker, контейнер
ms.openlocfilehash: e157e976186f03aa984877435c42b996ce476740
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102040198"
---
# <a name="install-and-run-docker-containers-for-luis"></a>Установка и запуск контейнеров DOCKER для LUIS

[!INCLUDE [container image location note](../containers/includes/image-location-note.md)]

Контейнеры позволяют использовать LUIS в собственной среде. Контейнеры соответствуют конкретным требованиям к безопасности и управлению данными. В этой статье вы узнаете, как скачать, установить и запустить контейнер LUIS.

Контейнер Распознавания речи (LUIS) загружает подготовленную или опубликованную модель Распознавания речи. Как [Luis приложение](https://www.luis.ai)контейнер DOCKER предоставляет доступ к прогнозам запросов из конечных точек API контейнера. Вы можете собирать журналы запросов из контейнера и отправлять их обратно в приложение Интеллектуальной службы распознавания речи для повышения точности прогнозов приложения.

В следующем видео показаны способы использования этого контейнера.

[![Демонстрация контейнера для Cognitive Services](./media/luis-container-how-to/luis-containers-demo-video-still.png)](https://aka.ms/luis-container-demo)

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы запустить контейнер LUIS, обратите внимание на следующие предварительные требования.

|Обязательно|Назначение|
|--|--|
|Модуль Docker| На [главном компьютере](#the-host-computer) должен быть установлен модуль Docker. Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).<br><br> Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure. <br><br> **В ОС Windows** для Docker нужно также настроить поддержку контейнеров Linux.<br><br>|
|Опыт работы с Docker | Требуется базовое представление о понятиях Docker, включая реестры, репозитории, контейнеры и образы контейнеров, а также знание основных команд `docker`.|
|`Cognitive Services`Ресурс Azure и файл [упакованного приложения](luis-how-to-start-new-app.md) Luis |Для использования контейнера необходимо следующее:<br><br>* Ресурс Azure _Cognitive Services_ и соответствующий ключ выставления счетов URI конечной точки выставления счетов. Оба значения доступны на страницах обзора и ключей для ресурса и необходимы для запуска контейнера. <br>* Обученное или опубликованное приложение, упакованное в виде подключенных входных данных к контейнеру со связанным идентификатором приложения. Пакетный файл можно получить на портале LUIS или в API-интерфейсах разработки. Если вы получаете упакованное приложение LUIS из API- [интерфейсов разработки](#authoring-apis-for-package-file), вам также потребуется ваш _ключ разработки_.<br><br>Эти требования используются для передачи аргументов командной строки в следующие переменные:<br><br>**{AUTHORING_KEY}**: этот ключ используется для получения упакованного приложения из службы Luis в облаке и отправки журналов запросов обратно в облако. Формат — `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`.<br><br>**{APP_ID}**: этот идентификатор используется для выбора приложения. Формат — `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.<br><br>**{API_KEY}**: этот ключ используется для запуска контейнера. Ключ конечной точки можно найти в двух местах: Первый — это портал Azure в списке ключей ресурса _Cognitive Services_ . а также на портале LUIS на странице параметров ключей и конечных точек. Не используйте ключ starter.<br><br>**{ENDPOINT_URI}**: конечная точка, указанная на странице обзора.<br><br>[Ключ разработки и ключ конечной точки](luis-limits.md#key-limits) служат для разных целей. Не используйте их как взаимозаменяемые. |

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="authoring-apis-for-package-file"></a>Создание API-интерфейсов для файла пакета

Создание API для упакованных приложений:

* [API опубликованного пакета](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip)
* [Неопубликованный, только обученный пакет API пакета](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip)

### <a name="the-host-computer"></a>Главный компьютер

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Требования к контейнеру и рекомендации

В таблице ниже приведены минимальные и рекомендуемые значения для узла контейнера. Требования могут изменяться в зависимости от объема трафика.

|Контейнер| Минимальные | Рекомендуемая | ПЛАТЫ<br>(Минимум, максимум)|
|-----------|---------|-------------|--|
|LUIS|1 ядро, 2 ГБ памяти|1 ядро, 4 ГБ памяти|20, 40|

* Частота каждого ядра должна быть минимум 2,6 ГГц.
* TPS — транзакций в секунду.

Ядро и память соответствуют параметрам `--cpus` и `--memory`, которые используются как часть команды `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Получение образа контейнера с помощью `docker pull`

Используйте [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) команду, чтобы скачать образ контейнера из `mcr.microsoft.com/azure-cognitive-services/language/luis` репозитория:

```
docker pull mcr.microsoft.com/azure-cognitive-services/language/luis:latest
```

Полное описание доступных тегов, таких как `latest` из предыдущей команды, см. в разделе [LUIS](https://go.microsoft.com/fwlink/?linkid=2043204) на сайте Docker Hub.

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

## <a name="how-to-use-the-container"></a>Использование контейнера

После размещения контейнера на [главном компьютере](#the-host-computer) воспользуйтесь следующей процедурой для работы с ним.

![Процесс использования контейнера Интеллектуальной службы распознавания речи (LUIS)](./media/luis-container-how-to/luis-flow-with-containers-diagram.jpg)

1. [Экспортируйте пакет](#export-packaged-app-from-luis) для контейнера с портала LUIS или из API LUIS.
1. Переместите файл пакета в необходимый каталог **input** на [главном компьютере](#the-host-computer). Не переименовывайте, не изменяйте, не перезаписывайте или распаковывает файл пакета LUIS.
1. [Запустите контейнер](#run-the-container-with-docker-run) с необходимыми настройками _input mount_ и выставления счетов. Доступны дополнительные [примеры](luis-container-configuration.md#example-docker-run-commands) команды `docker run`.
1. Выполните [запрос конечной точки прогнозирования контейнера](#query-the-containers-prediction-endpoint).
1. Завершив работу с контейнером, [импортируйте журналы конечной точки](#import-the-endpoint-logs-for-active-learning) из выходного подключения на портале LUIS и [остановите](#stop-the-container) контейнер.
1. Воспользуйтесь средствами [активного обучения](luis-how-to-review-endpoint-utterances.md) на портале LUIS или страницей **просмотра фраз конечной точки** для улучшения приложения.

Приложение, выполняющееся в контейнере, изменить нельзя. Чтобы изменить приложение в контейнере, необходимо изменить приложение в службе LUIS с помощью портала [Luis](https://www.luis.ai) или с помощью [API-интерфейсов разработки](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f)Luis. Затем выполните обучение и (или) публикацию, скачайте новый пакет и снова запустите контейнер.

Приложение LUIS, содержащееся в контейнере, нельзя экспортировать обратно в службу LUIS. Отправить можно только журналы запросов.

## <a name="export-packaged-app-from-luis"></a>Экспорт упакованного приложения из LUIS

Контейнер LUIS ожидает от обученного или опубликованного приложения LUIS ответа на запросы прогноза пользовательских фраз. Чтобы получить приложение LUIS, используйте API обученного или опубликованного пакета.

Расположение по умолчанию — вложенный каталог `input` по отношению к каталогу, для которого была выполнена команда `docker run`.

Поместите файл пакета в каталог и укажите ссылку этот каталог как входное подключение при запуске контейнера Docker.

### <a name="package-types"></a>Типы пакетов

Входной каталог подключения может содержать **рабочие**, **промежуточные** и **версии** моделей приложения одновременно. Все пакеты являются подключенными.

|Тип пакета|Запрос API конечной точки|Запрос доступности|Формат имени файла пакета|
|--|--|--|--|
|Версиями|GET, POST|Только контейнер|`{APP_ID}_v{APP_VERSION}.gz`|
|Промежуточная|GET, POST|Azure и контейнер|`{APP_ID}_STAGING.gz`|
|Производство|GET, POST|Azure и контейнер|`{APP_ID}_PRODUCTION.gz`|

> [!IMPORTANT]
> Не переименовывайте, не изменяйте, перезаписывают или распаковывает файлы пакетов LUIS.

### <a name="packaging-prerequisites"></a>Необходимые условия для упаковки

Перед упаковкой приложения LUIS потребуется следующее:

|Требования к упаковке|Сведения|
|--|--|
|Экземпляр ресурса _Cognitive Services_ Azure|Поддерживаемые регионы:<br><br>Западная часть США (`westus`)<br>Западная Европа (`westeurope`)<br>Восточная Австралия (`australiaeast`)|
|Обученное или опубликованное приложение LUIS|Без [неподдерживаемых зависимостей][unsupported-dependencies]. |
|Доступ к файловой системе [главного компьютера](#the-host-computer) |Главный компьютер должен разрешать [входные подключения](luis-container-configuration.md#mount-settings).|

### <a name="export-app-package-from-luis-portal"></a>Экспорт пакета приложения с портала LUIS

На [портале](https://www.luis.ai) LUIS можно экспортировать пакет обученного или опубликованного приложения.

### <a name="export-published-apps-package-from-luis-portal"></a>Экспорт пакета опубликованного приложения с портала LUIS

Пакет опубликованного приложения доступен на странице списка **My Apps** (Мои приложения).

1. Войдите на портал [LUIS](https://www.luis.ai).
1. Установите флажок слева от имени приложения в списке.
1. Выберите элемент **Export** (Экспорт) на контекстной панели инструментов над списком.
1. Выберите **Export for container (GZIP)** (Экспорт для контейнера (GZIP)).
1. Выберите **Production slot** (Рабочий слот) или **Staging slot** (Промежуточный слот).
1. Пакет будет скачан из браузера.

![Экспорт опубликованного пакета для контейнера из меню экспорта на странице приложения](./media/luis-container-how-to/export-published-package-for-container.png)

### <a name="export-versioned-apps-package-from-luis-portal"></a>Экспорт пакета приложения с управлением версиями с портала LUIS

Пакет приложения для управления версиями доступен на странице списка **версий** .

1. Войдите на портал [LUIS](https://www.luis.ai).
1. Выберите приложение в списке.
1. Выберите **Manage** (Управление) на панели навигации приложения.
1. Выберите **Versions** (Версии) на панели навигации слева.
1. Установите флажок слева от имени версии в списке.
1. Выберите элемент **Export** (Экспорт) на контекстной панели инструментов над списком.
1. Выберите **Export for container (GZIP)** (Экспорт для контейнера (GZIP)).
1. Пакет будет скачан из браузера.

![Экспорт обученного пакета для контейнера из меню экспорта на странице версий](./media/luis-container-how-to/export-trained-package-for-container.png)

### <a name="export-published-apps-package-from-api"></a>Экспорт пакета опубликованного приложения из API

Воспользуйтесь следующим методом REST API, чтобы упаковать приложение LUIS, которое уже [опубликовано](luis-how-to-publish-app.md). Замените заполнители в вызове API собственными соответствующими значениями, используя таблицу, приведенную под спецификацией HTTP.

```http
GET /luis/api/v2.0/package/{APP_ID}/slot/{SLOT_NAME}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| Заполнитель | Значение |
|-------------|-------|
| **{APP_ID}** | Идентификатор опубликованного приложения LUIS. |
| **{SLOT_NAME}** | Среда опубликованного приложения LUIS. Используйте одно из следующих значений:<br/>`PRODUCTION`<br/>`STAGING` |
| **{AUTHORING_KEY}** | Ключ разработки учетной записи LUIS для опубликованного приложения LUIS.<br/>Ключ разработки можно получить на странице **User Settings** (Параметры пользователя) на портале LUIS. |
| **{AZURE_REGION}** | Соответствующий регион Azure:<br/><br/>`westus` — западная часть США<br/>`westeurope` — Западная Европа<br/>`australiaeast` — Восточная Австралия |

Чтобы скачать опубликованный пакет, см. [документацию по API здесь][download-published-package]. Если загрузка успешно завершена, ответ представляет собой файл пакета LUIS. Сохраните файл в месте хранения, указанном для входного подключения контейнера.

### <a name="export-versioned-apps-package-from-api"></a>Экспорт пакета приложения с управлением версиями из API

Воспользуйтесь следующим методом REST API, чтобы упаковать приложение LUIS, которое уже [обучено](luis-how-to-train.md). Замените заполнители в вызове API собственными соответствующими значениями, используя таблицу, приведенную под спецификацией HTTP.

```http
GET /luis/api/v2.0/package/{APP_ID}/versions/{APP_VERSION}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| Заполнитель | Значение |
|-------------|-------|
| **{APP_ID}** | Идентификатор приложения обученного приложения LUIS. |
| **{APP_VERSION}** | Версия приложения обученного LUIS приложения. |
| **{AUTHORING_KEY}** | Ключ разработки учетной записи LUIS для опубликованного приложения LUIS.<br/>Ключ разработки можно получить на странице **User Settings** (Параметры пользователя) на портале LUIS. |
| **{AZURE_REGION}** | Соответствующий регион Azure:<br/><br/>`westus` — западная часть США<br/>`westeurope` — Западная Европа<br/>`australiaeast` — Восточная Австралия |

Чтобы скачать пакет с управлением версиями, см. [документацию по API здесь][download-versioned-package]. Если загрузка успешно завершена, ответ представляет собой файл пакета LUIS. Сохраните файл в месте хранения, указанном для входного подключения контейнера.

## <a name="run-the-container-with-docker-run"></a>Запуск контейнера с помощью команды `docker run`

Воспользуйтесь командой [docker run](https://docs.docker.com/engine/reference/commandline/run/) для запуска контейнера. Дополнительные сведения о том, как получить значения и, см. в разделе [сбор обязательных параметров](#gathering-required-parameters) `{ENDPOINT_URI}` `{API_KEY}` .

Доступны [примеры](luis-container-configuration.md#example-docker-run-commands) `docker run` команд.

```console
docker run --rm -it -p 5000:5000 ^
--memory 4g ^
--cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output\,target=/output ^
mcr.microsoft.com/azure-cognitive-services/language/luis ^
Eula=accept ^
Billing={ENDPOINT_URI} ^
ApiKey={API_KEY}
```

* В этом примере каталог находится за пределами `C:` диска во избежание конфликтов разрешений в Windows. Если вам нужен конкретный каталог для входных данных, может потребоваться предоставить соответствующие разрешения службе Docker.
* Не изменяйте порядок аргументов, если вы не знакомы с контейнерами Docker.
* Если используется другая операционная система, используйте правильную консоль или терминал, синтаксис папок для подключений и символ продолжения строки для вашей системы. В этих примерах предполагается наличие в консоли Windows символа продолжения строки `^` . Так как контейнер является операционной системой Linux, целевое подключение использует синтаксис папки в формате Linux.

Команда:

* запускает контейнер из образа контейнера LUIS;
* Загружает приложение LUIS из входных данных Mount в *к:\инпут*, расположенном на узле контейнера.
* выделяет 2 ядра ЦП и 4 ГБ памяти;
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* Сохраняет журналы контейнеров и LUIS для подключения к выходу в *C:\output*, расположенном на узле контейнера.
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

Доступны дополнительные [примеры](luis-container-configuration.md#example-docker-run-commands) команды `docker run`.

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится.  Дополнительные сведения см. в [разделе о выставлении счетов](#billing).
> Значение ApiKey — это **ключ** на странице **ресурсов Azure** на портале Luis. Он также доступен на `Cognitive Services` странице ключей ресурсов Azure.

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="endpoint-apis-supported-by-the-container"></a>Интерфейсы API конечной точки, поддерживаемые контейнером

В контейнере доступны версии v2 и [v3](luis-migration-api-v3.md) API.

## <a name="query-the-containers-prediction-endpoint"></a>Запрос конечной точки прогнозирования контейнера

Контейнер предоставляет интерфейсы REST API конечной точки прогнозирования запросов. Конечные точки для опубликованных (промежуточных или рабочих) приложений имеют _разные_ маршруты, чем конечные точки для приложений с управлением версиями.

Используйте узел `http://localhost:5000` для API контейнера.

# <a name="v3-prediction-endpoint"></a>[Конечная точка прогнозирования V3](#tab/v3)

|Тип пакета|HTTP-команда|Маршрут|Параметры запроса|
|--|--|--|--|
|Опубликован|GET, POST|`/luis/v3.0/apps/{appId}/slots/{slotName}/predict?`|`query={query}`<br>[`&verbose`]<br>[`&log`]<br>[`&show-all-intents`]|
|Версиями|GET, POST|`/luis/v3.0/apps/{appId}/versions/{versionId}/predict?`|`query={query}`<br>[`&verbose`]<br>[`&log`]<br>[`&show-all-intents`]|

Параметры запроса определяют содержимое ответа на запрос и способ его получения:

|Параметр запроса|Тип|Назначение|
|--|--|--|
|`query`|строка|Фраза пользователя.|
|`verbose`|Логическое|Логическое значение, указывающее, следует ли возвращать все метаданные для прогнозируемых моделей. Значение по умолчанию — false.|
|`log`|Логическое|Записывает запросы в журнал, который затем можно использовать для [активного обучения](luis-how-to-review-endpoint-utterances.md). Значение по умолчанию — false.|
|`show-all-intents`|Логическое|Логическое значение, указывающее, следует ли возвращать только цели или цель наибольшей оценки. Значение по умолчанию — false.|

# <a name="v2-prediction-endpoint"></a>[Конечная точка прогнозирования V2](#tab/v2)

|Тип пакета|HTTP-команда|Маршрут|Параметры запроса|
|--|--|--|--|
|Опубликован|[Get](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee78), [POST](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee79)|`/luis/v2.0/apps/{appId}?`|`q={q}`<br>`&staging`<br>[`&timezoneOffset`]<br>[`&verbose`]<br>[`&log`]<br>|
|Версиями|GET, POST|`/luis/v2.0/apps/{appId}/versions/{versionId}?`|`q={q}`<br>[`&timezoneOffset`]<br>[`&verbose`]<br>[`&log`]|

Параметры запроса определяют содержимое ответа на запрос и способ его получения:

|Параметр запроса|Тип|Назначение|
|--|--|--|
|`q`|строка|Фраза пользователя.|
|`timezoneOffset`|Число|Параметр timezoneOffset позволяет [изменить часовой пояс](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity), используемый предварительно созданной сущностью datetimeV2.|
|`verbose`|Логическое|Возвращает все намерения и их оценки, если задано значение true. По умолчанию задано значение false, при котором возвращается только верхнее намерение.|
|`staging`|Логическое|Возвращает запрос из результатов промежуточной среды, если задано значение true. |
|`log`|Логическое|Записывает запросы в журнал, который затем можно использовать для [активного обучения](luis-how-to-review-endpoint-utterances.md). Значение по умолчанию — true.|

***

### <a name="query-the-luis-app"></a>Запрос к приложению LUIS

Ниже приведен пример команды CURL для выполнения запросов к контейнеру для опубликованного приложения:

# <a name="v3-prediction-endpoint"></a>[Конечная точка прогнозирования V3](#tab/v3)

Чтобы запросить модель в слоте, используйте следующий API:

```bash
curl -G \
-d verbose=false \
-d log=true \
--data-urlencode "query=turn the lights on" \
"http://localhost:5000/luis/v3.0/apps/{APP_ID}/slots/production/predict"
```

Чтобы выполнить запросы к **промежуточной** среде, замените `production` в маршруте на `staging` :

`http://localhost:5000/luis/v3.0/apps/{APP_ID}/slots/staging/predict`

Чтобы запросить модель с управлением версиями, используйте следующий API:

```bash
curl -G \
-d verbose=false \
-d log=false \
--data-urlencode "query=turn the lights on" \
"http://localhost:5000/luis/v3.0/apps/{APP_ID}/versions/{APP_VERSION}/predict"
```

# <a name="v2-prediction-endpoint"></a>[Конечная точка прогнозирования V2](#tab/v2)

Чтобы запросить модель в слоте, используйте следующий API:

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APP_ID}?q=turn%20on%20the%20lights&staging=false&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
Для выполнения запросов к **промежуточной** среде измените значение параметра **staging** строки запроса на true:

`staging=true`

Чтобы запросить модель с управлением версиями, используйте следующий API:

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APP_ID}/versions/{APP_VERSION}?q=turn%20on%20the%20lights&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
Имя версии имеет максимальную длину 10 символов и содержит только символы, разрешенные в URL-адресе.

***

## <a name="import-the-endpoint-logs-for-active-learning"></a>Импорт журналов конечной точки для активного обучения

Если для контейнера LUIS указана подключенная выходная папка, файлы журнала запросов приложений сохраняются в выходном каталоге, где `{INSTANCE_ID}` — это идентификатор контейнера. Журнал запросов приложения содержит запрос, ответ и метки времени для каждого прогнозирующего запроса, отправленного в контейнер LUIS.

В следующем расположении показана вложенная структура каталогов для файлов журнала контейнера.
```
/output/luis/{INSTANCE_ID}/
```

На портале LUIS выберите приложение, затем выберите **Import endpoint logs** (Импорт журналов конечной точки) для отправки этих журналов.

![Импорт файлов журнала контейнера для активного обучения](./media/luis-container-how-to/upload-endpoint-log-files.png)

После отправки журнала [просмотрите фразы конечной точки](./luis-concept-review-endpoint-utterances.md) на портале LUIS.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Остановка контейнера

Чтобы завершить работу контейнера, в среде командной строки, где выполняется контейнер, нажмите клавиши **CTRL+C**.

## <a name="troubleshooting"></a>Устранение неполадок

Если контейнер запускается с выходным [подключением](luis-container-configuration.md#mount-settings) и включенным ведением журнала, контейнер создает файлы журнала, которые удобно использовать для устранения неполадок, возникающих во время запуска или работы контейнера.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Выставление счетов

Контейнер LUIS отправляет сведения о выставлении счетов в Azure с помощью _Cognitive Services_ ресурса в учетной записи Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Дополнительные сведения об этих параметрах см. в статье [Настройка контейнеров](luis-container-configuration.md).

## <a name="summary"></a>Итоги

В этой статье описаны основные понятия и рабочий процесс для скачивания, установки и выполнения контейнеров Интеллектуальной службы распознавания речи (LUIS). В разделе "Сводка" сделайте следующее.

* Интеллектуальная служба распознавания речи (LUIS) предоставляет один контейнер Linux для Docker, обеспечивая прогнозы фраз по запросам конечных точек.
* Образы контейнеров скачиваются из Реестра контейнеров Майкрософт (MCR).
* Образы контейнеров выполняются в Docker.
* С помощью REST API можно выполнять запросы к конечным точкам контейнера, указывая URI узла контейнера.
* При создании экземпляра контейнера нужно указать данные для выставления счетов.

> [!IMPORTANT]
> Контейнеры Cognitive Services не лицензируются для запуска без подключения к Azure для отслеживания использования. Клиенты должны разрешить контейнерам непрерывную передачу данных для выставления счетов в службу контроля потребления. Контейнеры Cognitive Services не отправляют в корпорацию Майкрософт данные клиента (например анализируемые изображения или тексты).

## <a name="next-steps"></a>Дальнейшие действия

* Проверьте настройки [контейнеров](luis-container-configuration.md) на наличие параметров конфигурации.
* См. раздел [ограничения контейнера Luis](luis-container-limitations.md) для известных ограничений возможностей.
* Ознакомьтесь со статьей об [устранение неполадок](troubleshooting.md), чтобы устранить проблемы, связанные с функциональностью LUIS.
* Воспользуйтесь [дополнительными контейнерами Cognitive Services](../cognitive-services-container-support.md)

<!-- Links - external -->
[download-published-package]: https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip
[download-versioned-package]: https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip

[unsupported-dependencies]: luis-container-limitations.md#unsupported-dependencies-for-latest-container