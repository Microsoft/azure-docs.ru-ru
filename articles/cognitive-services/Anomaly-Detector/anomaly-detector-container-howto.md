---
title: Установка и запуск контейнеров DOCKER для API детектора аномалий
titleSuffix: Azure Cognitive Services
description: Используйте алгоритмы API детектора аномалий для поиска аномалий в данных локально с помощью контейнера DOCKER.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 09/28/2020
ms.author: mbullwin
ms.custom: cog-serv-seo-aug-2020
keywords: локальная среда, Docker, контейнер, потоковая передача, алгоритмы
ms.openlocfilehash: 70e5950f6577ce2cca2f28be070f3ba372d46a7e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97862312"
---
# <a name="install-and-run-docker-containers-for-the-anomaly-detector-api"></a>Установка и запуск контейнеров DOCKER для API детектора аномалий 

[!INCLUDE [container image location note](../containers/includes/image-location-note.md)]

Контейнеры позволяют использовать API детектора аномалий в собственной среде. Контейнеры соответствуют конкретным требованиям к безопасности и управлению данными. В этой статье вы узнаете, как скачать, установить и запустить контейнер детекторов аномалий.

Детектор аномалий предлагает один контейнер DOCKER для локального использования API. Используйте контейнер для:
* Использование алгоритмов детектора аномалии для данных
* Отслеживайте потоковые данные и выявлять аномалии в режиме реального времени.
* Обнаружение аномалий набора данных в пакетном режиме. 
* Обнаружение точек изменения тенденций в наборе данных в виде пакета.
* Настройте чувствительность алгоритма обнаружения аномалий, чтобы лучше соответствовать вашим данным.

