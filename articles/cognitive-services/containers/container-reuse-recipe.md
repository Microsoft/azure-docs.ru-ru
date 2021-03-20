---
title: Рецепты для контейнеров DOCKER
titleSuffix: Azure Cognitive Services
description: Узнайте, как создавать, тестировать и хранить контейнеры с помощью некоторых или всех параметров конфигурации для развертывания и повторного использования.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 7380ff58d033a68565de7e419ff318f7bdec121d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80875084"
---
# <a name="create-containers-for-reuse"></a>Создание контейнеров для повторного использования

Используйте эти рецепты контейнеров для создания Cognitive Services контейнеров, которые можно использовать повторно. Контейнеры можно построить с помощью некоторых или всех параметров конфигурации, чтобы они _не_ требовались при запуске контейнера.

После создания нового слоя контейнера (с параметрами) и его локального тестирования можно сохранить контейнер в реестре контейнеров. При запуске контейнера потребуются только те параметры, которые в данный момент не хранятся в контейнере. Закрытый контейнер реестра предоставляет пространство конфигурации для передачи этих параметров в.

## <a name="docker-run-syntax"></a>Синтаксис выполнения DOCKER

Все `docker run` примеры в этом документе предполагают наличие в консоли Windows `^` символа продолжения строки. Примите во внимание следующие сведения для собственного использования:

* Не изменяйте порядок аргументов, если вы не являетесь уверенным пользователем контейнеров Docker.
* Если используется операционная система, отличная от Windows, или консоль, отличная от консоли Windows, используйте правильную консоль или терминал, синтаксис папок для подключений и символ продолжения строки для консоли и системы.  Так как контейнер Cognitive Services является операционной системой Linux, целевое подключение использует синтаксис папки в формате Linux.
* `docker run` в примерах можно использовать каталог на `c:` диске, чтобы избежать конфликтов разрешений в Windows. Если вам нужен конкретный каталог для входных данных, может потребоваться предоставить соответствующие разрешения службе Docker.

## <a name="store-no-configuration-settings-in-image"></a>Не сохранять параметры конфигурации в образе

В примерах `docker run` команд для каждой службы не сохраняются параметры конфигурации в контейнере. При запуске контейнера из консоли или службы реестра эти параметры конфигурации должны быть переданы. Закрытый контейнер реестра предоставляет пространство конфигурации для передачи этих параметров в.

## <a name="reuse-recipe-store-all-configuration-settings-with-container"></a>Рецепт повторного использования: хранение всех параметров конфигурации в контейнере

Чтобы сохранить все параметры конфигурации, создайте `Dockerfile` с этими параметрами.

Проблемы с таким подходом:

* Новый контейнер имеет отдельное имя и тег из исходного контейнера.
* Чтобы изменить эти параметры, необходимо изменить значения Dockerfile, перестроить образ и повторно опубликовать в реестре.
* Если кто-то получает доступ к реестру контейнеров или локальному узлу, он может запустить контейнер и использовать конечные точки Cognitive Services.
* Если для работы службы не требуются входные подключения, не добавляйте `COPY` строки в Dockerfile.

Создайте Dockerfile, потянув из существующего контейнера Cognitive Services, который вы хотите использовать, а затем используйте команды DOCKER в Dockerfile, чтобы задать или извлечь сведения, необходимые для контейнера.

В этом примере:

* Задает конечную точку выставления счетов `{BILLING_ENDPOINT}` из ключа среды узла с помощью `ENV` .
* Задает ключ API выставления счетов `{ENDPOINT_KEY}` из ключа среды узла с помощью "env.

### <a name="reuse-recipe-store-billing-settings-with-container"></a>Рецепт повторного использования: хранение параметров выставления счетов с помощью контейнера

В этом примере показано, как создать контейнер тональности Анализ текста из Dockerfile.

```Dockerfile
FROM mcr.microsoft.com/azure-cognitive-services/sentiment:latest
ENV billing={BILLING_ENDPOINT}
ENV apikey={ENDPOINT_KEY}
ENV EULA=accept
```

