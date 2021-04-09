---
title: Руководство по Kubernetes в Azure. Подготовка приложения
description: В этом руководстве по Службе Azure Kubernetes (AKS) вы узнаете, как подготовить и создать многоконтейнерное приложение с помощью Docker Compose, которое можно затем развернуть в AKS.
services: container-service
ms.topic: tutorial
ms.date: 01/12/2021
ms.custom: mvc
ms.openlocfilehash: 349bf90ea0b344d5232c885358814f39fba4c19f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98251965"
---
# <a name="tutorial-prepare-an-application-for-azure-kubernetes-service-aks"></a>Руководство. Подготовка приложения для Службы Azure Kubernetes (AKS)

В этом руководстве (часть 1 из 7) выполняется подготовка многоконтейнерного приложения к использованию в Kubernetes. Имеющиеся средства разработки, такие как Docker Compose, используются для локального создания приложения и его тестирования. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Клонирование источника примера приложения с GitHub.
> * Создание образа контейнера из источника примера приложения.
> * Тестирование многоконтейнерного приложения в локальной среде Docker.

После выполнения всех действий в вашей локальной среде разработки выполняется следующее приложение:

:::image type="content" source="./media/container-service-kubernetes-tutorials/azure-vote-local.png" alt-text="Снимок экрана: образ контейнера приложения Azure для голосования, выполняемого локально и открытого в веб-браузере." lightbox="./media/container-service-kubernetes-tutorials/azure-vote-local.png":::

В дальнейших руководствах образ контейнера отправляется в Реестр контейнеров Azure, а затем развертывается в кластере AKS.

## <a name="before-you-begin"></a>Перед началом

Для выполнения действий, описанных в этом руководстве, необходимо базовое понимание основных понятий Docker, таких как контейнеры, образы контейнеров и команды `docker`. [Руководство по началу работы с Docker][docker-get-started] содержит базовые сведения о контейнерах.

Для работы с этим руководством требуется локальная среда разработки Docker для выполнения контейнеров Linux. Docker предоставляет пакеты, которые позволяют настроить Docker в системе [Mac][docker-for-mac], [Windows][docker-for-windows] или [Linux][docker-for-linux].

> [!NOTE]
> Azure Cloud Shell не включает в себя компоненты Docker, необходимые для выполнения каждого шага этих руководств. Таким образом мы рекомендуем полную среду разработки Docker.

## <a name="get-application-code"></a>Получение кода приложения

Используемый в этом руководстве [пример приложения][sample-application] — это базовое приложение для голосования, состоящее из интерфейсного веб-компонента и экземпляра серверной части Redis. Веб-компонент упаковывается в пользовательский образ контейнера, а для экземпляра Redis используется неизмененный образ из Docker Hub.

Используйте команду [git][], чтобы клонировать пример приложения в среду разработки:

```console
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

Перейдите в клонированный каталог.

```console
cd azure-voting-app-redis
```

Внутри каталога содержится исходный код приложения, предварительно созданный файл Docker Compose и файл манифеста Kubernetes. Эти файлы используются в руководстве. Содержимое и структура каталога следующие:

```output
azure-voting-app-redis
│   azure-vote-all-in-one-redis.yaml
│   docker-compose.yaml
│   LICENSE
│   README.md
│
├───azure-vote
│   │   app_init.supervisord.conf
│   │   Dockerfile
│   │   Dockerfile-for-app-service
│   │   sshd_config
│   │
│   └───azure-vote
│       │   config_file.cfg
│       │   main.py
│       │
│       ├───static
│       │       default.css
│       │
│       └───templates
│               index.html
│
└───jenkins-tutorial
        config-jenkins.sh
        deploy-jenkins-vm.sh
```

## <a name="create-container-images"></a>Создание образов контейнеров

С помощью [Docker Compose][docker-compose] можно автоматизировать создание дополнительных образов контейнеров и развертывание многоконтейнерных приложений.

Используйте пример файла `docker-compose.yaml`, чтобы создать образ контейнера, скачать образ Redis и запустить приложение:

```console
docker-compose up -d
```

После завершения выполните команду [docker images][docker-images], чтобы увидеть созданные образы. Было создано или скачано три образа. Образ *azure-vote-front* содержит интерфейсное приложение и использует образ *nginx-flask* в качестве основы. Образ *redis* используется для запуска экземпляра Redis.

```
$ docker images

REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/azuredocs/azure-vote-front   v1                  84b41c268ad9        9 seconds ago       944MB
mcr.microsoft.com/oss/bitnami/redis            6.0.8               3a54a920bb6c        2 days ago          103MB
tiangolo/uwsgi-nginx-flask                     python3.6           a16ce562e863        6 weeks ago         944MB
```

Выполните команду [docker ps][docker-ps], чтобы просмотреть выполняемые контейнеры:

```
$ docker ps

CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS              PORTS                           NAMES
d10e5244f237        mcr.microsoft.com/azuredocs/azure-vote-front:v1   "/entrypoint.sh /sta…"   3 minutes ago       Up 3 minutes        443/tcp, 0.0.0.0:8080->80/tcp   azure-vote-front
21574cb38c1f        mcr.microsoft.com/oss/bitnami/redis:6.0.8         "/opt/bitnami/script…"   3 minutes ago       Up 3 minutes        0.0.0.0:6379->6379/tcp          azure-vote-back
```

## <a name="test-application-locally"></a>Локальное тестирование приложения

Чтобы увидеть работающее приложение, введите `http://localhost:8080` в локальном веб-браузере. Нагрузки примера приложения, как показано в следующем примере:

:::image type="content" source="./media/container-service-kubernetes-tutorials/azure-vote-local.png" alt-text="Снимок экрана: образ контейнера приложения Azure для голосования, выполняемого локально и открытого в веб-браузере." lightbox="./media/container-service-kubernetes-tutorials/azure-vote-local.png":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Теперь, когда функциональные возможности приложения проверены, запущенные контейнеры можно остановить и удалить. ***Не удаляйте образы контейнеров.** _ В следующем руководстве образ _azure-vote-front* будет отправлен в экземпляр Реестра контейнеров Azure.

Остановите и удалите экземпляры контейнеров и ресурсы с помощью команды [docker-compose down][docker-compose-down]:

```console
docker-compose down
```

При удалении локального приложения у вас остается образ Docker, содержащий приложение Azure для голосования (*azure-vote-front*), которое используется в следующем руководстве.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы протестировали приложение и создали для него образы контейнеров. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Клонирование источника примера приложения с GitHub.
> * Создание образа контейнера из источника примера приложения.
> * Тестирование многоконтейнерного приложения в локальной среде Docker.

Переходите к следующему руководству, чтобы узнать о том, как хранить образы контейнеров в Реестре контейнеров Azure.

> [!div class="nextstepaction"]
> [Развертывание реестра контейнеров Azure и его использование][aks-tutorial-prepare-acr]

<!-- LINKS - external -->
[docker-compose]: https://docs.docker.com/compose/
[docker-for-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-for-mac]: https://docs.docker.com/docker-for-mac/
[docker-for-windows]: https://docs.docker.com/docker-for-windows/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-ps]: https://docs.docker.com/engine/reference/commandline/ps/
[docker-compose-down]: https://docs.docker.com/compose/reference/down
[git]: https://git-scm.com/downloads
[sample-application]: https://github.com/Azure-Samples/azure-voting-app-redis

<!-- LINKS - internal -->
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