Подробные сведения об API см. в следующих статьях:
* [Дополнительные сведения о службе API детектора аномалий](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем использовать контейнеры детекторов аномалий, необходимо выполнить следующие предварительные требования:

|Обязательно|Назначение|
|--|--|
|Модуль Docker| На [главном компьютере](#the-host-computer) должен быть установлен модуль Docker. Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).<br><br> Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure. <br><br> **В ОС Windows** для Docker нужно также настроить поддержку контейнеров Linux.<br><br>|
|Опыт работы с Docker | Требуется базовое представление о понятиях Docker, включая реестры, репозитории, контейнеры и образы контейнеров, а также знание основных команд `docker`.|
|Ресурс детектора аномалий |Для использования контейнеров необходимо следующее:<br><br>Ресурс _детектора аномалий_ Azure для получения соответствующего ключа API и URI конечной точки. Оба значения доступны на страницах "Обзор **детекторов аномалий** " и "ключи" портал Azure и необходимы для запуска контейнера.<br><br>**{API_KEY}**: один из двух доступных ключей ресурсов на странице " **ключи** "<br><br>**{ENDPOINT_URI}**: конечная точка, указанная на странице **обзора**|

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>Главный компьютер

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

<!--* [Azure IoT Edge](../../iot-edge/index.yml). For instructions of deploying Anomaly Detector module in IoT Edge, see [How to deploy Anomaly Detector module in IoT Edge](how-to-deploy-anomaly-detector-module-in-iot-edge.md).-->

### <a name="container-requirements-and-recommendations"></a>Требования к контейнеру и рекомендации

В следующей таблице описаны минимальные и Рекомендуемые ядра ЦП и память, выделяемые для контейнера детекторов обнаружения аномалий.

| QPS (запросов в секунду) | Минимальные | Рекомендуемая |
|-----------|---------|-------------|
| 10 QPS | 4 ядра, 1 ГБ памяти | 8-ядерный 2 ГБ памяти |
| 20 QPS | 8 ядер, 2 ГБ памяти | 16-ядерный 4 ГБ памяти |

Частота каждого ядра должна быть минимум 2,6 ГГц.

Ядро и память соответствуют параметрам `--cpus` и `--memory`, которые используются как часть команды `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Получение образа контейнера с помощью `docker pull`

Используйте [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) команду, чтобы скачать образ контейнера.

| Контейнер | Хранилище |
|-----------|------------|
| обнаружение аномалий-службы-детектор | `mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector:latest` |

<!--
For a full description of available tags, such as `latest` used in the preceding command, see [anomaly-detector](https://go.microsoft.com/fwlink/?linkid=2083827&clcid=0x409) on Docker Hub.
-->
[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-anomaly-detector-container"></a>Извлечение DOCKER для контейнера детекторов аномалий

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/anomaly-detector:latest
```

## <a name="how-to-use-the-container"></a>Использование контейнера

После размещения контейнера на [главном компьютере](#the-host-computer) воспользуйтесь следующей процедурой для работы с ним.

1. [Запустите контейнер](#run-the-container-with-docker-run) с необходимыми настройками выставления счетов. Доступны дополнительные [примеры](anomaly-detector-container-configuration.md#example-docker-run-commands) команды `docker run`.
1. [Запросите конечную точку прогнозирования контейнера](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Запуск контейнера с помощью команды `docker run`

Воспользуйтесь командой [docker run](https://docs.docker.com/engine/reference/commandline/run/) для запуска контейнера. Дополнительные сведения о том, как получить значения и, см. в разделе [сбор обязательных параметров](#gathering-required-parameters) `{ENDPOINT_URI}` `{API_KEY}` .

Доступны [примеры](anomaly-detector-container-configuration.md#example-docker-run-commands) `docker run` команд.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Запускает контейнер детектора аномалий из образа контейнера
* выделяет одно ядро ЦП и 4 ГБ памяти;
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится.  Дополнительные сведения см. в [разделе о выставлении счетов](#billing).

### <a name="running-multiple-containers-on-the-same-host"></a>Запуск нескольких контейнеров на одном узле

Если вы планируете запускать несколько контейнеров при открытых портах, обязательно назначьте каждому контейнеру отдельный порт. Например, запускайте первый контейнер на порте 5000, а второй — на порте 5001.

Замените `<container-registry>` и `<container-name>` значениями используемых контейнеров. При этом можно использовать разные контейнеры. Контейнер детектора аномалий и контейнер LUIS, выполняющиеся на узле, можно использовать совместно, либо можно использовать несколько контейнеров обнаружения аномалий.

Выполните первый контейнер на порте 5000.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Запустите второй контейнер на порте 5001.


```bash
docker run --rm -it -p 5000:5001 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Каждому последующему контейнеру должен быть назначен отдельный порт.

## <a name="query-the-containers-prediction-endpoint"></a>Запрос конечной точки прогнозирования контейнера

Контейнер предоставляет интерфейсы REST API конечной точки прогнозирования запросов.

Используйте узел http://localhost:5000 для API контейнера.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Остановка контейнера

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Устранение неполадок

Если контейнер запускается с выходным [подключением](anomaly-detector-container-configuration.md#mount-settings) и включенным ведением журнала, контейнер создает файлы журнала, которые удобно использовать для устранения неполадок, возникающих во время запуска или работы контейнера.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Выставление счетов

Контейнеры детекторов аномалий отправляют сведения о выставлении счетов в Azure с помощью ресурса _детектора аномалий_ в учетной записи Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Дополнительные сведения об этих параметрах см. в статье [Настройка контейнеров](anomaly-detector-container-configuration.md).

## <a name="summary"></a>Итоги

В этой статье вы узнали основные понятия и рабочий процесс по скачиванию, установке и запуску контейнеров детекторов аномалий. В разделе "Сводка" сделайте следующее.

* Детектор аномалий предоставляет один контейнер Linux для DOCKER, инкапсуляцию обнаружения аномалий с помощью пакетной и потоковой передачи, ожидаемого определения диапазона и настройки чувствительности.
* Образы контейнеров загружаются из закрытого реестра контейнеров Azure, выделенного для контейнеров.
* Образы контейнеров выполняются в Docker.
* Вы можете использовать REST API или пакет SDK для вызова операций в контейнерах детектора аномалий, указав универсальный код ресурса (URI) узла контейнера.
* При создании экземпляра контейнера нужно указать данные для выставления счетов.

> [!IMPORTANT]
> Контейнеры Cognitive Services не лицензируются для запуска без подключения к Azure для отслеживания использования. Клиенты должны разрешить контейнерам непрерывную передачу данных для выставления счетов в службу контроля потребления. Cognitive Services контейнеры не отправляют данные клиента (например, анализируемые данные временных рядов) в корпорацию Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей о [конфигурации контейнеров](anomaly-detector-container-configuration.md).
* [Развертывание контейнера детекторов аномалий в службе "экземпляры контейнеров Azure"](how-to/deploy-anomaly-detection-on-container-instances.md)
* [Дополнительные сведения о службе API детектора аномалий](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)