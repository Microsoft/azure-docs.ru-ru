---
title: Отправить & образ контейнера извлечения
description: Отправка и извлечение образов DOCKER в закрытый реестр контейнеров в Azure с помощью DOCKER CLI
ms.topic: article
ms.date: 01/23/2019
ms.custom: seodec18, H1Hack27Feb2017
ms.openlocfilehash: 83ef385313b035f5e5d7d993e7948725906c75a7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99987765"
---
# <a name="push-your-first-image-to-your-azure-container-registry-using-the-docker-cli"></a>Отправка первого образа в реестр контейнеров Azure с помощью DOCKER CLI

Реестр контейнеров Azure хранит и управляет частными образами контейнеров и другими артефактами, аналогично тому, как [центр DOCKER](https://hub.docker.com/) хранит общедоступные образы контейнеров DOCKER. Вы можете использовать [интерфейс командной строки DOCKER](https://docs.docker.com/engine/reference/commandline/cli/) (DOCKER CLI) для [входа](https://docs.docker.com/engine/reference/commandline/login/), [отправки](https://docs.docker.com/engine/reference/commandline/push/), [извлечения](https://docs.docker.com/engine/reference/commandline/pull/)и других операций с образами контейнеров в реестре контейнеров.

На следующих шагах вы скачиваете общедоступный [образ nginx](https://store.docker.com/images/nginx), размещаете его для частного реестра контейнеров Azure, отправляете его в реестр, а затем извлекаете из реестра.

## <a name="prerequisites"></a>Предварительные условия

* **Реестр контейнеров Azure** . Создайте реестр контейнеров в подписке Azure. Это можно сделать на [портале Azure](container-registry-get-started-portal.md) или с помощью [Azure CLI](container-registry-get-started-azure-cli.md).
* **Docker CLI**. Также необходим локально установленный модуль Docker. Docker предоставляет пакеты, которые позволяют быстро настроить Docker в системе под управлением [macOS][docker-mac], [Windows][docker-windows] или [Linux][docker-linux].

## <a name="log-in-to-a-registry"></a>Вход в раздел реестра

Существуют [несколько способов выполнить аутентификацию](container-registry-authentication.md) в частном реестре контейнеров. При работе в командной строке мы рекомендуем выполнить команду Azure CLI [az acr login](/cli/azure/acr#az-acr-login). Например, чтобы войти в реестр с именем *myregistry*, войдите в Azure CLI, а затем выполните аутентификацию в реестре:

```azurecli
az login
az acr login --name myregistry
```

Вы также можете выполнить вход с помощью команды [docker login](https://docs.docker.com/engine/reference/commandline/login/). Например, для сценария автоматизации может быть [назначен субъект-служба](container-registry-authentication.md#service-principal) в реестре. При запуске следующей команды, когда появляется запрос в интерактивном режиме, укажите идентификатор приложения (имя пользователя) и пароль субъекта-службы. Рекомендации по управлению учетными данными см. в справочнике по команде [docker login](https://docs.docker.com/engine/reference/commandline/login/).

```
docker login myregistry.azurecr.io
```

По завершению обе команды возвращают `Login Succeeded`.
> [!NOTE]
>* Для более быстрого и удобного входа можно использовать Visual Studio Code с расширенными возможностями DOCKER.

> [!TIP]
> Всегда указывайте полное имя реестра (все знаки в нижнем реестре), когда используете `docker login` и когда отмечаете изображения тегами для отправки в свой реестр. В примерах этой статьи полное доменное имя выглядит так: *myregistry.azurecr.io*.

## <a name="pull-a-public-nginx-image"></a>Извлечение общедоступного образа nginx

Сначала необходимо получить открытый образ nginx на локальном компьютере. В этом примере извлекается образ из реестра контейнеров (Майкрософт).

```
docker pull mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

## <a name="run-the-container-locally"></a>Локальный запуск контейнера

Выполните следующую команду [DOCKER Run](https://docs.docker.com/engine/reference/run/) , чтобы запустить локальный экземпляр контейнера nginx в интерактивном режиме ( `-it` ) через порт 8080. Аргумент `--rm` указывает, что контейнер следует удалить, когда вы его остановите.

```
docker run -it --rm -p 8080:80 mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

Перейдите по адресу `http://localhost:8080`, чтобы просмотреть веб-страницу по умолчанию, обслуживаемую Nginx в выполняющемся контейнере. Вы должны увидеть страницу, аналогичную показанной ниже:

![Nginx на локальном компьютере](./media/container-registry-get-started-docker-cli/nginx.png)

Так как вы запустили контейнер в интерактивном режиме с помощью `-it`, вы можете увидеть выходные данные сервера Nginx в командной строке после перемещения к нему в браузере.

Чтобы остановить и удалить контейнер, нажмите `Control`+`C`.

## <a name="create-an-alias-of-the-image"></a>Создание псевдонима образа

Выполните команду [docker tag](https://docs.docker.com/engine/reference/commandline/tag/), чтобы создать псевдоним образа с полным путем к вашему реестру. Чтобы избежать беспорядка в корне реестра, эта команда указывает пространство имен `samples`.

```
docker tag mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine myregistry.azurecr.io/samples/nginx
```

Дополнительные сведения о расстановке тегов с помощью пространства имен см. в разделе [Пространства имен репозитория](container-registry-best-practices.md#repository-namespaces) статьи [Рекомендации по использованию реестра контейнеров Azure](container-registry-best-practices.md).

## <a name="push-the-image-to-your-registry"></a>Отправка образа в реестр

Теперь, когда вы отметили образ с абсолютным путем к частному реестру тегами, его можно отправить в реестр, выполнив команду [docker push](https://docs.docker.com/engine/reference/commandline/push/).

```
docker push myregistry.azurecr.io/samples/nginx
```

## <a name="pull-the-image-from-your-registry"></a>Извлечение образа из реестра

Выполните команду [docker pull](https://docs.docker.com/engine/reference/commandline/pull/), чтобы извлечь образ из реестра:

```
docker pull myregistry.azurecr.io/samples/nginx
```

## <a name="start-the-nginx-container"></a>Запуск контейнера Nginx

Выполните команду [docker run](https://docs.docker.com/engine/reference/run/), чтобы запустить образ, извлеченный из реестра:

```
docker run -it --rm -p 8080:80 myregistry.azurecr.io/samples/nginx
```

Перейдите по адресу `http://localhost:8080`, чтобы просмотреть запущенный контейнер.

Чтобы остановить и удалить контейнер, нажмите `Control`+`C`.

## <a name="remove-the-image-optional"></a>Удаление образа (необязательно)

Если вам больше не нужен образ Nginx, можно удалить его локально, выполнив команду [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/).

```
docker rmi myregistry.azurecr.io/samples/nginx
```

Чтобы удалить образы из реестра контейнеров Azure, выполните команду Azure CLI [az acr repository delete](/cli/azure/acr/repository#az-acr-repository-delete). Например, эта команда удаляет манифест, на который ссылается тег `samples/nginx:latest`, все данные отдельного слоя и все другие теги, ссылающиеся на манифест.

```azurecli
az acr repository delete --name myregistry --image samples/nginx:latest
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знаете основы, можно приступать к использованию реестра. Например, можно развернуть образы контейнера из реестра в следующие службы:

* [Служба Azure Kubernetes (AKS)](../aks/tutorial-kubernetes-prepare-app.md)
* [Экземпляры контейнеров Azure](../container-instances/container-instances-tutorial-prepare-app.md);
* [Service Fabric](../service-fabric/service-fabric-tutorial-create-container-images.md)

При необходимости установите [расширение Docker для Visual Studio Code](https://code.visualstudio.com/docs/azure/docker) и расширение [учетной записи Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account) для работы со своими реестрами контейнеров Azure. Извлекайте и отправляйте образы в реестр контейнеров Azure или запускайте Задачи ACR в Visual Studio Code.


<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/
