---
title: Создание образов контейнеров в Service Fabric в Azure
description: Из этого руководства вы узнаете, как создавать образы контейнеров для многоконтейнерного приложения Service Fabric.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 31b5f870465bc1dff9d6ff7827a4efed084bcf62
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92739056"
---
# <a name="tutorial-create-container-images-on-a-linux-service-fabric-cluster"></a>Руководство. Создание образов контейнеров в кластере Service Fabric в Linux

Это руководство представляет собой первую часть цикла руководств, посвященного использованию контейнеров в кластере Service Fabric на платформе Linux. В этом руководстве выполняется подготовка многоконтейнерного приложения к использованию в Service Fabric. В последующих руководствах созданные образы используются как часть приложения Service Fabric. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * клонирование исходного кода приложения из GitHub;
> * создание образа контейнера из исходного кода приложения;
> * развертывание экземпляра реестра контейнеров Azure;
> * добавление тегов к образу контейнера для реестра контейнеров Azure;
> * отправка образа в реестр контейнеров Azure.

Из этой серии руководств вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * создание образов контейнеров для Service Fabric;
> * [создание и развертывание приложения Service Fabric с использованием контейнеров](service-fabric-tutorial-package-containers.md);
> * [отработка отказа и масштабирование в Service Fabric](service-fabric-tutorial-containers-failover.md).

## <a name="prerequisites"></a>предварительные требования

