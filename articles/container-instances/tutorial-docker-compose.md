---
title: Учебник по развертыванию многоконтейнерной группы с помощью Docker Compose
description: Использование Docker Compose для создания и запуска многоконтейнерного приложения и последующего его переноса в Экземпляры контейнеров Azure
ms.topic: tutorial
ms.date: 10/28/2020
ms.custom: ''
ms.openlocfilehash: ef08b9f9e0f596f1d94c0e6edfd46f735fe78053
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786926"
---
# <a name="tutorial-deploy-a-multi-container-group-using-docker-compose"></a>Руководство по развертыванию многоконтейнерной группы с помощью Docker Compose 

В рамках этого учебника с помощью [Docker Compose](https://docs.docker.com/compose/) вы определите и запустите многоконтейнерное приложение локально, а затем развернете его как [группу контейнеров](container-instances-container-groups.md) в Экземплярах контейнеров Azure. 

Если вы разрабатываете облачные приложения с помощью Docker и хотите легко переходить с локальной разработки на облачное развертывание, запустите контейнеры в Экземплярах контейнеров Azure по запросу. Эта возможность предоставляется благодаря [интеграции Docker и Azure](https://docs.docker.com/engine/context/aci-integration/). Вы можете использовать собственные команды Docker для запуска [одного экземпляра контейнера](quickstart-docker-cli.md) или многоконтейнерной группы в Azure.

> [!IMPORTANT]
> Поддерживаются не все функции Экземпляров контейнеров Azure. Предоставьте отзыв об интеграции Docker-Azure, создав вопрос в репозитории GitHub в разделе [интеграции Docker ACI](https://github.com/docker/aci-integration-beta).

> [!TIP]
> Кроме того, вы можете использовать [расширение Docker для Visual Studio Code](https://aka.ms/VSCodeDocker), чтобы получить интегрированный интерфейс для разработки, запуска контейнеров, изображений и контекстов, а также управления ими.

Работая с этой статьей, вы выполните следующие задачи:

> [!div class="checklist"]
> * Создание реестра контейнеров Azure
> * клонировать исходный код приложения из GitHub;
> * использование Docker Compose для создания образа и запуска многоконтейнерного приложения в локальной среде;
> * отправка образа приложения в реестр контейнеров;
> * создание контекста Azure для Docker;
> * запуск приложения в Экземплярах контейнеров Azure.

## <a name="prerequisites"></a>Prerequisites

* **Azure CLI**. На локальном компьютере должен быть установлен Azure CLI. Рекомендуется использовать версию 2.10.1 или более позднюю. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

* **Docker Desktop**. Необходимо использовать Docker Desktop 2.3.0.5 или более поздней версии, доступный для [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) или [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). Или установите [интерфейс командной строки интеграции Docker ACI для Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux).

[!INCLUDE [container-instances-create-registry](../../includes/container-instances-create-registry.md)]

## <a name="get-application-code"></a>Получение кода приложения

В этом руководстве используется простое приложение для голосования. Приложение состоит из веб-компонента интерфейсной части и серверного экземпляра Redis. Веб-компонент упаковывается в пользовательский образ контейнера, а для экземпляра Redis используется неизмененный образ из Docker Hub.

Используйте команду [git](https://git-scm.com/downloads), чтобы клонировать пример приложения в среду разработки:

```console
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

Перейдите в клонированный каталог.

```console
cd azure-voting-app-redis
```

Внутри каталога содержится исходный код приложения, предварительно созданный файл Docker Compose (docker-compose.yaml).

## <a name="modify-docker-compose-file"></a>Изменение файла Docker Compose

Откройте docker-compose.yaml в текстовом редакторе. Файл настраивает службы `azure-vote-back` и `azure-vote-front`.

```yml
version: '3'
services:
  azure-vote-back:
    image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
    container_name: azure-vote-back
    environment:
      ALLOW_EMPTY_PASSWORD: "yes"
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "8080:80"
```

В конфигурации `azure-vote-front` выполните указанные ниже два изменения.

1. Обновите свойство `image` в службе `azure-vote-front`. Добавьте к имени образа имя сервера для входа в реестр контейнеров Azure (\<acrName\>.azurecr.io) в виде префикса. Например, если реестр имеет имя *myregistry*, именем сервера для входа является *myregistry.azurecr.io* (нижний регистр), а свойством образа — `myregistry.azurecr.io/azure-vote-front`.
1. Измените сопоставление `ports` на `80:80`. Сохраните файл.

Обновленный файл должен выглядеть следующим образом:

```yml
version: '3'
services:
  azure-vote-back:
    image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
    container_name: azure-vote-back
    environment:
      ALLOW_EMPTY_PASSWORD: "yes"
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: myregistry.azurecr.io/azure-vote-front
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "80:80"
```

В результате этих замен образ `azure-vote-front`, который вы создадите в следующем шаге, будет помечен тегами для вашего реестра контейнеров Azure. Этот образ можно будет извлечь для запуска в Экземплярах контейнеров Azure.

> [!TIP]
> В этом сценарии не нужно использовать реестр контейнеров Azure. Например, для размещения образа приложения можно выбрать частный репозиторий в Docker Hub. При выборе другого реестра обновите свойство образа соответственно.

## <a name="run-multi-container-application-locally"></a>Локальное выполнение многоконтейнерного приложения

Запустите [docker-compose up](https://docs.docker.com/compose/reference/up/), который использует пример файла `docker-compose.yaml` для создания образа контейнера, скачайте образ Redis и запустите приложение:

```console
docker-compose up --build -d
```

После завершения выполните команду [docker images](https://docs.docker.com/engine/reference/commandline/images/), чтобы увидеть созданные образы. Было создано или скачано три образа. Образ `azure-vote-front` содержит внешнее приложение, в котором в качестве основы используется образ `uwsgi-nginx-flask`. Образ `redis` используется для запуска экземпляра Redis.

```
$ docker images

REPOSITORY                                TAG        IMAGE ID            CREATED             SIZE
myregistry.azurecr.io/azure-vote-front    latest     9cc914e25834        40 seconds ago      944MB
mcr.microsoft.com/oss/bitnami/redis       6.0.8      3a54a920bb6c        4 weeks ago          103MB
tiangolo/uwsgi-nginx-flask                python3.6  788ca94b2313        9 months ago        9444MB
```

Выполните команду [docker ps](https://docs.docker.com/engine/reference/commandline/ps/), чтобы просмотреть выполняемые контейнеры:

```
$ docker ps

CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS                           NAMES
82411933e8f9        myregistry.azurecr.io/azure-vote-front     "/entrypoint.sh /sta…"   57 seconds ago      Up 30 seconds       443/tcp, 0.0.0.0:80->80/tcp   azure-vote-front
b62b47a7d313        mcr.microsoft.com/oss/bitnami/redis:6.0.8  "/opt/bitnami/script…"   57 seconds ago      Up 30 seconds       0.0.0.0:6379->6379/tcp          azure-vote-back
```

Чтобы увидеть работающее приложение, введите `http://localhost:80` в локальном веб-браузере. Нагрузки примера приложения, как показано в следующем примере:

:::image type="content" source="media/tutorial-docker-compose/azure-vote.png" alt-text="Образ приложения для голосования":::

После использования локального приложения запустите [docker-compose down](https://docs.docker.com/compose/reference/down/), чтобы закрыть приложение и удалить контейнеры.

```console
docker-compose down
```

## <a name="push-image-to-container-registry"></a>Отправка образа в реестр контейнеров

Чтобы развернуть приложение в Экземплярах контейнеров Azure, необходимо отправить образ `azure-vote-front` в реестр контейнеров. Чтобы отправить образ, выполните [docker-compose push](https://docs.docker.com/compose/reference/push):

```console
docker-compose push
```

Отправка в реестр может занять несколько минут.

Чтобы убедиться, что образ хранится в реестре, выполните команду [az acr repository show](/cli/azure/acr/repository#az_acr_repository_show):

```azurecli
az acr repository show --name <acrName> --repository azure-vote-front
```

[!INCLUDE [container-instances-create-docker-context](../../includes/container-instances-create-docker-context.md)]

## <a name="deploy-application-to-azure-container-instances"></a>Развертывание приложения в Экземплярах контейнеров Azure

Перейдите в контекст ACI. В этом контексте выполняются последующие команды Docker.

```console
docker context use myacicontext
```

Выполните `docker compose up`, чтобы запустить приложение в Экземплярах контейнеров Azure. Образ `azure-vote-front` извлекается из реестра контейнеров, и в Экземплярах контейнеров Azure создается группа контейнеров.

```console
docker compose up
```

> [!NOTE]
> Команды Docker Compose `docker compose up` и `docker compose down` в настоящее время доступны в контексте ACI. В этих командах отсутствует дефис между `docker` и `compose`.

Через некоторое время группа контейнеров будет развернута. Образец вывода:

```
[+] Running 3/3
 ⠿ Group azurevotingappredis  Created                          3.6s
 ⠿ azure-vote-back            Done                             10.6s
 ⠿ azure-vote-front           Done                             10.6s
```

Запустите `docker ps`, чтобы просмотреть запущенные контейнеры и IP-адрес, назначенный группе контейнеров.

```console
docker ps
```

Образец вывода:

```
CONTAINER ID                           IMAGE                                         COMMAND             STATUS              PORTS
azurevotingappredis_azure-vote-back    mcr.microsoft.com/oss/bitnami/redis:6.0.8                         Running             52.179.23.131:6379->6379/tcp
azurevotingappredis_azure-vote-front   myregistry.azurecr.io/azure-vote-front                            Running             52.179.23.131:80->80/tcp
```

Чтобы просмотреть запущенное приложение в облаке, введите отображаемый IP-адрес в локальном веб-браузере. Для данного примера введите `52.179.23.131`. Нагрузки примера приложения, как показано в следующем примере:

:::image type="content" source="media/tutorial-docker-compose/azure-vote-aci.png" alt-text="Образ приложения для голосования в ACI":::

Чтобы просмотреть журналы внешнего контейнера, выполните команду [docker logs](https://docs.docker.com/engine/reference/commandline/logs). Пример:

```console
docker logs azurevotingappredis_azure-vote-front
```

Вы также можете использовать портал Azure или другие инструменты Azure, чтобы просмотреть свойства и состояние развернутой группы контейнеров.

Завершив использование приложения, закройте его и контейнеры с помощью `docker compose down`:

```console
docker compose down
```

Эта команда удаляет группу контейнеров в Экземплярах контейнеров Azure.

## <a name="next-steps"></a>Следующие шаги

При изучении этого учебника вы использовали Docker Compose, чтобы переключиться с запуска многоконтейнерного приложения в локальной среде на запуск в Экземплярах контейнеров Azure. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание реестра контейнеров Azure
> * клонировать исходный код приложения из GitHub;
> * использование Docker Compose для создания образа и запуска многоконтейнерного приложения в локальной среде;
> * отправка образа приложения в реестр контейнеров;
> * создание контекста Azure для Docker;
> * запуск приложения в Экземплярах контейнеров Azure.

Кроме того, вы можете использовать [расширение Docker для Visual Studio Code](https://aka.ms/VSCodeDocker), чтобы получить доступ к интегрированному интерфейсу для разработки, запуска контейнеров, образов и контекстов, а также управления ими.

Если вы хотите воспользоваться преимуществами дополнительных функций в Экземплярах контейнеров Azure, с помощью инструментов Azure укажите многоконтейнерную группу. Например, ознакомьтесь с учебниками по развертыванию группы контейнеров с помощью Azure CLI и [YAML-файла](container-instances-multi-container-yaml.md) или выполните развертывание с помощью [шаблона Azure Resource Manager](container-instances-multi-container-group.md). 