Создавайте и запускайте контейнер [локально](#how-to-use-container-on-your-local-host) или из [закрытого контейнера реестра](#how-to-add-container-to-private-registry) по мере необходимости.

### <a name="reuse-recipe-store-billing-and-mount-settings-with-container"></a>Рецепт повторного использования: хранение параметров выставления счетов и подключения с помощью контейнера

В этом примере показано, как использовать Распознавание речи, сохранив выставление счетов и модели из Dockerfile.

* Копирует файл модели Распознавание речи (LUIS) из файловой системы узла с помощью `COPY` .
* Контейнер LUIS поддерживает более одной модели. Если все модели хранятся в одной и той же папке, все они должны иметь одну `COPY` инструкцию.
* Запустите файл DOCKER из относительного родителя входного каталога модели. В следующем примере выполните `docker build` `docker run` команды и из относительного родителя `/input` . Первый `/input` в `COPY` команде — это каталог главного компьютера. Второй `/input` — Каталог контейнера.

```Dockerfile
FROM <container-registry>/<cognitive-service-container-name>:<tag>
ENV billing={BILLING_ENDPOINT}
ENV apikey={ENDPOINT_KEY}
ENV EULA=accept
COPY /input /input
```

Создавайте и запускайте контейнер [локально](#how-to-use-container-on-your-local-host) или из [закрытого контейнера реестра](#how-to-add-container-to-private-registry) по мере необходимости.

## <a name="how-to-use-container-on-your-local-host"></a>Использование контейнера на локальном узле

Чтобы создать файл DOCKER, замените на `<your-image-name>` новое имя образа, а затем используйте:

```console
docker build -t <your-image-name> .
```

Чтобы запустить образ и удалить его при остановке контейнера ( `--rm` ):

```console
docker run --rm <your-image-name>
```

## <a name="how-to-add-container-to-private-registry"></a>Добавление контейнера в частный реестр

Выполните следующие действия, чтобы использовать Dockerfile и поместить новый образ в закрытый реестр контейнеров.  

1. Создайте `Dockerfile` с помощью рецепта "текст из повторного использования". `Dockerfile`Не имеет расширения.

1. Замените все значения в угловых скобках собственными значениями.

1. Создайте файл в образе в командной строке или терминале, используя следующую команду. Замените значения в угловых скобках на `<>` собственное имя контейнера и тег.  

    Параметр тега, `-t` — это способ добавления сведений о том, что было изменено для контейнера. Например, имя контейнера `modified-LUIS` указывает, что исходный контейнер был слоем. Имя тега `with-billing-and-model` показывает, как был изменен контейнер распознавание речи (Luis).

    ```Bash
    docker build -t <your-new-container-name>:<your-new-tag-name> .
    ```

1. Войдите в Azure CLI из консоли. Эта команда открывает браузер и требует проверки подлинности. После проверки подлинности можно закрыть браузер и продолжить работу в консоли.

    ```azurecli
    az login
    ```

1. Войдите в свой частный реестр с Azure CLI из консоли.

    Замените значения в угловых скобках на `<my-registry>` собственное имя реестра.  

    ```azurecli
    az acr login --name <my-registry>
    ```

    Вы также можете войти с помощью DOCKER Login, если назначен субъект-служба.

    ```Bash
    docker login <my-registry>.azurecr.io
    ```

1. Пометьте контейнер с частным расположением реестра. Замените значения в угловых скобках на `<my-registry>` собственное имя реестра. 

    ```Bash
    docker tag <your-new-container-name>:<your-new-tag-name> <my-registry>.azurecr.io/<your-new-container-name-in-registry>:<your-new-tag-name>
    ```

    Если имя тега не используется, `latest` подразумевается.

1. Отправьте новый образ в закрытый реестр контейнеров. При просмотре закрытого реестра контейнеров имя контейнера, используемое в следующей команде CLI, будет именем репозитория.

    ```Bash
    docker push <my-registry>.azurecr.io/<your-new-container-name-in-registry>:<your-new-tag-name>
    ```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание и использование экземпляра контейнера Azure](azure-container-instance-recipe.md)

<!--
## Store input and output configuration settings

Bake in input params only

FROM containerpreview.azurecr.io/microsoft/cognitive-services-luis:<tag>
COPY luisModel1 /input/
COPY luisModel2 /input/

## Store all configuration settings

If you are a single manager of the container, you may want to store all settings in the container. The new, resulting container will not need any variables passed in to run. 

Issues with this approach:

* In order to change these settings, you will have to change the values of the Dockerfile and rebuild the file. 
* If someone gets access to your container registry or your local host, they can run the container and use the Cognitive Services endpoints. 

The following _partial_ Dockerfile shows how to statically set the values for billing and model. This example uses the 

```Dockerfile
FROM <container-registry>/<cognitive-service-container-name>:<tag>
ENV billing=<billing value>
ENV apikey=<apikey value>
COPY luisModel1 /input/
COPY luisModel2 /input/
```

->