---
title: Краткое руководство. Создание образа контейнера по запросу в Azure
description: Узнайте, как использовать команды Реестра контейнеров для быстрого создания, отправки и выполнения образа контейнера Docker по запросу в облаке Azure.
ms.topic: quickstart
ms.date: 09/25/2020
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: c6fe1fc246d112218b492072155175b2db99c8c9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97032955"
---
# <a name="quickstart-build-and-run-a-container-image-using-azure-container-registry-tasks"></a>Краткое руководство. Сборка и запуск образа контейнера с помощью Задач Реестра контейнеров Azure

При работе с этим кратким руководством вы будете использовать команды [Задач Реестра контейнеров Azure][container-registry-tasks-overview], чтобы быстро создавать, отправлять и запускать образ контейнера Docker непосредственно в Azure без локальной установки Docker. Задачи Реестра контейнеров Azure — это набор функций в Реестре контейнеров Azure, которые призваны упростить управление образами контейнеров и их изменение в течение всего жизненного цикла контейнеров. В этом примере показано, как выгрузить внутренний цикл разработки образа контейнера в облако с помощью сборок по запросу, использующих локальный файл Dockerfile. 

После знакомства с этим кратким руководством ознакомьтесь с более сложными функциями Задач Реестра контейнеров Azure с помощью [этих учебников](container-registry-tutorial-quick-task.md). Задачи Реестра контейнеров Azure позволяют автоматизировать создание образов на основе фиксированного кода или обновления базовых образов или проверять несколько контейнеров параллельно, а также использовать другие сценарии. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]
    
- Для работы с этим кратким руководством требуется Azure CLI версии не ниже 2.0.58. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Если у вас еще нет реестра контейнеров, сначала создайте группу ресурсов с помощью команды [az group create][az-group-create]. Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>Создание реестра контейнеров

Создайте реестр контейнеров с помощью команды [az acr create][az-acr-create]. Имя реестра должно быть уникальным в пределах Azure и содержать от 5 до 50 буквенно-цифровых символов. В примере ниже используется имя *myContainerRegistry008*. Замените его уникальным значением.

```azurecli-interactive
az acr create --resource-group myResourceGroup \
  --name myContainerRegistry008 --sku Basic
```

В этом примере создается реестр ценовой категории *Базовый*; это экономный вариант для разработчиков, изучающих Реестр контейнеров Azure. Дополнительные сведения об уровнях служб см. в статье [Уровни служб реестра контейнеров][container-registry-skus].

## <a name="build-and-push-image-from-a-dockerfile"></a>Создание и отправка образа с помощью Dockerfile

Используйте Реестр контейнеров Azure для создания и отправки образа. Сначала создайте локальный рабочий каталог, а затем создайте Dockerfile с именем *Dockerfile*, содержащий одну строку: `FROM mcr.microsoft.com/hello-world`. Это простой пример того, как создать образ контейнера Linux на основе образа `hello-world`, размещенного в Реестре контейнеров Майкрософт. Вы можете создавать собственные стандартные образы Dockerfile и образы сборки для других платформ. Если вы работаете в оболочке bash, создайте Dockerfile с помощью следующей команды:

```bash
echo FROM mcr.microsoft.com/hello-world > Dockerfile
```

Выполните команду [az acr build][az-acr-build], которая создает образ и отправляет его в реестр. В примере ниже показаны создание и отправка образа `sample/hello-world:v1`. Символ `.` в конце команды задает расположение Dockerfile, в данном случае это текущий каталог.

```azurecli-interactive
az acr build --image sample/hello-world:v1 \
  --registry myContainerRegistry008 \
  --file Dockerfile . 
```

