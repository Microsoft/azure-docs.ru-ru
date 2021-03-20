---
title: Настройка контейнеров — лицо
titleSuffix: Azure Cognitive Services
description: Среда выполнения контейнера Распознавания лиц настраивается с помощью аргументов команды `docker run`. Существуют и обязательные, и необязательные параметры.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 2f608843e27b79d02697df8e2a7f2aba6695e10a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80878431"
---
# <a name="configure-face-docker-containers"></a>Настройка контейнеров Docker распознавания лиц

Среда выполнения контейнера **Распознавания лиц** настраивается с помощью аргументов команды `docker run`. Контейнер поддерживает несколько обязательных и несколько необязательных параметров. Доступны несколько [примеров](#example-docker-run-commands) этой команды. Для конкретного контейнера настраиваются входные параметры выставления счетов. 

## <a name="configuration-settings"></a>Параметры конфигурации

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> [`ApiKey`](#apikey-configuration-setting)Параметры, [`Billing`](#billing-configuration-setting) и [`Eula`](#eula-setting) используются совместно, и необходимо указать допустимые значения для всех трех из них; в противном случае контейнер не будет запущен. Дополнительные сведения об использовании этих параметров конфигурации для создания экземпляра контейнера см. в разделе [Выставление счетов](face-how-to-install-containers.md#billing).

## <a name="apikey-configuration-setting"></a>Параметр конфигурации ApiKey

Параметр `ApiKey` определяет ключ ресурса Azure, который используется для отслеживания данных для выставления счетов для контейнера. Необходимо указать значение для параметра ApiKey, а значение должно быть допустимым ключом для ресурса _Cognitive Services_ , указанного для [`Billing`](#billing-configuration-setting) параметра конфигурации.

Этот параметр можно найти в следующем месте.

* Портал Azure: управление ресурсами **Cognitive Services** в разделе **ключи**

## <a name="applicationinsights-setting"></a>Параметр ApplicationInsights.

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Параметр конфигурации выставления счетов

`Billing`Параметр указывает URI конечной точки ресурса _Cognitive Services_ в Azure, который используется для отслеживания сведений о выставлении счетов для контейнера. Необходимо указать значение для этого параметра конфигурации, а значение должно быть допустимым URI конечной точки для ресурса _Cognitive Services_ в Azure. Отчеты об использовании контейнера примерно каждые 10—15 минут.

Этот параметр можно найти в следующем месте.

* Портал Azure: **Cognitive Services** обзор с меткой `Endpoint`

Не забудьте добавить в универсальный код ресурса (URI) конечной точки маршрутизацию _лиц_ , как показано в примере. 

|Обязательно| Имя | Тип данных | Описание |
|--|------|-----------|-------------|
|Да| `Billing` | Строка | URI конечной точки выставления счетов. Дополнительные сведения о получении URI выставления счетов см. в разделе [сбор обязательных параметров](face-how-to-install-containers.md#gathering-required-parameters). Дополнительные сведения и полный список региональных конечных точек см. в статье [Custom subdomain names for Cognitive Services](../cognitive-services-custom-subdomains.md) (Пользовательские имена поддоменов для Cognitive Services). |

<!-- specific to face only -->

## <a name="cloudai-configuration-settings"></a>Параметры конфигурации CloudAI

В разделе `CloudAI` представлены специальные параметры, уникальные для вашего контейнера. Для контейнера распознавания лиц в разделе `CloudAI` поддерживаются следующие параметры и объекты:

| Имя | Тип данных | Описание |
|------|-----------|-------------|
| `Storage` | Объект | Сценарий хранения, используемый контейнером распознавания лиц. Дополнительные сведения о сценариях хранения и соответствующих параметрах объекта `Storage` см. в разделе [Параметры сценариев хранения](#storage-scenario-settings) |

### <a name="storage-scenario-settings"></a>Параметры сценариев хранения

В контейнере распознавания лиц хранятся большие двоичные объекты, кэш, метаданные и очереди, в зависимости от того, какие данные хранятся. Например, результаты и индексы обучения для большой группы людей хранятся в виде больших двоичных объектов. Контейнер распознавания лиц предоставляет два разных сценария хранения при взаимодействии с данными указанных ниже типов и их хранения.

* Память  
  В памяти хранятся данные всех четырех типов. Они не распределяются и не являются постоянными. При остановке или удалении контейнера распознавания лиц ликвидируются все данные в хранилище для этого контейнера.  
  Это стандартный сценарий хранения для контейнера распознавания лиц.
* Azure  
  Контейнер распознавания лиц использует службу хранилища Azure и Azure Cosmos DB для распределения данных этих четырех типов в постоянном хранилище. Большие двоичные данные и очереди данных обрабатывает служба хранилища Azure. Метаданные и данные кэша обрабатывает служба Azure Cosmos DB. При остановке или удалении контейнера распознавания лиц все данные в хранилище для этого контейнера остаются в службе хранилища Azure и Azure Cosmos DB.  
  К ресурсам, используемым в сценариях хранения Azure, применяются перечисленные ниже дополнительные требования.
  * Ресурс службы хранилища Azure должен использовать тип учетной записи StorageV2.
  * Ресурс Azure Cosmos DB должен использовать API Azure Cosmos DB для MongoDB.

Сценариями хранения и соответствующими параметрами конфигурации управляет объект `Storage` в разделе конфигурации `CloudAI`. В объекте `Storage` доступны следующие параметры конфигурации:

| Имя | Тип данных | Описание |
|------|-----------|-------------|
| `StorageScenario` | Строка | Сценарий хранения, поддерживаемый контейнером. Доступны следующие значения:<br/>`Memory` — значение по умолчанию. Контейнер использует непостоянное, нераспределенное и размещенное в памяти хранилище для временного использования в одном узле. При остановке или удалении контейнера ликвидируется соответствующее хранилище.<br/>`Azure` — контейнер использует ресурсы Azure для хранения. При остановке или удалении контейнера соответствующее хранилище сохраняется.|
| `ConnectionStringOfAzureStorage` | Строка | Строка подключения для ресурса службы хранилища Azure, используемого контейнером.<br/>Этот параметр применяется только в том случае, если для параметра `StorageScenario` задано значение `Azure`. |
| `ConnectionStringOfCosmosMongo` | Строка | Строка подключения MongoDB для ресурса Azure Cosmos, используемого контейнером.<br/>Этот параметр применяется только в том случае, если для параметра `StorageScenario` задано значение `Azure`. |

Например, приведенная ниже команда указывает сценарий службы хранилища Azure и предоставляет примеры строк подключения для ресурсов службы хранилища Azure и Cosmos DB, которые используются при хранении данных для контейнера распознавания лиц.

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 containerpreview.azurecr.io/microsoft/cognitive-services-face Eula=accept Billing=https://westcentralus.api.cognitive.microsoft.com/face/v1.0 ApiKey=0123456789 CloudAI:Storage:StorageScenario=Azure CloudAI:Storage:ConnectionStringOfCosmosMongo="mongodb://samplecosmosdb:0123456789@samplecosmosdb.documents.azure.com:10255/?ssl=true&replicaSet=globaldb" CloudAI:Storage:ConnectionStringOfAzureStorage="DefaultEndpointsProtocol=https;AccountName=sampleazurestorage;AccountKey=0123456789;EndpointSuffix=core.windows.net"
  ```

Сценарий хранения обрабатывается отдельно от входных и выходных подключений. Вы можете указать сочетание этих функций для одного контейнера. Например, приведенная ниже команда определяет подключение привязки Docker к папке `D:\Output` на хост-компьютере в выходном подключении, а затем создает экземпляр контейнера распознавания лиц из образа, сохраняя файлы журнала в формате JSON во внешнем подключении. Эта команда также указывает сценарий службы хранилища Azure и предоставляет примеры строк подключения для ресурсов службы хранилища Azure и Cosmos DB, которые используются при хранении данных для контейнера распознавания лиц.

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 --mount type=bind,source=D:\Output,destination=/output containerpreview.azurecr.io/microsoft/cognitive-services-face Eula=accept Billing=https://westcentralus.api.cognitive.microsoft.com/face/v1.0 ApiKey=0123456789 Logging:Disk:Format=json CloudAI:Storage:StorageScenario=Azure CloudAI:Storage:ConnectionStringOfCosmosMongo="mongodb://samplecosmosdb:0123456789@samplecosmosdb.documents.azure.com:10255/?ssl=true&replicaSet=globaldb" CloudAI:Storage:ConnectionStringOfAzureStorage="DefaultEndpointsProtocol=https;AccountName=sampleazurestorage;AccountKey=0123456789;EndpointSuffix=core.windows.net"
  ```

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

Контейнеры распознавания лиц не используют входные или выходные подключения для хранения учебных данных или данных службы. 

Точный синтаксис расположения подключения к узлу зависит от операционной системы узла. Кроме того, расположение подключения [главного компьютера](face-how-to-install-containers.md#the-host-computer)может быть недоступно из-за конфликта между разрешениями, используемыми учетной записью службы DOCKER, и разрешениями на расположение для подключения к узлу. 

|Необязательно| Имя | Тип данных | Описание |
|-------|------|-----------|-------------|
|Нельзя использовать| `Input` | Строка | Контейнеры распознавания лиц не используют этот элемент.|
|Необязательный| `Output` | Строка | Цель выходного подключения. Значение по умолчанию — `/output`. Это расположение файлов журналов. Сюда входят журналы контейнера. <br><br>Пример<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Примеры команд docker run 

В следующих примерах параметры конфигурации иллюстрируют процесс написания и использования команд `docker run`.  После запуска контейнер продолжает работу, пока вы его не [остановите](face-how-to-install-containers.md#stop-the-container).

* **Символ продолжения строки**: команды DOCKER, приведенные в следующих разделах, используют обратную косую черту в `\` качестве символа продолжения строки. Замените или удалите ее в соответствии с требованиями вашей операционной системы. 
* **Порядок аргументов**. не изменяйте порядок аргументов, если вы не знакомы с контейнерами DOCKER.

Замените строку {_имя_аргумента_} собственными значениями.

| Заполнитель | Значение | Формат или пример |
|-------------|-------|---|
| **{API_KEY}** | Ключ конечной точки `Face` ресурса на `Face` странице ключей Azure. | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
| **{ENDPOINT_URI}** | Значение конечной точки выставления счетов доступно на `Face` странице обзора Azure.| См. раздел [сбор обязательных параметров](face-how-to-install-containers.md#gathering-required-parameters) для явных примеров. |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится.  Дополнительные сведения см. в [разделе о выставлении счетов](face-how-to-install-containers.md#billing).
> Значение ApiKey является **ключом** на `Cognitive Services` странице ключей ресурсов Azure. 

## <a name="face-container-docker-examples"></a>Примеры контейнера Docker распознавания лиц

Следующие примеры Docker предназначены для контейнера распознавания лиц. 

### <a name="basic-example"></a>Простой пример 

  ```
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  containerpreview.azurecr.io/microsoft/cognitive-services-face \
  Eula=accept \
  Billing={ENDPOINT_URI} \
  ApiKey={API_KEY} 
  ```

### <a name="logging-example"></a>Пример ведения журнала 

  ```
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 containerpreview.azurecr.io/microsoft/cognitive-services-face \
  Eula=accept \
  Billing={ENDPOINT_URI} ApiKey={API_KEY} \
  Logging:Console:LogLevel:Default=Information
  ```

## <a name="next-steps"></a>Дальнейшие действия

* Изучите статью об [установке и запуске контейнеров](face-how-to-install-containers.md).
