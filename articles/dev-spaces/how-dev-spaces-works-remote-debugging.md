---
title: Как выполняется удаленная отладка кода с помощью Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Описание процессов удаленной отладки в службе Kubernetes Azure с помощью Azure Dev Spaces
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Служба Azure Kubernetes, контейнеры
ms.openlocfilehash: 0487b80d23974a66bafe93ee1fbdf9b796d0ab53
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91975048"
---
# <a name="how-remote-debugging-your-code-with-azure-dev-spaces-works"></a>Как выполняется удаленная отладка кода с помощью Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces предоставляет несколько способов быстро перебора и отладки приложений Kubernetes, а также совместной работы в кластере Службы Azure Kubernetes (AKS). После запуска проекта в пространстве разработки Azure Dev Spaces предоставляет способ подключения и отладки работающего приложения в AKS.

В этой статье описывается, как работает удаленная отладка с использованием пространств разработки.

## <a name="debug-your-code"></a>Отладка кода

Для приложений Java, .NET Core и Node.js можно выполнить отладку приложения, работающего непосредственно в пространстве разработки, с помощью Visual Studio Code или Visual Studio. Visual Studio Code и Visual Studio предоставляют средства для подключения к сфере разработки, запуска приложения и подключения отладчика. После выполнения `azds prep` проект можно открыть в Visual Studio Code или Visual Studio. Visual Studio Code или Visual Studio создаст собственные файлы конфигурации для подключения, которые отделены от выполнения `azds prep` . В Visual Studio Code или Visual Studio можно задать точки останова и запустить приложение в пространстве разработки.

![Отладка кода](media/get-started-node/debug-configuration-nodejs2.png)

При запуске приложения с помощью Visual Studio Code или Visual Studio для отладки они управляют запуском и подключением к пространству разработки таким же образом, как и при запуске `azds up` . Кроме того, средства на стороне клиента в Visual Studio Code и Visual Studio предоставляют дополнительный параметр с определенной информацией для отладки. Параметр содержит имя образа отладчика, расположение отладчика в образе отладчика и конечное расположение в контейнере приложения для подключения папки отладчика.

Изображение отладчика автоматически определяется средствами на стороне клиента. В нем используется метод, аналогичный использованному во время работы Dockerfile и Helm диаграмма `azds prep` . После подключения отладчика к образу приложения он запускается с помощью `azds exec` .

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о принципах работы Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Принцип работы Azure Dev Spaces](how-dev-spaces-works.md)