* Настроенная среда разработки Linux для Service Fabric. Следуйте [этим](service-fabric-get-started-linux.md) инструкциям по настройке среды Linux.
* Для этого руководства требуется Azure CLI версии 2.0.4 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).
* Кроме того, требуется наличие подписки Azure. Дополнительные сведения о бесплатной пробной версии можно получить [здесь](https://azure.microsoft.com/free/).

## <a name="get-application-code"></a>Получение кода приложения

В этом руководстве используется пример приложения для голосования. Приложение состоит из веб-компонента интерфейсной части и серверного экземпляра Redis. Компоненты упаковываются в образы контейнеров.

Используйте git, чтобы скачать копию приложения в среду разработки.

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/
```

Решение содержит две папки и файл docker-compose.yml. В папке azure-vote содержится интерфейсная служба Python и файл Dockerfile, используемый для создания образа. В каталоге Voting содержится пакет приложения Service Fabric, который развертывается в кластер. Эти каталоги содержат необходимые ресурсы для текущего руководства.

## <a name="create-container-images"></a>Создание образов контейнеров

Выполните указанную ниже команду в каталоге **azure-vote**, чтобы создать образ для веб-компонента интерфейсной части. Эта команда использует файл Dockerfile, расположенный в этом каталоге, для создания образа.

```bash
docker build -t azure-vote-front .
```
> [!Note]
> Если вам отказано в доступе, ознакомьтесь с [этой](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) документацией о том, как работать с Docker без прав sudo.

Для выполнения этой команды может потребоваться некоторое время, так как нужно извлечь все необходимые зависимости из Docker Hub. После завершения выполните команду [docker images](https://docs.docker.com/engine/reference/commandline/images/), чтобы увидеть созданные образы.

```bash
docker images
```

Вы увидите, что были скачаны или созданы два образа. Образ *azure-vote-front* содержит приложение. Он является производным от образа *python* из Docker Hub.

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
azure-vote-front             latest              052c549a75bf        About a minute ago   708MB
tiangolo/uwsgi-nginx-flask   python3.6           590e17342131        5 days ago           707MB

```

## <a name="deploy-azure-container-registry"></a>Развертывание реестра контейнеров Azure

Сначала выполните команду **az login**, чтобы войти в учетную запись Azure.

```azurecli
az login
```

Затем введите команду **az account**, чтобы выбрать подписку для создания реестра контейнеров Azure. Вместо <subscription_id> нужно указать идентификатор подписки Azure.

```azurecli
az account set --subscription <subscription_id>
```

При развертывании реестра контейнеров Azure сначала необходимо создать группу ресурсов. Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды **az group create**. В этом примере создается группа ресурсов *myResourceGroup* в регионе *westus*.

```azurecli
az group create --name <myResourceGroup> --location westus
```

Создайте реестр контейнеров Azure с помощью команды **az acr create**. Замените \<acrName> именем реестра контейнеров, который нужно создать в подписке. Это имя может включать только буквы и цифры и должно быть уникальным.

```azurecli
az acr create --resource-group <myResourceGroup> --name <acrName> --sku Basic --admin-enabled true
```

В остальной части этого руководства acrName будет заменять в примерах имя реестра контейнеров. Запишите это значение.

## <a name="sign-in-to-your-container-registry"></a>Вход в реестр контейнеров

Войдите в свой экземпляр реестра контейнеров Azure, прежде чем отправлять в него образы. Используйте команду **az acr login**, чтобы выполнить операцию. Укажите уникальное имя реестра контейнеров, заданное при его создании.

```azurecli
az acr login --name <acrName>
```

После выполнения эта команда возвращает сообщение Login Succeeded (Вход выполнен).

## <a name="tag-container-images"></a>Присвоение тегов образам контейнеров

Каждый образ контейнера должен иметь тег с именем сервера входа (loginServer), указанным для реестра. Данный тег используется для маршрутизации при отправке образов контейнеров в реестр образов.

Чтобы просмотреть список сохраненных образов, используйте команду [docker images](https://docs.docker.com/engine/reference/commandline/images/).

```bash
docker images
```

Выходные данные:

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
azure-vote-front             latest              052c549a75bf        About a minute ago   708MB
tiangolo/uwsgi-nginx-flask   python3.6           590e17342131        5 days ago           707MB
```

Чтобы получить имя loginServer, выполните следующую команду.

```azurecli
az acr show --name <acrName> --query loginServer --output table
```

Эта команда выводит таблицу с приведенными ниже результатами. Этот результат будет использоваться для добавления тега к образу **azure-vote-front**, прежде чем он будет отправлен в реестр контейнеров на следующем шаге.

```output
Result
------------------
<acrName>.azurecr.io
```

Теперь добавьте к образу *azure-vote-front* тег loginServer реестра контейнеров. Кроме того, добавьте `:v1` в конец имени образа. Этот тег обозначает номер версии образа.

```bash
docker tag azure-vote-front <acrName>.azurecr.io/azure-vote-front:v1
```

После пометки выполните команду docker images для проверки операции.

Выходные данные:

```output
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front                       latest              052c549a75bf        23 minutes ago      708MB
<acrName>.azurecr.io/azure-vote-front   v1                  052c549a75bf       23 minutes ago      708MB
tiangolo/uwsgi-nginx-flask             python3.6           590e17342131        5 days ago          707MB

```

## <a name="push-images-to-registry"></a>Отправка образов в реестр

Отправьте образ *azure-vote-front* в реестр. 

Используйте следующий пример, заменив в нем имя loginServer ACR именем loginServer своей среды.

```bash
docker push <acrName>.azurecr.io/azure-vote-front:v1
```

Выполнение команд docker push может занять несколько минут.

## <a name="list-images-in-registry"></a>Перечисление образов в реестре

Чтобы получить список образов, отправленных в реестр контейнеров Azure, выполните команду [az acr repository list](/cli/azure/acr/repository). Укажите в команде имя нужного экземпляра ACR.

```azurecli
az acr repository list --name <acrName> --output table
```

Выходные данные:

```output
Result
----------------
azure-vote-front
```

По завершении работы с этим руководством образ контейнера будет сохранен в частном экземпляре реестра контейнеров Azure. В следующих руководствах мы развернем этот образ из ACR в кластер Service Fabric.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве приложение было извлечено из GitHub, а также были созданы образы контейнеров, которые затем были помещены в реестр. Были выполнены следующие действия:

> [!div class="checklist"]
> * клонирование исходного кода приложения из GitHub;
> * создание образа контейнера из исходного кода приложения;
> * развертывание экземпляра реестра контейнеров Azure;
> * добавление тегов к образу контейнера для реестра контейнеров Azure;
> * отправка образа в реестр контейнеров Azure.

Перейдите к следующему руководству, чтобы узнать об упаковке контейнеров в приложение Service Fabric с помощью Yeoman.

> [!div class="nextstepaction"]
> [Упаковка и развертывание контейнеров в виде приложения Service Fabric](service-fabric-tutorial-package-containers.md)
