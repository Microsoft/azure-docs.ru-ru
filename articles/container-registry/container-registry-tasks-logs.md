---
title: Просмотр журналов выполнения задач — задачи
description: Просмотр журналов выполнения, созданных задачами контроля доступа, и управление ими.
ms.topic: article
ms.date: 03/09/2020
ms.openlocfilehash: b2a10d4a3a2746acf38445673af994c6317c77de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93027182"
---
# <a name="view-and-manage-task-run-logs"></a>Просмотр журналов выполнения задач и управление ими

Каждая задача, выполняемая в [задачах реестра контейнеров Azure](container-registry-tasks-overview.md) , создает выходные данные журнала, которые можно проверить, чтобы определить, успешно ли выполнялись шаги задачи. 

В этой статье объясняется, как просматривать журналы выполнения задач и управлять ими.

## <a name="view-streamed-logs"></a>Просмотр потоковых журналов

При ручном запуске задачи выходные данные журнала передаются непосредственно в консоль. Например, если вы запускаете задачу вручную с помощью команды [AZ запись контроля](/cli/azure/acr#az-acr-build)доступа, [AZ для запуска](/cli/azure/acr#az-acr-run)записи или [AZ контроля](/cli/azure/acr/task#az-acr-task-run) доступа, вы видите вывод журнала в консоль. 

В следующем примере [AZ запись контроля доступа запускается](/cli/azure/acr#az-acr-run) вручную задача, которая запускает контейнер, извлеченный из того же реестра:

```azurecli
az acr run --registry mycontainerregistry1220 \
  --cmd '$Registry/samples/hello-world:v1' /dev/null
```

Потоковый журнал:

```console
Queued a run with ID: cf4
Waiting for an agent...
2020/03/09 20:30:10 Alias support enabled for version >= 1.1.0, please see https://aka.ms/acr/tasks/task-aliases for more information.
2020/03/09 20:30:10 Creating Docker network: acb_default_network, driver: 'bridge'
2020/03/09 20:30:10 Successfully set up Docker network: acb_default_network
2020/03/09 20:30:10 Setting up Docker configuration...
2020/03/09 20:30:11 Successfully set up Docker configuration
2020/03/09 20:30:11 Logging in to registry: mycontainerregistry1220azurecr.io
2020/03/09 20:30:12 Successfully logged into mycontainerregistry1220azurecr.io
2020/03/09 20:30:12 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2020/03/09 20:30:12 Launching container with name: acb_step_0
Unable to find image 'mycontainerregistry1220azurecr.io/samples/hello-world:v1' locally
v1: Pulling from samples/hello-world
Digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e888a
Status: Downloaded newer image for mycontainerregistry1220azurecr.io/samples/hello-world:v1

Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]

2020/03/09 20:30:13 Successfully executed container: acb_step_0
2020/03/09 20:30:13 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 1.180081)

Run ID: cf4 was successful after 5s
```

## <a name="view-stored-logs"></a>Просмотр сохраненных журналов 

Реестр контейнеров Azure хранит журналы выполнения для всех задач. Сохраненные журналы запуска можно просмотреть в портал Azure. Чтобы просмотреть выбранный журнал, можно также использовать команду [AZ контроля](/cli/azure/acr/task#az-acr-task-logs) доступа к журналам задач. По умолчанию журналы хранятся в течение 30 дней.

Если задача запускается автоматически, например при обновлении исходного кода, доступ к сохраненным журналам является *единственным* способом просмотра журналов выполнения. Автоматические триггеры задач включают фиксации исходного кода или запросы на включение внесенных изменений, базовые обновления образа и триггеры таймера.

Чтобы просмотреть журналы выполнения на портале, выполните следующие действия.

1. Перейдите к реестру контейнеров.
1. В окне **службы** выберите пункт   >  **выполняемые** задачи.
1. Выберите **идентификатор запуска** , чтобы просмотреть состояние выполнения и журналы выполнения. Журнал содержит те же сведения, что и потоковый журнал, если он создан.

![Просмотр портала запуска для входа задач](./media/container-registry-tasks-logs/portal-task-run-logs.png)

Чтобы просмотреть журнал с помощью Azure CLI, выполните команду [AZ запись контроля](/cli/azure/acr/task#az-acr-task-logs) доступа и укажите идентификатор запуска, имя задачи или конкретный образ, созданный задачей сборки. Если указано имя задачи, команда отображает журнал для последнего созданного запуска.

В следующем примере выводится журнал для запуска с ИДЕНТИФИКАТОРом *CF4*:

```azurecli
az acr task logs --registry mycontainerregistry1220 \
  --run-id cf4
```

## <a name="alternative-log-storage"></a>Альтернативное хранилище журналов

Вы можете хранить журналы выполнения задач в локальной файловой системе или использовать альтернативное решение архивации, такое как служба хранилища Azure.

Например, создайте локальный каталог *тасклогс* и перенаправьте выходные данные [журналов задач AZ контроля](/cli/azure/acr/task#az-acr-task-logs) доступа в локальный файл:

```azurecli
mkdir ~/tasklogs

az acr task logs --registry mycontainerregistry1220 \
  --run-id cf4 > ~/tasklogs/cf4.log
```

Кроме того, можно сохранять локальные файлы журналов в службе хранилища Azure. Например, используйте [Azure CLI](../storage/blobs/storage-quickstart-blobs-cli.md), [портал Azure](../storage/blobs/storage-quickstart-blobs-portal.md)или другие методы для отправки файлов в учетную запись хранения.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [задачах реестра контейнеров Azure](container-registry-tasks-overview.md)


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-pack-build]: /cli/azure/acr/pack#az-acr-pack-build
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-update]: /cli/azure/acr/task#az-acr-task-update
[az-login]: /cli/azure/reference-index#az-login
[az-login-service-principal]: /cli/azure/authenticate-azure-cli

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
