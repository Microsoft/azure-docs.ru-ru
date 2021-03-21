---
title: Установка и запуск контейнера для распознавателя форм
titleSuffix: Azure Cognitive Services
description: В этой статье объясняется, как использовать контейнер распознавателя форм Azure Cognitive Services для анализа данных формы и таблицы.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 1be8afb58b22435f4f43b2d6884332a38b7f1e11
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103467482"
---
# <a name="install-and-run-form-recognizer-containers-retiring"></a>Установка и запуск контейнеров распознавателей форм (снятие с учета)

[!INCLUDE [Form Recognizer containers limit](includes/container-limit.md)]

Распознаватель документов Azure применяет технологии машинного обучения, чтобы идентифицировать и извлечь пары "ключ — значение" и таблицы из форм. Он связывает значения и записи в таблицах с парами "ключ — значение", а затем выводит структурированные данные, которые включают в себя связи в исходном файле. 

Вы можете вызывать пользовательскую модель Распознавателя документов с помощью простого REST API, чтобы упростить и без труда интегрировать ее в свой рабочий процесс автоматизации или другое приложение. Требуется только пять документов форм, что позволяет быстро и точно получать результаты и адаптироваться к конкретному содержимому. При этом не требуется выполнять трудоемкие операции вручную или хорошо разбираться в обработке и анализе данных. В процессе также не предусмотрено добавление меток данных или заметок к данным.

| Компонент | Компоненты |
|----------|----------|
| Распознаватель документов | <li>Обрабатывает файлы PDF, PNG и JPG<li>Обучение пользовательских моделей как минимум из пяти форм одинакового макета <li>Извлекает пары "ключ — значение" и сведения о таблице <li>Использует функцию "Распознавания текста" API Компьютерного зрения Cognitive Service для обнаружения и извлечения печатного текста из изображений в формах<li>Не требует добавления заметок или меток |

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем использовать контейнеры Распознавателя документов, необходимо выполнить следующие условия:

| Обязательно | Назначение |
|----------|---------|
| Модуль Docker | На [главном компьютере](#the-host-computer) должен быть установлен модуль Docker. Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).<br><br> Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure. <br><br> В ОС Windows для Docker нужно также настроить поддержку контейнеров Linux.<br><br> |
| Опыт работы с Docker | Требуется базовое представление о понятиях Docker, таких как реестры, репозитории, контейнеры и образы контейнеров, а также знание основных команд `docker`. |
| Azure CLI | Установите в узле [Azure CLI](/cli/azure/install-azure-cli). |
| Ресурс API Компьютерного зрения | Для обработки отсканированных документов и изображений требуется API Компьютерного зрения. Вы можете получить доступ к функции Распознавание текста как к ресурсу Azure (REST API или пакету SDK) или к [контейнеру](../Computer-vision/computer-vision-how-to-install-containers.md#get-the-container-image-with-docker-pull)" *Распознавание-служба-текст* ". Взымается обычная плата. <br><br>Передайте ключ API и конечные точки для ресурса Компьютерное зрение (облако Azure или контейнер Cognitive Services). Используйте этот ключ API и конечную точку как **{COMPUTER_VISION_API_KEY}** и **{COMPUTER_VISION_ENDPOINT_URI}**.<br><br> Если вы используете контейнер *cognitive-services-recognize-text*, проверьте следующее:<br><br>Ваш ключ Компьютерного зрения для контейнера Распознавателя документов — это ключ, указанный в команде `docker run` Компьютерного зрения для контейнера *cognitive-services-recognize-text*.<br>Ваша конечная точка выставления счетов — это конечная точка контейнера, например `http://localhost:5000`. При совместном использовании контейнеров Компьютерного зрения и Распознавателя документов в одному узле их нельзя одновременно запустить с помощью порта *5000*, заданного по умолчанию. |
| Ресурс Распознавателя документов | Для использования контейнеров необходимо следующее:<br><br>Ресурс **распознавателя форм** Azure для получения соответствующего ключа API и URI конечной точки. Оба значения доступны на странице Обзор **распознавателя форм** портал Azure и на страницах ключей, и оба значения необходимы для запуска контейнера.<br><br>**{FORM_RECOGNIZER_API_KEY}**: один из двух доступных ключей ресурсов на странице "ключи"<br><br>**{FORM_RECOGNIZER_ENDPOINT_URI}**: конечная точка, указанная на странице обзора |

> [!NOTE]
> Имя ресурса Компьютерное зрение должно быть одним словом, без дефиса `-` или другими специальными символами. Это ограничение позволяет обеспечить распознаватель форм и Распознавание текста совместимость с контейнерами.

## <a name="gathering-required-parameters"></a>Сбор обязательных параметров

Есть три основных параметра для всех обязательных контейнеров Cognitive Services ". Лицензионное соглашение должно быть установлено со значением `accept` . Кроме того, вам понадобятся URL-адрес конечной точки и ключ API.

### <a name="endpoint-uri-computer_vision_endpoint_uri-and-form_recognizer_endpoint_uri"></a>URI конечной точки `{COMPUTER_VISION_ENDPOINT_URI}` и `{FORM_RECOGNIZER_ENDPOINT_URI}`

Значение URI **конечной точки** доступно на странице *обзора* портал Azure соответствующего ресурса службы. Перейдите на страницу *обзора* , наведите указатель мыши на конечную точку, и `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span> появится значок. Копирование и использование при необходимости.

![Сбор URI конечной точки для последующего использования](../containers/media/overview-endpoint-uri.png)

### <a name="keys-computer_vision_api_key-and-form_recognizer_api_key"></a>Ключи `{COMPUTER_VISION_API_KEY}` и `{FORM_RECOGNIZER_API_KEY}`

Этот ключ используется для запуска контейнера и доступен на странице ключей портал Azure соответствующего ресурса службы. Перейдите на страницу *ключи* и щелкните `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span> значок.

![Получите один из двух ключей для последующего использования](../containers/media/keys-copy-api-key.png)

> [!IMPORTANT]
> Эти ключи подписки используются для доступа к API Cognitive Service. Не предоставляйте доступ к ключам другим пользователям. Храните их безопасно, например, с помощью Azure Key Vault. Также рекомендуется регулярно повторно создавать эти ключи. Для вызова API необходим только один ключ. При повторном создании первого ключа второй ключ можно использовать для бесперебойного доступа к службе.

## <a name="the-host-computer"></a>Главный компьютер

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Требования к контейнеру и рекомендации

В приведенной ниже таблице указаны минимальное и рекомендуемое число ядер ЦП и памяти для каждого контейнера Распознавателя документов.

| Контейнер | Минимальные | Рекомендуемая |
|-----------|---------|-------------|
| Распознаватель документов | 2 ядра, 4 ГБ памяти | 4 ядра, 8 ГБ памяти |
| Распознавание текста | 1 ядро, 8 ГБ памяти | 2 ядра, 8 ГБ памяти |

* Частота каждого ядра должна быть минимум 2,6 ГГц.
* Ядро и память соответствуют параметрам `--cpus` и `--memory`, которые используются как часть команды `docker run`.

> [!Note]
> Минимальные и рекомендуемые значения основаны на ограничениях Docker, *а не на* ресурсах узла.

Вам понадобятся как средство распознавания форм, так и контейнеры Распознавание текста. Обратите внимание, что контейнер **Распознавание текста** [подробный за пределами этой статьи.](../Computer-vision/computer-vision-how-to-install-containers.md#get-the-container-image-with-docker-pull)

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

## <a name="how-to-use-the-container"></a>Использование контейнера

После размещения контейнера на [главном компьютере](#the-host-computer) выполните приведенные ниже инструкции для работы с ним.

1. [Запустите контейнер](#run-the-container-by-using-the-docker-run-command) с необходимыми настройками выставления счетов. Доступны дополнительные [примеры](form-recognizer-container-configuration.md#example-docker-run-commands) команды `docker run`.
1. [Запросите конечную точку прогнозирования контейнера](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-by-using-the-docker-run-command"></a>Команда docker run, выполняющая запуск контейнера

Воспользуйтесь командой [docker run](https://docs.docker.com/engine/reference/commandline/run/) для запуска контейнера. Дополнительные сведения о получении значений, и см. в разделе [сбор обязательных параметров](#gathering-required-parameters) `{COMPUTER_VISION_ENDPOINT_URI}` `{COMPUTER_VISION_API_KEY}` `{FORM_RECOGNIZER_ENDPOINT_URI}` `{FORM_RECOGNIZER_API_KEY}` .

Доступны [примеры](form-recognizer-container-configuration.md#example-docker-run-commands) `docker run` команд.

### <a name="form-recognizer"></a>Распознаватель документов

> [!NOTE]
> `--mount`В этих примерах используются каталоги с путями к каталогам Windows. Если вы используете Linux или macOS, измените параметр для своей среды. 

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

Команда:

* запускает контейнер Распознавателя документов из образа контейнера;
* выделяет 2 ядра ЦП и 8 гигабайт (ГБ) памяти;
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.
* подключает тома /input и /output к контейнеру.

[!INCLUDE [Running multiple containers on the same host H2](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

### <a name="run-separate-containers-as-separate-docker-run-commands"></a>Запуск отдельных контейнеров в качестве отдельных команд docker run

Для совместного локального размещения Распознавателя документов и Распознавателя текста в одном узле используйте два следующих примера команд CLI Docker:

Выполните первый контейнер на порте 5000. 

```bash 
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY}
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

Запустите второй контейнер на порте 5001.

```bash 
docker run --rm -it -p 5001:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text \
Eula=accept \
Billing={COMPUTER_VISION_ENDPOINT_URI} \
ApiKey={COMPUTER_VISION_API_KEY}
```
Каждому последующему контейнеру должен быть назначен отдельный порт. 

### <a name="run-separate-containers-with-docker-compose"></a>Запуск отдельных контейнеров с помощью Docker Compose

Для совместного локального размещения Распознавателя документов и Распознавателя текста в одном узле используйте следующий пример YAML-файла Docker Compose: Распознаватель текста `{COMPUTER_VISION_API_KEY}` должен быть одинаковым для обоих контейнеров: `formrecognizer` и `ocr`. `{COMPUTER_VISION_ENDPOINT_URI}` используется только в контейнере `ocr`, так как контейнер `formrecognizer` использует имя и порт `ocr`. 

```docker
version: '3.3'
services:   
  ocr:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{COMPUTER_VISION_ENDPOINT_URI}"
      apikey: "{COMPUTER_VISION_API_KEY}"

  formrecognizer:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{FORM_RECOGNIZER_ENDPOINT_URI}"
      apikey: "{FORM_RECOGNIZER_API_KEY}"
      FormRecognizer__ComputerVisionApiKey: {COMPUTER_VISION_API_KEY}
      FormRecognizer__ComputerVisionEndpointUri: "http://ocr:5000"
      FormRecognizer__SyncProcessTaskCancelLimitInSecs: 75
    links:
      - ocr
    volumes:
      - type: bind
        source: c:\output
        target: /output
      - type: bind
        source: c:\input
        target: /input
    ports:
      - "5000:5000"
```

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`, а также параметры `FormRecognizer:ComputerVisionApiKey` и `FormRecognizer:ComputerVisionEndpointUri`. Иначе контейнер не запустится. Дополнительные сведения см. в [разделе о выставлении счетов](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Запрос конечной точки прогнозирования контейнера

|Контейнер|Конечная точка|
|--|--|
|form-recognizer|http://localhost:5000

### <a name="form-recognizer"></a>Распознаватель документов

Контейнер предоставляет API конечных точек запроса на основе веб-сокетов, доступ к которым осуществляется с помощью [документации по пакету SDK для служб Распознавателя документов](./index.yml).

По умолчанию пакет SDK Распознавателя документов использует веб-службы. Чтобы использовать контейнер, вам необходимо изменить метод инициализации. См. следующие примеры.

#### <a name="for-c"></a>Для C#

Смените этот вызов инициализации облака Azure:

```csharp
var config =
    FormRecognizerConfig.FromSubscription(
        "YourSubscriptionKey",
        "YourServiceRegion");
```
на этот вызов, в котором используется конечная точка контейнера:

```csharp
var config =
    FormRecognizerConfig.FromEndpoint(
        "ws://localhost:5000/formrecognizer/v1.0-preview/custom",
        "YourSubscriptionKey");
```

#### <a name="for-python"></a>Для Python

Смените этот вызов инициализации облака Azure:

```python
formrecognizer_config =
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key, region=service_region)
```

на этот вызов, в котором используется конечная точка контейнера:

```python
formrecognizer_config = 
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key,
        endpoint="ws://localhost:5000/formrecognizer/v1.0-preview/custom"
```

### <a name="form-recognizer"></a>Распознаватель документов

Контейнер предоставляет интерфейсы API конечной точки RESTFUL, которые можно найти на странице справки [API распознавателя форм] https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeWithCustomForm) .


[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]


## <a name="stop-the-container"></a>Остановка контейнера

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Устранение неполадок

Если контейнер запускается с выходным [подключением](form-recognizer-container-configuration.md#mount-settings) и включенным ведением журнала, контейнер создает файлы журнала, которые удобно использовать для устранения неполадок, возникающих во время запуска или работы контейнера.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Выставление счетов

Контейнеры Распознавателя документов отправляют данные для выставления счетов в Azure с помощью ресурса _Распознавателя документов_ в учетной записи Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Дополнительные сведения об этих параметрах см. в статье [Настройка контейнеров](form-recognizer-container-configuration.md).

## <a name="summary"></a>Итоги

В этой статье вы узнали основные понятия и рабочий процесс для скачивания, установки и выполнения контейнеров Распознавателя документов. В разделе "Сводка" сделайте следующее.

* Распознаватель документов предоставляет один контейнер Linux для Docker.
* Образы контейнеров скачиваются из частного реестра контейнеров в Azure.
* Образы контейнеров выполняются в Docker.
* Указав узел URI контейнера, вы можете вызвать операции в контейнерах Распознавателя документов с помощью пакета SDK REST или REST API.
* При создании экземпляра контейнера нужно указать данные для выставления счетов.

> [!IMPORTANT]
>  Контейнеры Cognitive Services не лицензируются для запуска без подключения к Azure для отслеживания использования. Клиенты должны разрешить контейнерам непрерывную передачу данных для выставления счетов в службу контроля потребления. Контейнеры Cognitive Services не отправляют в корпорацию Майкрософт данные клиента (например анализируемые изображения или тексты).

## <a name="next-steps"></a>Следующие шаги

* Проверьте настройки [контейнеров](form-recognizer-container-configuration.md) на наличие параметров конфигурации.
* Используйте больше [контейнеров Cognitive Services](../cognitive-services-container-support.md).