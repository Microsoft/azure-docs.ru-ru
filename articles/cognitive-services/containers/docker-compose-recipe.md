---
title: Развертывание нескольких контейнеров с помощью Docker Compose
titleSuffix: Azure Cognitive Services
description: Узнайте, как развернуть несколько контейнеров Cognitive Services. В этой статье показано, как управлять несколькими образами контейнера DOCKER с помощью Docker Compose.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: aahi
ms.openlocfilehash: cedcf8a3fcd656c4af0ca7493c598791d35d20d9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95996137"
---
# <a name="use-docker-compose-to-deploy-multiple-containers"></a>Развертывание нескольких контейнеров с помощью Docker Compose

В этой статье показано, как развернуть несколько контейнеров Azure Cognitive Services. В частности, вы узнаете, как использовать Docker Compose для координации нескольких образов контейнеров DOCKER.

> [DOCKER Compose](https://docs.docker.com/compose/) — это средство для определения и запуска приложений DOCKER с несколькими контейнерами. В процессе составления для настройки служб приложения используется файл YAML. Затем создайте и запустите все службы из конфигурации, выполнив одну команду.

Может оказаться полезным управлять несколькими образами контейнеров на одном хост-компьютере. В этой статье мы будем объединять контейнеры для чтения и распознавания форм.

## <a name="prerequisites"></a>Предварительные условия

Для этой процедуры требуется несколько средств, которые необходимо установить и запустить локально:

* Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services), прежде чем начать работу.
* [Подсистема DOCKER](https://www.docker.com/products/docker-engine). Убедитесь, что интерфейс командной строки DOCKER работает в окне консоли.
* Ресурс Azure с правильной ценовой категорией. Для этого контейнера работает только Следующая ценовая категория:
  * **Компьютерное зрениеный** ресурс только для ценовой категории F0 или Standard.
  * Ресурс **распознавателя форм** с F0 или "Стандартная ценовая категория".
  * Ресурс **Cognitive Services** с ценовой категорией S0.
* Если вы используете контейнер для предварительной версии с условным просмотром, вам потребуется заполнить [форму интерактивного запроса](https://aka.ms/csgate/) , чтобы использовать ее.

## <a name="docker-compose-file"></a>Файл Docker Compose

Файл YAML определяет все службы для развертывания. Эти службы зависят от либо `DockerFile` существующего образа контейнера. В этом случае мы будем использовать два изображения предварительной версии. Скопируйте и вставьте следующий файл YAML и сохраните его как *DOCKER-сформируйте. YAML*. Укажите соответствующие значения **apiKey**, **выставления счетов** и **EndpointUri** в файле.

```yaml
version: '3.7'
services:
  forms:
    image: "mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout"
    environment:
       eula: accept
       billing: # < Your form recognizer billing URL >
       apikey: # < Your form recognizer API key >
       FormRecognizer__ComputerVisionApiKey: # < Your form recognizer API key >
       FormRecognizer__ComputerVisionEndpointUri: # < Your form recognizer URI >
    volumes:
       - type: bind
         source: E:\publicpreview\output
         target: /output
       - type: bind
         source: E:\publicpreview\input
         target: /input
    ports:
      - "5010:5000"

  ocr:
    image: "mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview"
    environment:
      eula: accept
      apikey: # < Your computer vision API key >
      billing: # < Your computer vision billing URL >
    ports:
      - "5021:5000"
```

> [!IMPORTANT]
> Создайте каталоги на размещающем компьютере, которые указаны в узле **тома** . Этот подход необходим, поскольку каталоги должны существовать до попытки подключения образа с помощью привязок томов.

## <a name="start-the-configured-docker-compose-services"></a>Запуск настроенных служб Docker Compose

Файл Docker Compose обеспечивает управление всеми этапами в жизненном цикле определенной службы: запуск, остановка и перестроение служб; Просмотр состояния службы; и потоковая передача журналов. Откройте интерфейс командной строки из каталога проекта (где находится файл DOCKER-сформируйте. YAML).

> [!NOTE]
> Чтобы избежать ошибок, убедитесь, что главный компьютер правильно использует диски с подсистемой DOCKER. Например, если *е:\публикпревиев* используется в качестве каталога в файле *DOCKER-сформируйте. YAML* , предоставьте общий доступ к диску **E** с помощью DOCKER.

В интерфейсе командной строки выполните следующую команду, чтобы запустить (или перезапустить) все службы, определенные в файле *DOCKER-сформируйте. YAML* :

```console
docker-compose up
```

Первый раз, когда DOCKER выполняет команду **DOCKER-создания up** с помощью этой конфигурации, он извлекает образы, настроенные в узле **службы** , а затем загружает и подключает их:

```console
Pulling forms (mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout:)...
latest: Pulling from azure-cognitive-services/form-recognizer/layout
743f2d6c1f65: Pull complete
72befba99561: Pull complete
2a40b9192d02: Pull complete
c7715c9d5c33: Pull complete
f0b33959f1c4: Pull complete
b8ab86c6ab26: Pull complete
41940c21ed3c: Pull complete
e3d37dd258d4: Pull complete
cdb5eb761109: Pull complete
fd93b5f95865: Pull complete
ef41dcbc5857: Pull complete
4d05c86a4178: Pull complete
34e811d37201: Pull complete
Pulling ocr (mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview:)...
latest: Pulling from /azure-cognitive-services/vision/read:3.1-preview
f476d66f5408: Already exists
8882c27f669e: Already exists
d9af21273955: Already exists
f5029279ec12: Already exists
1a578849dcd1: Pull complete
45064b1ab0bf: Download complete
4bb846705268: Downloading [=========================================>         ]  187.1MB/222.8MB
c56511552241: Waiting
e91d2aa0f1ad: Downloading [==============================================>    ]  162.2MB/176.1MB
```

После скачивания образов запускаются службы образов.

```console
Starting docker_ocr_1   ... done
Starting docker_forms_1 ... doneAttaching to docker_ocr_1, docker_forms_1forms_1  | forms_1  | forms_1  | Notice: This Preview is made available to you on the condition that you agree to the Supplemental Terms of Use for Microsoft Azure Previews [https://go.microsoft.com/fwlink/?linkid=2018815], which supplement your agreement [https://go.microsoft.com/fwlink/?linkid=2018657] governing your use of Azure. If you do not have an existing agreement governing your use of Azure, you agree that your agreement governing use of Azure is the Microsoft Online Subscription Agreement [https://go.microsoft.com/fwlink/?linkid=2018755] (which incorporates the Online Services Terms [https://go.microsoft.com/fwlink/?linkid=2018760]). By using the Preview you agree to these terms.
forms_1  | 
forms_1  | 
forms_1  | Using '/input' for reading models and other read-only data.
forms_1  | Using '/output/forms/812d811d1bcc' for writing logs and other output data.
forms_1  | Logging to console.
forms_1  | Submitting metering to 'https://westus2.api.cognitive.microsoft.com/'.
forms_1  | WARNING: No access control enabled!
forms_1  | warn: Microsoft.AspNetCore.Server.Kestrel[0]
forms_1  |       Overriding address(es) 'http://+:80'. Binding to endpoints defined in UseKestrel() instead.
forms_1  | Hosting environment: Production
forms_1  | Content root path: /app/forms
forms_1  | Now listening on: http://0.0.0.0:5000
forms_1  | Application started. Press Ctrl+C to shut down.
ocr_1    | 
ocr_1    | 
ocr_1    | Notice: This Preview is made available to you on the condition that you agree to the Supplemental Terms of Use for Microsoft Azure Previews [https://go.microsoft.com/fwlink/?linkid=2018815], which supplement your agreement [https://go.microsoft.com/fwlink/?linkid=2018657] governing your use of Azure. If you do not have an existing agreement governing your use of Azure, you agree that your agreement governing use of Azure is the Microsoft Online Subscription Agreement [https://go.microsoft.com/fwlink/?linkid=2018755] (which incorporates the Online Services Terms [https://go.microsoft.com/fwlink/?linkid=2018760]). By using the Preview you agree to these terms.
ocr_1    |
ocr_1    | 
ocr_1    | Logging to console.
ocr_1    | Submitting metering to 'https://westcentralus.api.cognitive.microsoft.com/'.
ocr_1    | WARNING: No access control enabled!
ocr_1    | Hosting environment: Production
ocr_1    | Content root path: /
ocr_1    | Now listening on: http://0.0.0.0:5000
ocr_1    | Application started. Press Ctrl+C to shut down.
```

## <a name="verify-the-service-availability"></a>Проверка доступности службы

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

Вот пример выходных данных:

```
IMAGE ID            REPOSITORY                                                                 TAG
2ce533f88e80        mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout          latest
4be104c126c5        mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview         latest
```

### <a name="test-containers"></a>Тестовые контейнеры

Откройте браузер на главном компьютере и перейдите на узел **localhost** , используя указанный порт из файла *DOCKER-сформируйте. YAML* , например http://localhost:5021/swagger/index.html . Например, можно использовать функцию **попробовать** в API для проверки конечной точки распознавателя форм. Оба контейнера могут быть доступны и тестируемы.

![Контейнер распознавателя форм](media/form-recognizer-swagger-page.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Контейнеры Cognitive Services](../cognitive-services-container-support.md)