Результат успешной сборки и перенос образа может иметь следующий вид:

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/build_archive_b0bc1e5d361b44f0833xxxx41b78c24e.tar.gz'...
Sending context (1.856 KiB) to registry: mycontainerregistry008...
Queued a build with ID: ca8
Waiting for agent...
2019/03/18 21:56:57 Using acb_vol_4c7ffa31-c862-4be3-xxxx-ab8e615c55c4 as the home volume
2019/03/18 21:56:57 Setting up Docker configuration...
2019/03/18 21:56:58 Successfully set up Docker configuration
2019/03/18 21:56:58 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/18 21:56:59 Executing step ID: build. Working directory: '', Network: ''
2019/03/18 21:56:59 Obtaining source code and scanning for dependencies...
2019/03/18 21:57:00 Successfully obtained source code and scanned for dependencies
2019/03/18 21:57:00 Launching container with name: build
Sending build context to Docker daemon  13.82kB
Step 1/1 : FROM mcr.microsoft.com/hello-world
latest: Pulling from hello-world
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586fxxxx21577a99efb77324b0fe535
Successfully built fce289e99eb9
Successfully tagged mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:01 Successfully executed container: build
2019/03/18 21:57:01 Executing step ID: push. Working directory: '', Network: ''
2019/03/18 21:57:01 Pushing image: mycontainerregistry008.azurecr.io/sample/hello-world:v1, attempt 1
The push refers to repository [mycontainerregistry008.azurecr.io/sample/hello-world]
af0b15c8625b: Preparing
af0b15c8625b: Layer already exists
v1: digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a size: 524
2019/03/18 21:57:03 Successfully pushed image: mycontainerregistry008.azurecr.io/sample/hello-world:v1
2019/03/18 21:57:03 Step ID: build marked as successful (elapsed time in seconds: 2.543040)
2019/03/18 21:57:03 Populating digests for step ID: build...
2019/03/18 21:57:05 Successfully populated digests for step ID: build
2019/03/18 21:57:05 Step ID: push marked as successful (elapsed time in seconds: 1.473581)
2019/03/18 21:57:05 The following dependencies were found:
2019/03/18 21:57:05
- image:
    registry: mycontainerregistry008.azurecr.io
    repository: sample/hello-world
    tag: v1
    digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: v1
    digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
  git: {}

Run ID: ca8 was successful after 10s
```

## <a name="run-the-image"></a>Запуск образа

Теперь быстро запустите созданный и переданный в реестр образ. Здесь вы используете [az acr run][az-acr-run] для запуска команды контейнера. В рабочем процессе разработки контейнера можно реализовать этап проверки перед развертыванием образа. Также можно включить команду в [многошаговый YAML-файл][container-registry-tasks-multi-step]. 

В следующем примере `$Registry` используется для указания реестра, в котором выполняется команда.

```azurecli-interactive
az acr run --registry myContainerRegistry008 \
  --cmd '$Registry/sample/hello-world:v1' /dev/null
```

Параметр `cmd` в этом примере охватывает запуск контейнера в конфигурации по умолчанию, но `cmd` поддерживает дополнительные параметры `docker run` или другие команды `docker`.

Результат аналогичен приведенному ниже:

```console
Packing source code into tar to upload...
Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
Sending context (1.855 KiB) to registry: mycontainerre...
Queued a run with ID: cab
Waiting for an agent...
2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
2019/03/19 19:01:53 Setting up Docker configuration...
2019/03/19 19:01:54 Successfully set up Docker configuration
2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
2019/03/19 19:01:55 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

2019/03/19 19:01:56 Successfully executed container: acb_step_0
2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)

Run ID: cab was successful after 6s
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Ненужные группу ресурсов, реестр контейнеров и все образы контейнеров можно удалить с помощью команды [az group delete][az-group-delete].

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы использовали функции Задач Реестра контейнеров Azure, чтобы быстро создавать, передавать и запускать образ контейнера Docker непосредственно в Azure без установки локальной системы Docker. Дополнительные сведения об использовании Задач Реестра контейнеров Azure для автоматизации создания и обновления образов, см. в руководствах по Задачам Реестра контейнеров Azure.

> [!div class="nextstepaction"]
> [Руководство. Создание и развертывание образов контейнера в облаке с помощью службы "Задачи Реестра контейнеров Azure"][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[azure-account]: https://azure.microsoft.com/free/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tasks-overview]: container-registry-tasks-overview.md
[container-registry-tasks-multi-step]: container-registry-tasks-multi-step.md
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[azure-cli-install]: /cli/azure/install-azure-cli
