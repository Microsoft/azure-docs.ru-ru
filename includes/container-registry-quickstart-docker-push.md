---
title: включить файл
description: включить файл
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 08/04/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 404fd10c3e3610671d2b6e5dbc6aba8bcaa70046
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100019053"
---
## <a name="push-image-to-registry"></a>Отправка образа в реестр

Чтобы отправить образ в реестр контейнеров Azure, сначала нужно получить этот образ. Если у вас еще нет образов локального контейнера, используйте следующую команду [docker pull][docker-pull], чтобы извлечь существующий общедоступный образ. Для этого примера извлеките образ `hello-world` из Реестра контейнеров Майкрософт.

```
docker pull mcr.microsoft.com/hello-world
```

Прежде чем отправить образ в реестр, нужно добавить в него тег с полным именем сервера входа для реестра. Имя сервера для входа имеет формат *\<registry-name\>.azurecr.io* (должно быть в нижнем регистре), например *mycontainerregistry.azurecr.io*.

Присвойте образу тег с помощью команды [docker tag][docker-tag]. Замените значение `<login-server>` именем сервера входа для экземпляра ACR.

```
docker tag mcr.microsoft.com/hello-world <login-server>/hello-world:v1
```

Пример

```
docker tag mcr.microsoft.com/hello-world mycontainerregistry.azurecr.io/hello-world:v1
```


Наконец, воспользуйтесь командой [docker push][docker-push] для принудительной отправки образа в экземпляр реестра. Замените значение `<login-server>` именем сервера входа для экземпляра реестра. В этом примере создается репозиторий **hello-world** с образом `hello-world:v1`.

```
docker push <login-server>/hello-world:v1
```

После передачи образа в реестр контейнеров удалите образ `hello-world:v1` из локальной среды Docker. (Обратите внимание, что команда [docker rmi][docker-rmi] не приводит к удалению образа из репозитория **hello-world** в реестре контейнеров Azure.)

```
docker rmi <login-server>/hello-world:v1
```

<!-- LINKS - External -->
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/

<!-- LINKS - Internal -->

