---
title: Установка и запуск контейнеров DOCKER для API распознавания лиц
titleSuffix: Azure Cognitive Services
description: Используйте контейнер DOCKER для API распознавания лиц, чтобы обнаружить и определить людей в образах.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 02/23/2021
ms.author: aahi
keywords: локальная среда, Docker, контейнер, обнаружение
ms.openlocfilehash: 36cbd7bd24304871593b107f9b8ed9be02ce46de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101706798"
---
# <a name="install-and-run-face-containers-preview"></a>Установка и запуск контейнеров лиц (Предварительная версия)

> [!IMPORTANT]
> Достигнут предел для пользователей контейнера Распознавания лиц. Сейчас мы не принимаем новые приложения для контейнера Распознавания лиц.

Azure Cognitive Services API распознавания лиц предоставляет контейнер DOCKER для Linux, который обнаруживает и анализирует человеческие лица в изображениях. Он также определяет атрибуты, которые включают ориентиры для лиц, такие как нос и глаза, пол, возраст и другие функции, прогнозируемые с точки зрения машинного лица. Кроме обнаружения, лицо может проверить, совпадают ли два лица в одном или разных изображениях с помощью оценки достоверности. Кроме того, лицо может сравнить лица с базой данных, чтобы узнать, существует ли уже похожий или аналогичный лицом. Он также может организовывать схожие лица в группы с помощью общих визуальных характеристик.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем использовать контейнеры службы лиц, необходимо выполнить следующие предварительные требования.

|Обязательно|Назначение|
|--|--|
|Модуль Docker| Подсистема DOCKER должна быть установлена на [главном компьютере](#the-host-computer). Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).<br><br> Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure. <br><br> В Windows DOCKER также должен быть настроен для поддержки контейнеров Linux.<br><br>|
|Опыт работы с Docker | Вам потребуется базовое понимание концепций DOCKER, таких как реестры, репозитории, контейнеры и образы контейнеров. Также необходимо знание основных `docker` команд.| 
|Ресурс лица |Для использования этого контейнера необходимо следующее:<br><br>Ресурс **лица** Azure, а также соответствующий ключ API и URI конечной точки. Оба значения доступны на страницах **обзора** и **ключей** для ресурса. Они необходимы для запуска контейнера.<br><br>**{API_KEY}**: один из двух доступных ключей ресурсов на странице " **ключи** "<br><br>**{ENDPOINT_URI}**: конечная точка, указанная на странице **обзора**

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="the-host-computer"></a>Главный компьютер

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Требования к контейнеру и рекомендации

В следующей таблице описаны минимальные и Рекомендуемые ядра ЦП и память, выделяемые для каждого контейнера службы лиц.

| Контейнер | Минимальные | Рекомендуемая | Транзакций в секунду<br>(Минимум, максимум)|
|-----------|---------|-------------|--|
|Распознавание лиц | 1 ядро, 2 ГБ памяти | 1 ядро, 4 ГБ памяти |10, 20|

* Каждое ядро должно иметь частоту не менее 2,6 ГГц или быстрее.
* Количество транзакций в секунду (TPS).

Ядро и память соответствуют параметрам `--cpus` и `--memory`, которые используются как часть команды `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Получение образа контейнера с помощью docker pull

Доступны образы контейнеров для службы распознавания лиц. 

| Контейнер | Хранилище |
|-----------|------------|
| Распознавание лиц | `containerpreview.azurecr.io/microsoft/cognitive-services-face:latest` |

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-face-container"></a>docker pull для контейнера распознавания лиц

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-face:latest
```

## <a name="use-the-container"></a>Использование контейнера

После размещения контейнера на [главном компьютере](#the-host-computer) выполните приведенные ниже инструкции для работы с ним.

1. [Запустите контейнер](#run-the-container-with-docker-run) с необходимыми параметрами выставления счетов. Доступны дополнительные [примеры](./face-resource-container-config.md#example-docker-run-commands) команды `docker run`. 
1. [Запросите конечную точку прогнозирования контейнера](#query-the-containers-prediction-endpoint). 

## <a name="run-the-container-with-docker-run"></a>Запуск контейнера с помощью запуска DOCKER

Воспользуйтесь командой [docker run](https://docs.docker.com/engine/reference/commandline/run/) для запуска контейнера. Дополнительные сведения о том, как получить значения и, см. в разделе [сбор обязательных параметров](#gathering-required-parameters) `{ENDPOINT_URI}` `{API_KEY}` .

Доступны [примеры](face-resource-container-config.md#example-docker-run-commands) `docker run` команд.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-face \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Запускает контейнер лицевой стороны из образа контейнера.
* Выделяется одно ядро ЦП и 4 ГБ памяти.
* Предоставляет TCP-порт 5000 и выделяет псевдо для контейнера.
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере. 

Доступны дополнительные [примеры](./face-resource-container-config.md#example-docker-run-commands) команды `docker run`. 

> [!IMPORTANT]
> `Eula` `Billing` `ApiKey` Для запуска контейнера необходимо указать параметры, и, чтобы контейнер не запускался. Дополнительные сведения см. в [разделе о выставлении счетов](#billing).

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]


## <a name="query-the-containers-prediction-endpoint"></a>Запрос конечной точки прогнозирования контейнера

Контейнер предоставляет интерфейсы REST API конечной точки прогнозирования запросов. 

Используйте узел `http://localhost:5000` для API контейнера.


<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Остановка контейнера

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Устранение неполадок

Если вы запускаете контейнер с выходным [подключением](./face-resource-container-config.md#mount-settings) и ведение журнала включено, контейнер создает файлы журнала, которые полезны для устранения неполадок, возникающих при запуске или запуске контейнера.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Выставление счетов

Контейнеры службы лиц отправляют сведения о выставлении счетов в Azure с помощью ресурса лицевой стороны в вашей учетной записи Azure. 

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Дополнительные сведения об этих параметрах см. в статье [Настройка контейнеров](./face-resource-container-config.md).

## <a name="summary"></a>Итоги

В этой статье вы узнали основные понятия и рабочий процесс по скачиванию, установке и запуску контейнеров службы лиц. В разделе "Сводка" сделайте следующее.

* Образы контейнеров загружаются из реестра контейнеров Azure.
* Образы контейнеров выполняются в Docker.
* Вы можете использовать REST API или пакет SDK для вызова операций в контейнерах службы лиц, указав URI узла контейнера.
* При создании экземпляра контейнера необходимо указать сведения о выставлении счетов.

> [!IMPORTANT]
> Cognitive Services контейнеры не лицензированы для запуска без подключения к Azure для измерения. Клиенты должны включить контейнеры для передачи сведений о выставлении счетов в службу контроля использования в любое время. Контейнеры Cognitive Services не отправляют в корпорацию Майкрософт данные клиента, например анализируемые изображения или тексты.

## <a name="next-steps"></a>Дальнейшие действия

* Параметры конфигурации см. в разделе [Configure Containers](face-resource-container-config.md).
* Дополнительные сведения об обнаружении и определении лиц см. в разделе [Обзор лиц](Overview.md).
* Дополнительные сведения о методах, поддерживаемых контейнером, см. в [API распознавания лиц](//westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).
* Чтобы использовать больше контейнеров Cognitive Services, см. раздел [Cognitive Services Containers](../cognitive-services-container-support.md).
