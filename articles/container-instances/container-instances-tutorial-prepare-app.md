---
title: Учебник. Подготовка образа контейнера для развертывания
description: Руководство по службе "Экземпляры контейнеров Azure" —часть 1 из 3. Подготовка приложения в образе контейнера для развертывания в службе "Экземпляры контейнеров Azure".
ms.topic: tutorial
ms.date: 03/21/2018
ms.custom: seodec18, mvc, devx-track-js
ms.openlocfilehash: 1eca34fb5da50b15f15322a3b534556bcd3bda09
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91303120"
---
# <a name="tutorial-create-a-container-image-for-deployment-to-azure-container-instances"></a>Руководство. Создание образа контейнера для развертывания в службе "Экземпляры контейнеров Azure"

Служба "Экземпляры контейнеров Azure" позволяет развертывать контейнеры Docker в инфраструктуре Azure без подготовки виртуальных машин или применения службы более высокого уровня. С помощью этого руководства мы создадим пакет небольшого веб-приложения Node.js в образе контейнера, который можно запустить в службе "Экземпляры контейнеров Azure".

В этой статье (первой части цикла) вы:

> [!div class="checklist"]
> * клонировать исходный код приложения из GitHub;
> * создадите образ контейнера из источника приложения;
> * протестируете образ в локальном окружении Docker.

С помощью следующих руководств мы отправим образ в Реестр контейнеров Azure, а затем развернем его в службе "Экземпляры контейнеров Azure".

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="get-application-code"></a>Получение кода приложения

Пример приложения в этом руководстве включает простое веб-приложение, написанное на [Node.js][nodejs]. Приложение является статической HTML-страницей и выглядит примерно так:

![Приложение из руководства, отображающееся в браузере][aci-tutorial-app]

Используйте Git, чтобы клонировать репозиторий с примером приложения:

```bash
git clone https://github.com/Azure-Samples/aci-helloworld.git
```

Вы также можете [скачать ZIP-архив][aci-helloworld-zip] напрямую из GitHub.

## <a name="build-the-container-image"></a>Создание образа контейнера

На примере файла Dockerfile в примере приложения показано, как создается контейнер. Он запускается из [официального образа Node.js][docker-hub-nodeimage] на основе [Alpine Linux][alpine-linux], небольшого дистрибутива, который хорошо подходит для использования с контейнерами. Затем файлы приложения копируются в контейнер, при помощи диспетчера пакетов узла устанавливаются зависимости, после чего запускается приложение.

```Dockerfile
FROM node:8.9.3-alpine
RUN mkdir -p /usr/src/app
COPY ./app/ /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

Используйте команду [docker build][docker-build], чтобы создать образ контейнера, и добавьте тег *aci-tutorial-app*:

```bash
docker build ./aci-helloworld -t aci-tutorial-app
```

Выходные данные команды [docker build][docker-build] будут выглядеть примерно так (сокращено для удобства):

```console
$ docker build ./aci-helloworld -t aci-tutorial-app
Sending build context to Docker daemon  119.3kB
Step 1/6 : FROM node:8.9.3-alpine
8.9.3-alpine: Pulling from library/node
88286f41530e: Pull complete
84f3a4bf8410: Pull complete
d0d9b2214720: Pull complete
Digest: sha256:c73277ccc763752b42bb2400d1aaecb4e3d32e3a9dbedd0e49885c71bea07354
Status: Downloaded newer image for node:8.9.3-alpine
 ---> 90f5ee24bee2
...
Step 6/6 : CMD node /usr/src/app/index.js
 ---> Running in f4a1ea099eec
 ---> 6edad76d09e9
Removing intermediate container f4a1ea099eec
Successfully built 6edad76d09e9
Successfully tagged aci-tutorial-app:latest
```

Выполните команду [docker images][docker-images], чтобы просмотреть созданный образ:

```bash
docker images
```

Созданный образ должен отображаться в списке:

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 seconds ago    68.1 MB
```

## <a name="run-the-container-locally"></a>Локальный запуск контейнера

Прежде чем развертывать контейнер в службе "Экземпляры контейнеров Azure", запустите его локально с помощью команды [docker run][docker-run], чтобы проверить, работает ли он. При помощи параметра `-d` можно запустить контейнер в фоновом режиме, а при помощи `-p` — сопоставить произвольный порт на компьютере с портом 80 в контейнере.

```bash
docker run -d -p 8080:80 aci-tutorial-app
```

Выходные данные команды `docker run` отображают идентификатор запущенного контейнера, если команда выполнена успешно:

```console
$ docker run -d -p 8080:80 aci-tutorial-app
a2e3e4435db58ab0c664ce521854c2e1a1bda88c9cf2fcff46aedf48df86cccf
```

Откройте в браузере страницу `http://localhost:8080`, чтобы проверить, запущен ли контейнер. Вы должны увидеть похожую страницу:

![Локальный запуск приложения в браузере][aci-tutorial-app-local]

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы создали образ контейнера, который можно развернуть в службе "Экземпляры контейнеров Azure", и проверили его запуск локально. На этом этапе вы выполнили следующие задачи:

> [!div class="checklist"]
> * клонирование источника приложения из GitHub;
> * создание образа контейнера из источника приложения;
> * локальное тестирование контейнера.

Переходите к следующему руководству, чтобы узнать о хранении образов контейнеров в Реестре контейнеров Azure.

> [!div class="nextstepaction"]
> [Передача образа в реестр контейнеров Azure](container-instances-tutorial-prepare-acr.md)

<!--- IMAGES --->
[aci-tutorial-app]:./media/container-instances-quickstart/aci-app-browser.png
[aci-tutorial-app-local]: ./media/container-instances-tutorial-prepare-app/aci-app-browser-local.png

<!-- LINKS - External -->
[aci-helloworld-zip]: https://github.com/Azure-Samples/aci-helloworld/archive/master.zip
[alpine-linux]: https://alpinelinux.org/
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: https://nodejs.org

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
