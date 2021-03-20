---
title: Как готовится проект для Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Описание подготовки проекта с помощью Azure Dev Spaces
keywords: аздс. YAML, Azure Dev Spaces, пространства разработки, Docker, Kubernetes, Azure, AKS, служба Azure Kubernetes, контейнеры
ms.openlocfilehash: d2da69dd8a8c2683ff584dfd0ffc61cb023f2ece
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91968163"
---
# <a name="how-preparing-a-project-for-azure-dev-spaces-works"></a>Как готовится проект для Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces предоставляет несколько способов быстро перебора и отладки приложений Kubernetes, а также совместной работы в кластере Службы Azure Kubernetes (AKS). Пространства разработки могут создавать диаграммы файлы dockerfile и Helm для проекта. Пространства разработки также создают и используют файл конфигурации для развертывания, запуска и отладки приложений Kubernetes в AKS. Все эти файлы находятся в коде приложения и могут быть добавлены в систему управления версиями.

В этой статье описывается, что происходит при подготовке проекта к работе в AKS с пространствами разработки.

## <a name="prepare-your-code"></a>Подготовка кода

Чтобы запустить приложение в пространстве разработки, оно должно быть контейнерным и необходимо определить, как оно должно быть развернуто в Kubernetes. Чтобы контейнеризовать приложение, вам потребуется Dockerfile. Чтобы определить, как приложение развертывается в Kubernetes, требуется [Диаграмма Helm](https://docs.helm.sh/). Чтобы упростить создание диаграммы Dockerfile и Helm для приложения, клиентские средства предоставляют `prep` команду:

```cmd
azds prep --enable-ingress
```

`prep`Команда выполнит просмотр файлов в проекте и попытается создать диаграмму Dockerfile и Helm для запуска приложения в Kubernetes. В настоящее время `prep` команда создаст диаграмму Dockerfile и Helm со следующими языками:

* Java
* Node.js
* .NET Core

*Необходимо* выполнить `prep` команду из каталога, содержащего исходный код. Выполнение `prep` команды из правильного каталога позволяет клиентским инструментам обнаруживать язык и создавать соответствующие Dockerfile для контейнеризовать приложения. Можно также выполнить `prep` команду из каталога, содержащего файл *pom.xml* для проектов Java.

Если выполнить `prep` команду из каталога, который не содержит исходный код, средства на стороне клиента не будут создавать Dockerfile. Кроме того, будет выведено сообщение об ошибке: *Dockerfile не удалось создать из-за неподдерживаемого языка*. Эта ошибка также возникает, если клиентские средства не распознают тип проекта.

При выполнении `prep` команды можно указать `--enable-ingress` флаг. Этот флаг указывает контроллеру создать конечную точку, доступную в Интернете, для этой службы. Если этот флаг не указан, служба доступна только в пределах кластера или с помощью туннеля localhost, созданного клиентскими средствами. Вы можете включить или отключить это поведение после выполнения `prep` команды, обновив созданную диаграмму Helm.

`prep`Команда не заменит существующие диаграммы файлы dockerfile или Helm в вашем проекте. Если существующая диаграмма Dockerfile или Helm использует те же соглашения об именовании, что и файлы `prep` , созданные командой, `prep` команда пропустит создание этих файлов. В противном случае `prep` команда создаст собственную диаграмму Dockerfile или Helm вместе с существующими файлами.

> [!IMPORTANT]
> Azure Dev Spaces использует диаграмму Dockerfile и Helm для создания и выполнения кода проекта, но эти файлы можно изменить, если нужно изменить способ построения и запуска проекта.

`prep`Команда также создаст `azds.yaml` файл в корне проекта. Azure Dev Spaces использует этот файл для создания, установки, настройки и запуска приложения. Этот файл конфигурации содержит сведения о расположении диаграммы Dockerfile и Helm, а также дополнительную настройку на основе этих артефактов.

Ниже приведен пример файла аздс. YAML, созданного с помощью [примера приложения .NET Core](https://github.com/Azure/dev-spaces/tree/master/samples/dotnetcore/getting-started/webfrontend).

```yaml
kind: helm-release
apiVersion: 1.1
build:
  context: .
  dockerfile: Dockerfile
install:
  chart: charts/webfrontend
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
  set:
    replicaCount: 1
    image:
      repository: webfrontend
      tag: $(tag)
      pullPolicy: Never
    ingress:
      annotations:
        kubernetes.io/ingress.class: traefik-azds
      hosts:
        # This expands to [space.s.][rootSpace.]webfrontend.<random suffix>.<region>.azds.io
        # Customize the public URL by changing the 'webfrontend' text between the $(rootSpacePrefix) and $(hostSuffix) tokens
        # For more information see https://aka.ms/devspaces/routing
        - $(spacePrefix)$(rootSpacePrefix)webfrontend$(hostSuffix)
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
    container:
      sync:
      - "**/Pages/**"
      - "**/Views/**"
      - "**/wwwroot/**"
      - "!**/*.{sln,csproj}"
      command: [dotnet, run, --no-restore, --no-build, --no-launch-profile, -c, "${BUILD_CONFIGURATION:-Debug}"]
      iterate:
        processesToKill: [dotnet, vsdbg]
        buildCommands:
        - [dotnet, build, --no-restore, -c, "${BUILD_CONFIGURATION:-Debug}"]
```

`azds.yaml`Файл, созданный командой, `prep` предназначен для простого сценария разработки с одним проектом. Если в конкретном проекте увеличилась сложность, может потребоваться обновить этот файл после выполнения `prep` команды. Например, проект может потребовать внесения некоторых изменений в процесс сборки или запуска в зависимости от потребностей разработки или отладки. Кроме того, в проекте может быть несколько приложений, для которых требуется несколько процессов сборки или другое содержимое сборки.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о выполнении кода в сфере разработки см. в статье [как выполнять код с помощью Azure dev Spaces Works][how-it-works-up].

[how-it-works-up]: how-dev-spaces-works-up.md