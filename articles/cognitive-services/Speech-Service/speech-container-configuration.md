---
title: Настройка контейнеров речи
titleSuffix: Azure Cognitive Services
description: Служба речи предоставляет каждому контейнеру общую платформу конфигурации, позволяющую легко настраивать хранилище, вести журнал и данные телеметрии, а также параметры безопасности для контейнеров.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: e65bb7c7d8fc04baec6b50a53519e689e748fbe1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96012175"
---
# <a name="configure-speech-service-containers"></a>Настройка контейнеров службы речи

Речевые контейнеры позволяют клиентам создавать одну архитектуру приложения для распознавания речи, оптимизированную для использования преимуществ как надежных облачных возможностей, так и пограничных локализации. Теперь поддерживаются пять речевых контейнеров: преобразование **речи в текст**, преобразование **речи в текст**, преобразование **текста в** речь, преобразование **нейронных текстов в речь** и преобразование **текста в речь**.

Среда выполнения **речевого** контейнера настраивается с помощью `docker run` аргументов команды. Контейнер поддерживает несколько обязательных и несколько необязательных параметров. Доступны несколько [примеров](#example-docker-run-commands) этой команды. Для конкретного контейнера настраиваются входные параметры выставления счетов.

## <a name="configuration-settings"></a>Параметры конфигурации

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> [`ApiKey`](#apikey-configuration-setting)Параметры, [`Billing`](#billing-configuration-setting) и [`Eula`](#eula-setting) используются совместно, и необходимо указать допустимые значения для всех трех из них; в противном случае контейнер не будет запущен. Дополнительные сведения об использовании этих параметров конфигурации для создания экземпляра контейнера см. в разделе [Выставление счетов](speech-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Параметр конфигурации ApiKey

Параметр `ApiKey` определяет ключ ресурса Azure, который используется для отслеживания данных для выставления счетов для контейнера. Необходимо указать значение для параметра ApiKey, а значение должно быть допустимым ключом для ресурса _речи_ , указанного для [`Billing`](#billing-configuration-setting) параметра конфигурации.

Этот параметр можно найти в следующем месте.

- Портал Azure: управление ресурсами **речи** в разделе **ключи**

## <a name="applicationinsights-setting"></a>Параметр ApplicationInsights.

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Параметр конфигурации выставления счетов

`Billing`Параметр указывает URI конечной точки для ресурса _речи_ в Azure, который используется для сбора сведений о выставлении счетов для контейнера. Необходимо указать значение для этого параметра конфигурации, а значение должно быть допустимым URI конечной точки для ресурса _речи_ в Azure. Отчеты об использовании контейнера примерно каждые 10—15 минут.

Этот параметр можно найти в следующем месте.

- Портал Azure: обзор **речи** , с метками `Endpoint`

| Обязательно | Имя | Тип данных | Описание |
| -------- | ---- | --------- | ----------- |
| Да | `Billing` | Строка | URI конечной точки выставления счетов. Дополнительные сведения о получении URI выставления счетов см. в разделе [сбор обязательных параметров](speech-container-howto.md#gathering-required-parameters). Дополнительные сведения и полный список региональных конечных точек см. в статье [Custom subdomain names for Cognitive Services](../cognitive-services-custom-subdomains.md) (Пользовательские имена поддоменов для Cognitive Services). |

## <a name="eula-setting"></a>Параметр Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Параметры Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Параметры учетных данных прокси-сервера HTTP

[!INCLUDE [Container shared HTTP proxy settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Параметры ведения журнала

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Параметры подключения

Используйте подключения привязок для чтения данных из контейнера и записи в него. Вы можете указать входное или выходное подключение, указав параметр `--mount` в команде [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Стандартные контейнеры распознавания речи не используют входные или выходные подключения для хранения данных для обучения или службы. Однако пользовательские контейнеры речи используют подключения томов.

Точный синтаксис расположения подключения к узлу зависит от операционной системы узла. Кроме того,расположение подключения на [главном компьютере](speech-container-howto.md#the-host-computer) может оказаться недоступным из-за конфликта между разрешениями для учетной записи службы Docker и расположения подключения к узлу.

| Необязательно | Имя | Тип данных | Описание |
| -------- | ---- | --------- | ----------- |
| Нельзя использовать | `Input` | Строка | Стандартные контейнеры распознавания речи этого не используют. Пользовательские контейнеры распознавания речи используют [подключения томов](#volume-mount-settings).                                                                                    |
| Необязательный | `Output` | Строка | Цель выходного подключения. Значение по умолчанию — `/output`. Это расположение файлов журналов. Сюда входят журналы контейнера. <br><br>Пример<br>`--mount type=bind,src=c:\output,target=/output` |

## <a name="volume-mount-settings"></a>Параметры подключения тома

Пользовательские контейнеры распознавания речи используют [подключения томов](https://docs.docker.com/storage/volumes/) для сохранения пользовательских моделей. Можно указать подключение тома, добавив `-v` параметр (или `--volume` ) к команде [DOCKER Run](https://docs.docker.com/engine/reference/commandline/run/) .

Пользовательские модели загружаются при первом запуске новой модели в рамках пользовательской команды DOCKER. Последовательные запуски одного и того же `ModelId` для пользовательского контейнера речевого ввода будут использовать ранее скачанную модель. Если подключение тома не указано, пользовательские модели не могут быть сохранены.

Параметр подключения тома состоит из трех полей, `:` разделенных цветом:

1. Первое поле — это имя тома на хост-компьютере, например _к:\инпут_.
2. Второе поле — это каталог в контейнере, например _/УСР/локал/Моделс_.
3. Третье поле (необязательно) представляет собой список параметров с разделителями-запятыми. Дополнительные сведения см. в разделе [использование томов](https://docs.docker.com/storage/volumes/).

### <a name="volume-mount-example"></a>Пример подключения тома

```bash
-v C:\input:/usr/local/models
```

Эта команда подключает каталог _к:\инпут_ главного компьютера к каталогу Containers _/УСР/локал/Моделс_ .

> [!IMPORTANT]
> Параметры подключения тома применимы только к контейнерам " **пользовательское распознавание речи-текст** " и " **пользовательские преобразования текста в речь** ". В контейнерах преобразования текста **в речь** и **нейронных текстовых подсетей** не  используются подключения томов.

## <a name="example-docker-run-commands"></a>Примеры команд docker run

В следующих примерах параметры конфигурации иллюстрируют процесс написания и использования команд `docker run`. После запуска контейнер продолжает работу, пока вы его не [остановите](speech-container-howto.md#stop-the-container).

- **Символ продолжения строки**: команды DOCKER, приведенные в следующих разделах, используют обратную косую черту в `\` качестве символа продолжения строки. Замените или удалите ее в соответствии с требованиями вашей операционной системы.
- **Порядок аргументов**. не изменяйте порядок аргументов, если вы не знакомы с контейнерами DOCKER.

Замените строку {_имя_аргумента_} собственными значениями.

| Заполнитель | Значение | Формат или пример |
| ----------- | ----- | ----------------- |
| **{API_KEY}** | Ключ конечной точки `Speech` ресурса на `Speech` странице ключей Azure.   | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`                                                                                  |
| **{ENDPOINT_URI}** | Значение конечной точки выставления счетов доступно на `Speech` странице обзора Azure. | См. раздел [сбор обязательных параметров](speech-container-howto.md#gathering-required-parameters) для явных примеров. |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится. Дополнительные сведения см. в [разделе о выставлении счетов](#billing-configuration-setting).
> Значение ApiKey является **ключом** на странице ключей ресурсов речевого ввода Azure.

## <a name="speech-container-docker-examples"></a>Примеры DOCKER контейнеров речи

Следующие примеры DOCKER предназначены для контейнера речи.

## <a name="speech-to-text"></a>[Преобразование речи в текст](#tab/stt)

### <a name="basic-example-for-speech-to-text"></a>Базовый пример преобразования речи в текст

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-to-text"></a>Пример ведения журнала для преобразования речи в текст

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-speech-to-text"></a>[пользовательское преобразование речи в текст;](#tab/cstt)

### <a name="basic-example-for-custom-speech-to-text"></a>Базовый пример для Пользовательское распознавание речи в текст

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-speech-to-text"></a>Пример ведения журнала для Пользовательское распознавание речи в текст

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="text-to-speech"></a>[Преобразование текста в речь](#tab/tss)

### <a name="basic-example-for-text-to-speech"></a>Базовый пример преобразования текста в речь

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-text-to-speech"></a>Пример ведения журнала для преобразования текста в речь

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-text-to-speech"></a>[Пользовательский текст в речь](#tab/ctts)

### <a name="basic-example-for-custom-text-to-speech"></a>Базовый пример пользовательского преобразования текста в речь

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-text-to-speech"></a>Пример ведения журнала для пользовательского преобразования текста в речь

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="neural-text-to-speech"></a>[нейронное преобразование текста в речь;](#tab/ntts)

### <a name="basic-example-for-neural-text-to-speech"></a>Базовый пример преобразования текста нейронной речи в речь

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-neural-text-to-speech"></a>Пример ведения журнала для преобразования текста нейронных текстов в речь

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="speech-language-detection"></a>[Речь распознавание языка](#tab/lid)

### <a name="basic-example-for-speech-language-detection"></a>Простой пример определения языка речи

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-language-detection"></a>Пример ведения журнала для распознавания языка речи

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

---

## <a name="next-steps"></a>Дальнейшие действия

- Изучите статью об [установке и запуске контейнеров](speech-container-howto.md).
