---
title: Параметры контейнера DOCKER — LUIS
titleSuffix: Azure Cognitive Services
description: Среда выполнения контейнера LUIS настраивается с помощью аргументов команды `docker run`. LUIS поддерживает несколько обязательных и несколько необязательных параметров.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: a4f2b07edc6c290fa030621a4dc400ab50890bba
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96001204"
---
# <a name="configure-language-understanding-docker-containers"></a>Настройка контейнеров Docker Интеллектуальной службы распознавания речи 

Среда выполнения контейнера **Распознавание речи** (Luis) настраивается с помощью `docker run` аргументов команды. LUIS поддерживает несколько обязательных и несколько необязательных параметров. Доступны несколько [примеров](#example-docker-run-commands) этой команды. Для конкретного контейнера настраиваются входные [параметры подключения](#mount-settings) и параметры выставления счетов. 

## <a name="configuration-settings"></a>Параметры конфигурации

К контейнеру применяются следующие параметры конфигурации.

|Обязательно|Параметр|Назначение|
|--|--|--|
|Да|[ApiKey](#apikey-setting)|Используется для отслеживания данных для выставлении счетов.|
|Нет|[ApplicationInsights](#applicationinsights-setting)|Позволяет добавить в контейнер поддержку телеметрии [Azure Application Insights](/azure/application-insights).|
|Да|[Выставление счетов](#billing-setting)|Задает URI конечной точки для ресурса службы в Azure.|
|Да|[Лицензионное соглашение](#eula-setting)| Указывает, что вы приняли условия лицензии для контейнера.|
|Нет|[Fluentd](#fluentd-settings)|Записывает данные в журнал и (необязательно) передает метрики на сервер Fluentd.|
|Нет|[Прокси-сервер HTTP](#http-proxy-credentials-settings)|Настраивает прокси-сервер HTTP для исходящих запросов.|
|Нет|[Logging](#logging-settings)|Обеспечивает поддержку ведения журнала ASP.NET Core для вашего контейнера. |
|Да|[Подключения](#mount-settings)|Читает и записывает данные с главного компьютера в контейнер и обратно.|

> [!IMPORTANT]
> [`ApiKey`](#apikey-setting)Параметры, [`Billing`](#billing-setting) и [`Eula`](#eula-setting) используются совместно, и необходимо указать допустимые значения для всех трех из них; в противном случае контейнер не будет запущен. Дополнительные сведения об использовании этих параметров конфигурации для создания экземпляра контейнера см. в разделе [Выставление счетов](luis-container-howto.md#billing).

## <a name="apikey-setting"></a>Параметр ApiKey

Параметр `ApiKey` определяет ключ ресурса Azure, который используется для отслеживания данных для выставления счетов для контейнера. Необходимо указать значение для параметра ApiKey, а значение должно быть допустимым ключом для ресурса _Cognitive Services_ , указанного для [`Billing`](#billing-setting) параметра конфигурации.

Этот параметр можно найти в следующих местах.

* Портал Azure: управление ресурсами **Cognitive Services** в разделе **ключи**
* Портал LUIS: страница **параметров ключей и конечных точек** . 

Не используйте начальный ключ или ключ разработки. 

## <a name="applicationinsights-setting"></a>Параметр ApplicationInsights.

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-setting"></a>Настройка выставления счетов

`Billing`Параметр указывает URI конечной точки ресурса _Cognitive Services_ в Azure, который используется для отслеживания сведений о выставлении счетов для контейнера. Необходимо указать значение для этого параметра конфигурации, а значение должно быть допустимым URI конечной точки для ресурса _Cognitive Services_ в Azure. Отчеты об использовании контейнера примерно каждые 10—15 минут.

Этот параметр можно найти в следующих местах.

* Портал Azure: **Cognitive Services** обзор с меткой `Endpoint`
* Портал LUIS. Страница **параметров ключей и конечных точек** в универсальном коде ресурса (URI) конечной точки.

| Обязательно | Имя | Тип данных | Описание |
|----------|------|-----------|-------------|
| Да      | `Billing` | строка | URI конечной точки выставления счетов. Дополнительные сведения о получении URI выставления счетов см. в разделе [сбор обязательных параметров](luis-container-howto.md#gathering-required-parameters). Дополнительные сведения и полный список региональных конечных точек см. в статье [Custom subdomain names for Cognitive Services](../cognitive-services-custom-subdomains.md) (Пользовательские имена поддоменов для Cognitive Services). |

## <a name="eula-setting"></a>Параметр Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Параметры Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Параметры учетных данных прокси-сервера HTTP

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Параметры ведения журнала
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Параметры подключения

Используйте подключения привязок для чтения данных из контейнера и записи в него. Вы можете указать входное или выходное подключение, указав параметр `--mount` в команде [docker run](https://docs.docker.com/engine/reference/commandline/run/). 

Контейнер LUIS не использует входные или выходные подключения для хранения учебных данных или данных службы. 

Точный синтаксис расположения подключения к узлу зависит от операционной системы узла. Кроме того,расположение подключения на [главном компьютере](luis-container-howto.md#the-host-computer) может оказаться недоступным из-за конфликта между разрешениями для учетной записи службы Docker и расположения подключения к узлу. 

В следующей таблице описаны поддерживаемые параметры.

|Обязательно| Имя | Тип данных | Описание |
|-------|------|-----------|-------------|
|Да| `Input` | Строка | Цель входного подключения. Значение по умолчанию — `/input`. Это расположение файлов из пакета LUIS. <br><br>Пример<br>`--mount type=bind,src=c:\input,target=/input`|
|нет| `Output` | Строка | Цель выходного подключения. Значение по умолчанию — `/output`. Это расположение файлов журналов. Сюда относятся журналы запросов LUIS и журналы контейнера. <br><br>Пример<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Примеры команд docker run

В следующих примерах параметры конфигурации иллюстрируют процесс написания и использования команд `docker run`.  После запуска контейнер продолжает работу, пока вы его не [остановите](luis-container-howto.md#stop-the-container).

* В этих примерах используется каталог на `C:` диске, чтобы избежать конфликтов разрешений в Windows. Если вам нужен конкретный каталог для входных данных, может потребоваться предоставить соответствующие разрешения службе Docker. 
* Не изменяйте порядок аргументов, если вы не являетесь уверенным пользователем контейнеров Docker.
* Если используется другая операционная система, используйте правильную консоль или терминал, синтаксис папок для подключений и символ продолжения строки для вашей системы. В этих примерах предполагается наличие в консоли Windows символа продолжения строки `^` . Так как контейнер является операционной системой Linux, целевое подключение использует синтаксис папки в формате Linux.

Замените строку {_имя_аргумента_} собственными значениями.

| Заполнитель | Значение | Формат или пример |
|-------------|-------|---|
| **{API_KEY}** | Ключ конечной точки `LUIS` ресурса на `LUIS` странице ключей Azure. | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
| **{ENDPOINT_URI}** | Значение конечной точки выставления счетов доступно на `LUIS` странице обзора Azure.| См. раздел [сбор обязательных параметров](luis-container-howto.md#gathering-required-parameters) для явных примеров. |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится. Дополнительные сведения см. в [разделе о выставлении счетов](luis-container-howto.md#billing).
> Значение ApiKey — это **ключ** на странице "ключи и конечные точки" на портале Luis. Он также доступен на `Cognitive Services` странице "ключи ресурсов Azure". 

### <a name="basic-example"></a>Простой пример

В следующем примере указано минимальное число аргументов, с которым можно запустить контейнер.

```console
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output,target=/output ^
mcr.microsoft.com/azure-cognitive-services/luis:latest ^
Eula=accept ^
Billing={ENDPOINT_URL} ^
ApiKey={API_KEY}
```

### <a name="applicationinsights-example"></a>Пример ApplicationInsights

В следующем примере задается аргумент ApplicationInsights для отправки данных телеметрии в Application Insights во время выполнения контейнера:

```console
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output,target=/output ^
mcr.microsoft.com/azure-cognitive-services/luis:latest ^
Eula=accept ^
Billing={ENDPOINT_URL} ^
ApiKey={API_KEY} ^
InstrumentationKey={INSTRUMENTATION_KEY}
```

### <a name="logging-example"></a>Пример ведения журнала 

Следующая команда задает уровень ведения журнала, `Logging:Console:LogLevel` , чтобы настроить уровень ведения журнала [`Information`](https://msdn.microsoft.com) . 

```console
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output,target=/output ^
mcr.microsoft.com/azure-cognitive-services/luis:latest ^
Eula=accept ^
Billing={ENDPOINT_URL} ^
ApiKey={API_KEY} ^
Logging:Console:LogLevel:Default=Information
```

## <a name="next-steps"></a>Дальнейшие действия

* Изучите статью об [установке и запуске контейнеров](luis-container-howto.md).
* Ознакомьтесь со статьей об [устранение неполадок](troubleshooting.md), чтобы устранить проблемы, связанные с функциональностью LUIS.
* Воспользуйтесь [дополнительными контейнерами Cognitive Services](../cognitive-services-container-support.md)