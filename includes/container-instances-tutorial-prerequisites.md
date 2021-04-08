---
title: включить файл
description: включить файл
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 03/20/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 10fb9e8169b7f4159ccbf4a0ff36021f6033f811
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "75552429"
---
Для работы с этим руководством вам потребуются следующие ресурсы:

**Azure CLI.** Интерфейс командной строки Azure версии 2.0.29 или более поздней, установленный на локальном компьютере. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI][azure-cli-install].

**Docker**. Для выполнения действий, описанных в этом учебнике, требуется базовое понимание таких основных понятий Docker, как контейнеры и образы контейнеров, а также знание основных команд `docker`. Ознакомьтесь с [общими сведениями о Docker и контейнерах][docker-get-started].

**Docker**. Для работы с этим руководством требуется установить Docker локально. Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS][docker-mac], [Windows][docker-windows] и [Linux][docker-linux].

> [!IMPORTANT]
> Так как Azure Cloud Shell не включает управляющую программу Docker, для работы с этим руководством *необходимо* установить среду разработки Azure CLI и модуль Docker на свой *локальный компьютер*. В этом руководстве Azure Cloud Shell не используется.

<!-- LINKS - External -->
[docker-get-started]: https://docs.docker.com/engine/docker-overview/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
