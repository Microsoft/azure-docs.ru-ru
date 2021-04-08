---
title: Руководство по Kubernetes в Azure. Обновление приложения
description: В этом руководстве по Службе Azure Kubernetes (AKS) вы узнаете, как обновить имеющееся развертывание приложения для AKS с помощью новой версии кода приложения.
services: container-service
ms.topic: tutorial
ms.date: 01/12/2021
ms.custom: mvc
ms.openlocfilehash: b969e3ec1c670c0a12129289c8ff7eb81df51ff9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98250661"
---
# <a name="tutorial-update-an-application-in-azure-kubernetes-service-aks"></a>Руководство. Обновление приложения в службе Azure Kubernetes

После развертывания приложения в Kubernetes его можно обновить, указав новый образ контейнера или версию образа. Обновление выполняется поэтапно, поэтому одновременно обновляется только часть развертывания. Такое поэтапное обновление позволяет приложению продолжать работать во время обновления. Оно также обеспечивает механизм отката на случай, если произойдет сбой развертывания.

В этом руководстве (часть 7 из 8) обновляется пример приложения Vote Azure. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Обновление кода внешнего приложения.
> * Создание обновленного образа контейнера.
> * Передача образа контейнера в Реестр контейнеров Azure.
> * Развертывание обновленного образа контейнера.

## <a name="before-you-begin"></a>Перед началом

В предыдущих руководствах приложение было упаковано в образ контейнера. Этот образ был передан в Реестр контейнеров Azure, и вы создали кластер AKS. Затем приложение было развернуто в кластере AKS.

Кроме того, был клонирован репозиторий приложения, включая исходный код приложения и предварительно созданный файл Docker Compose, используемый в этом руководстве. Проверьте, создали ли вы клон репозитория и изменили ли каталоги на клонированный каталог. Если вы не выполнили эти действия и хотите продолжить изучение материала, начните с первого раздела руководства по [подготовке приложения для Службы Azure Kubernetes (AKS)][aks-tutorial-prepare-app].

Для выполнения задач из этого руководства требуется Azure CLI 2.0.53 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="update-an-application"></a>Обновление приложения

Внесите изменения в пример приложения, а затем обновите уже развернутую версию в кластере AKS. Перейдите в клонированный каталог *azure-voting-app-redis*. Пример исходного кода приложения можно найти в каталоге *azure-vote*. Откройте файл *config_file.cfg* с помощью редактора, например `vi`:

```console
vi azure-vote/azure-vote/config_file.cfg
```

Измените значения параметров *VOTE1VALUE* и *VOTE2VALUE* на другие значения, например цвета. В следующем примере показаны обновленные значения:

```
# UI Configurations
TITLE = 'Azure Voting App'
VOTE1VALUE = 'Blue'
VOTE2VALUE = 'Purple'
SHOWHOST = 'false'
```

Сохраните файл и закройте его. В `vi` используйте команду `:wq`.

## <a name="update-the-container-image"></a>Обновление образа контейнера

Используйте команду [docker-compose][docker-compose] для повторного создания образа внешнего приложения и проверки обновленного приложения. Аргумент `--build` указывает Docker Compose, что требуется повторно создать образ приложения.

```console
docker-compose up --build -d
```

## <a name="test-the-application-locally"></a>Локальное тестирование приложения

Чтобы убедиться, что обновленный образ контейнера показывает изменения, откройте в локальном веб-браузере `http://localhost:8080`.

:::image type="content" source="media/container-service-kubernetes-tutorials/vote-app-updated.png" alt-text="Снимок экрана: пример обновленного образа контейнера приложения Azure для голосования, выполняемого локально и открытого в веб-браузере.":::

Обновленные значения, заданные в файле *config_file.cfg*, отображаются в запущенном приложении.

## <a name="tag-and-push-the-image"></a>Пометка и отправка образов

Чтобы правильно использовать этот обновленный образ, добавьте к образу *azure-vote-front* имя сервера для входа реестра ACR в виде тега. Получите имя сервера для входа, выполнив команду [az acr list](/cli/azure/acr).

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

Используйте команду [docker tag][docker-tag], чтобы добавить тег для образа. Замените `<acrLoginServer>` именем сервера для входа ACR или именем узла общедоступного реестра и обновите версию образа до *:v2* следующим образом:

