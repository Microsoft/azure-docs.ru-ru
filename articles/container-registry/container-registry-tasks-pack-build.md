---
title: Создание образа с помощью собственного Буилдпакк в облаке
description: Используйте команду AZ Dockerfile Pack Build, чтобы создать образ контейнера из приложения и отправить его в реестр контейнеров Azure без использования.
ms.topic: article
ms.date: 10/24/2019
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 264fc7314c78088ebfefb9ddb8edbe38fa16581a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92736623"
---
# <a name="build-and-push-an-image-from-an-app-using-a-cloud-native-buildpack"></a>Сборка и отправка образа из приложения с помощью собственного облачного Буилдпакк

Команда Azure CLI `az acr pack build` использует [`pack`](https://github.com/buildpack/pack) средство CLI из [буилдпаккс](https://buildpacks.io/), чтобы создать приложение и отправить его образ в реестр контейнеров Azure. Эта функция предоставляет возможность быстро создать образ контейнера из исходного кода приложения на Node.js, Java и других языках без определения Dockerfile.

Для выполнения примеров, описанных в этой статье, можно использовать Azure Cloud Shell или локальную установку Azure CLI. Если вы хотите использовать его локально, требуется версия 2.0.70 или более поздняя. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

> [!IMPORTANT]
> Эта функция в настоящее время находится на стадии предварительной версии. Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="use-the-build-command"></a>Использование команды Build

Чтобы создать и отправить образ контейнера с помощью машинного кода Буилдпаккс, выполните команду [AZ запись пакета сборки][az-acr-pack-build] . В то время как команда [AZ запись контроля][az-acr-build] доступа создает и отправляет изображение из источника Dockerfile и связанного с ним кода, `az acr pack build` при этом непосредственно указывается дерево источника приложения.

Как минимум при запуске `az acr pack build` необходимо указать следующее:

* Реестр контейнеров Azure, в котором выполняется команда
* Имя и тег изображения для полученного изображения
* Одно из [поддерживаемых контекстных расположений](container-registry-tasks-overview.md#context-locations) для задач записи контроля доступа, таких как локальный каталог, репозиторий GitHub или удаленный tarball.
* Имя изображения построителя Буилдпакк, которое подходит для вашего приложения. Реестр контейнеров Azure кэширует образы построителя, например `cloudfoundry/cnb:0.0.34-cflinuxfs3` для ускорения сборок.  

`az acr pack build` поддерживает другие функции команд задач ACR, включая [переменные запуска](container-registry-tasks-reference-yaml.md#run-variables) и [журналы выполнения задач](container-registry-tasks-logs.md), которые передаются в потоке и также сохраняются для последующего извлечения.

## <a name="example-build-nodejs-image-with-cloud-foundry-builder"></a>Пример. сборка образа Node.js с помощью построителя Cloud Foundry

В следующем примере создается образ контейнера из Node.js приложения в репозитории [Azure-Samples/NodeJS-документация-Hello-World](https://github.com/Azure-Samples/nodejs-docs-hello-world) с помощью `cloudfoundry/cnb:0.0.34-cflinuxfs3` конструктора. Этот построитель кэшируется реестром контейнеров Azure, поэтому `--pull` параметр не требуется:

```azurecli
az acr pack build \
    --registry myregistry \
    --image {{.Run.Registry}}/node-app:1.0 \
    --builder cloudfoundry/cnb:0.0.34-cflinuxfs3 \
    https://github.com/Azure-Samples/nodejs-docs-hello-world.git
```

В этом примере `node-app` образ создается с помощью `1.0` тега и отправляется в реестр контейнеров *myregistry* . В этом примере имя целевого реестра явно добавляется в начало имени образа. Если не указано, имя сервера входа в реестр автоматически добавляется в начало имени образа.

Выходные данные команды показывают ход создания и отправки образа. 

После успешной сборки образа его можно запустить с помощью DOCKER, если он установлен. Сначала войдите в реестр:

```azurecli
az acr login --name myregistry
```

Запустите образ:

```console
docker run --rm -p 1337:1337 myregistry.azurecr.io/node-app:1.0
```

Перейдите в `localhost:1337` свой любимый браузер, чтобы просмотреть пример веб-приложения. Нажмите `[Ctrl]+[C]` , чтобы прерывать контейнер.

## <a name="example-build-java-image-with-heroku-builder"></a>Пример. сборка образа Java с помощью Heroku Builder

В следующем примере создается образ контейнера из приложения Java в репозитории [буилдпакк/Sample-Java-App](https://github.com/buildpack/sample-java-app) с помощью `heroku/buildpacks:18` конструктора. `--pull`Параметр указывает, что команда должна извлечь Последнее изображение построителя. 

```azurecli
az acr pack build \
    --registry myregistry \
    --image java-app:{{.Run.ID}} \
    --pull --builder heroku/buildpacks:18 \
    https://github.com/buildpack/sample-java-app.git
```

В этом примере выполняется сборка `java-app` образа, помеченного идентификатором запуска команды, и его отправка в реестр контейнеров *myregistry* .

Выходные данные команды показывают ход создания и отправки образа. 

После успешной сборки образа его можно запустить с помощью DOCKER, если он установлен. Сначала войдите в реестр:

```azurecli
az acr login --name myregistry
```

Запустите образ, подставив тег Image для *RunID*:

```console
docker run --rm -p 8080:8080 myregistry.azurecr.io/java-app:runid
```

Перейдите в `localhost:8080` свой любимый браузер, чтобы просмотреть пример веб-приложения. Нажмите `[Ctrl]+[C]` , чтобы прерывать контейнер.


## <a name="next-steps"></a>Дальнейшие действия

После сборки и отправки образа контейнера с помощью `az acr pack build` его можно развернуть как любой образ в выбранном целевом объекте. Варианты развертывания Azure включают в себя запуск в [службе приложений](../app-service/tutorial-custom-container.md) или [Azure Kubernetes Service](../aks/tutorial-kubernetes-deploy-cluster.md), а также другие.

Дополнительные сведения о функциях задач записи контроля доступа см. [в статье Автоматизация создания образов контейнеров и обслуживание с помощью задач контроля учетных](container-registry-tasks-overview.md)записей.


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr/task
[az-acr-pack-build]: /cli/azure/acr/pack#az-acr-pack-build