```console
docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 <acrLoginServer>/azure-vote-front:v2
```

Теперь используйте команду [docker push][docker-push], чтобы передать образ в реестр. Замените `<acrLoginServer>` именем сервера для входа ACR.

> [!NOTE]
> Если возникают проблемы с отправкой данных в реестр ACR, убедитесь, что вход в систему по-прежнему выполнен. Выполните команду [az acr login][az-acr-login], используя имя Реестра контейнеров Azure, который вы создали на шаге [создания Реестра контейнеров Azure](tutorial-kubernetes-prepare-acr.md#create-an-azure-container-registry). Например, `az acr login --name <azure container registry name>`.

```console
docker push <acrLoginServer>/azure-vote-front:v2
```

## <a name="deploy-the-updated-application"></a>Развертывание обновленного приложения

Чтобы обеспечить максимальное время доступности, должны быть запущены несколько экземпляров группы контейнеров приложения. Проверьте число запущенных экземпляров внешнего интерфейса с помощью команды [kubectl get pods][kubectl-get]:

```
$ kubectl get pods

NAME                               READY     STATUS    RESTARTS   AGE
azure-vote-back-217588096-5w632    1/1       Running   0          10m
azure-vote-front-233282510-b5pkz   1/1       Running   0          10m
azure-vote-front-233282510-dhrtr   1/1       Running   0          10m
azure-vote-front-233282510-pqbfk   1/1       Running   0          10m
```

Если у вас нет нескольких модулей pod внешнего интерфейса, измените масштаб развертывания *azure-vote-front* следующим образом:

```console
kubectl scale --replicas=3 deployment/azure-vote-front
```

Чтобы обновить приложение, используйте команду [kubectl set][kubectl-set]. Обновите `<acrLoginServer>`, используя имя сервера для входа или имя узла реестра контейнеров, и укажите версию приложения *v2*:

```console
kubectl set image deployment azure-vote-front azure-vote-front=<acrLoginServer>/azure-vote-front:v2
```

Для мониторинга развертывания используйте команду [kubectl get pod][kubectl-get]. По мере развертывания обновленного приложения ваши группы контейнеров прекращают работу и воссоздаются с новым образом контейнера.

```console
kubectl get pods
```

В следующем примере выходных данных показано завершение работы модулей pod и запуск новых экземпляров в ходе развертывания:

```
$ kubectl get pods

NAME                               READY     STATUS        RESTARTS   AGE
azure-vote-back-2978095810-gq9g0   1/1       Running       0          5m
azure-vote-front-1297194256-tpjlg  1/1       Running       0          1m
azure-vote-front-1297194256-tptnx  1/1       Running       0          5m
azure-vote-front-1297194256-zktw9  1/1       Terminating   0          1m
```

## <a name="test-the-updated-application"></a>Проверка обновленного приложения

Чтобы просмотреть обновленное приложение, сначала нужно получить внешний IP-адрес службы `azure-vote-front`:

```console
kubectl get service azure-vote-front
```

Теперь откройте веб-браузер и введите IP-адрес вашей службы:

:::image type="content" source="media/container-service-kubernetes-tutorials/vote-app-updated-external.png" alt-text="Снимок экрана: пример обновленного образа приложения Azure для голосования, выполняемого в кластере AKS и открытого в веб-браузере.":::

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы обновили приложение и развернули это обновление в кластер AKS. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Обновление кода внешнего приложения.
> * Создание обновленного образа контейнера.
> * Передача образа контейнера в Реестр контейнеров Azure.
> * Развертывание обновленного образа контейнера.

Перейдите к следующему руководству, чтобы узнать, как обновить кластер AKS до новой версии Kubernetes.

> [!div class="nextstepaction"]
> [Обновление Kubernetes в Службе контейнеров Azure (AKS)][aks-tutorial-upgrade]

<!-- LINKS - external -->
[docker-compose]: https://docs.docker.com/compose/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-set]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#set

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-upgrade]: ./tutorial-kubernetes-upgrade-cluster.md
[az-acr-login]: /cli/azure/acr
[azure-cli-install]: /cli/azure/install-azure-cli
