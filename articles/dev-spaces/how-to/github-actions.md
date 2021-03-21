---
title: Действия GitHub & службы Azure Kubernetes (Предварительная версия)
services: azure-dev-spaces
ms.date: 04/03/2020
ms.topic: conceptual
description: Проверка и проверка изменений в запросе на вытягивание непосредственно в службе Kubernetes Azure с помощью действий GitHub и Azure Dev Spaces
keywords: DOCKER, Kubernetes, Azure, AKS, служба Kubernetes Azure, контейнеры, действия GitHub, Helm, сеть службы, маршрутизация в сети службы, kubectl, K8S
manager: gwallace
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 37ad621609f5a5631b498e55483e5d16e8ac4472
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102202116"
---
# <a name="github-actions--azure-kubernetes-service-preview"></a>Действия GitHub & службы Azure Kubernetes (Предварительная версия)

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces предоставляет рабочий процесс с помощью действий GitHub, позволяющих проверять изменения из запроса на вытягивание непосредственно в AKS перед слиянием запроса на включение внесенных изменений в основную ветвь репозитория. Наличие работающего приложения для просмотра изменений запроса на вытягивание может повысить уверенность как разработчика, так и членов группы. Это работающее приложение также может помочь членам группы, таким как, менеджеры по продуктам и проектировщикам, стать частью процесса проверки на ранних стадиях разработки.

Из этого руководства вы узнаете, как выполнить следующие задачи:

* настройка Azure Dev Spaces в управляемом кластере Kubernetes в Azure;
* развертывание большого приложения с несколькими микрослужбами в пространстве разработки;
* Настройка CI/CD с помощью действий GitHub.
* тестирование отдельных микрослужб в изолированном пространстве разработки в контексте всего приложения.

> [!IMPORTANT]
> Эта функция в настоящее время находится на стадии предварительной версии. Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free).
* [Установленный Azure CLI][azure-cli-installed].
* [Установленная версия Helm 3][helm-installed].
* Учетная запись GitHub с [включенными действиями GitHub][github-actions-beta-signup].
* [Пример приложения для Azure dev Spacesного использования велосипеда](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp/README.md) , работающего в кластере AKS.

## <a name="create-an-azure-container-registry"></a>Создание реестра контейнеров Azure

Создание экземпляра службы "Реестр контейнеров Azure" (ACR).

```azurecli
az acr create --resource-group MyResourceGroup --name <acrName> --sku Basic
```

> [!IMPORTANT]
> Имя записи контроля доступа должно быть уникальным в пределах Azure и содержать 5-50 буквенно-цифровых символов. Все используемые буквы должны быть в нижнем регистре.

Сохраните значение *loginServer* из выходных данных, так как оно используется на более позднем шаге.

## <a name="create-a-service-principal-for-authentication"></a>Создание субъекта-службы для проверки подлинности

Для создания субъекта-службы используйте команду [AZ AD SP Create-для – RBAC][az-ad-sp-create-for-rbac] . Пример:

```azurecli
az ad sp create-for-rbac --sdk-auth --skip-assignment
```

Сохраните выходные данные JSON, так как они используются на более позднем шаге.

Чтобы отобразить *идентификатор* кластера AKS, используйте команду [AZ AKS Показать][az-aks-show] .

```azurecli
az aks show -g MyResourceGroup -n MyAKS  --query id
```

Для отображения *идентификатора* записи контроля доступа используйте команду [AZ запись контроля][az-acr-show] доступа.

```azurecli
az acr show --name <acrName> --query id
```

Используйте команду [AZ Role назначение][az-role-assignment-create] , чтобы предоставить *участнику* доступ к кластеру AKS и *акрпуш* доступ к записи контроля доступа.

```azurecli
az role assignment create --assignee <ClientId> --scope <AKSId> --role Contributor
az role assignment create --assignee <ClientId>  --scope <ACRId> --role AcrPush
```

> [!IMPORTANT]
> Чтобы предоставить субъекту-службе доступ к этим ресурсам, необходимо быть владельцем кластера AKS и записи контроля доступа.

## <a name="configure-your-github-action"></a>Настройка действия GitHub

> [!IMPORTANT]
> Для вашего репозитория должны быть включены действия GitHub. Чтобы включить действия GitHub для репозитория, перейдите в репозиторий на сайте GitHub, перейдите на вкладку действия и выберите параметр Включить действия для этого репозитория.

Перейдите к разветвленному репозиторию и щелкните *Параметры*. Щелкните *секреты* в левой боковой панели. Щелкните *Добавить новый секрет* , чтобы добавить каждый новый секрет, приведенный ниже.

1. *AZURE_CREDENTIALS*— все выходные данные создания субъекта-службы.
1. *RESOURCE_GROUP*: Группа ресурсов для кластера AKS, в данном примере — *MyResourceGroup*.
1. *CLUSTER_NAME*: имя кластера AKS, в данном примере — *мякс*.
1. *CONTAINER_REGISTRY*: *LOGINSERVER* для записи контроля доступа.
1. *Узел*. узел для пространства разработки, который принимает форму *<MASTER_SPACE>. <APP_NAME>. <HOST_SUFFIX*>, в этом примере — *dev.bikesharingweb.fedcab0987.EUS.azds.IO*.
1. *IMAGE_PULL_SECRET*— имя секрета, который вы хотите использовать, например " *демонстрационный секрет*".
1. *MASTER_SPACE*: имя родительского пространства разработки, которое в данном примере — *dev*.
1. *REGISTRY_USERNAME*: *ClientID* из выходных данных JSON для создания субъекта-службы.
1. *REGISTRY_PASSWORD*: *clientSecret* из выходных данных JSON при создании субъекта-службы.

> [!NOTE]
> Все эти секреты используются действием GitHub и настраиваются в [. GitHub/Workflows/Bikes. yml][github-action-yaml].

Кроме того, если необходимо обновить основное пространство после слияния запроса на вытягивание, добавьте *GATEWAY_HOST* секрет, который принимает форму *<MASTER_SPACE>. GATEWAY. <HOST_SUFFIX>*, в этом примере — *dev.Gateway.fedcab0987.EUS.azds.IO*. После объединения изменений в главную ветвь в вилке выполняется другое действие для перестроения и запуска всего приложения в основном пространстве разработки. В этом примере основное пространство — *dev*. Это действие настраивается в [. GitHub/Workflows/bikesharing. yml][github-action-bikesharing-yaml].

Кроме того, если вы хотите, чтобы изменения в запросе на вытягивание выполнялись в внучатый пространстве, обновите *MASTER_SPACE* и секреты *узла* . Например, если приложение выполняется в приложении *dev* с дочерним пространством *dev/azureuser1*, то для выполнения запроса на вытягивание в дочернем пространстве *dev/azureuser1*:

* Обновите *MASTER_SPACE* дочерним пространством, которое должно быть родительским пространством в этом примере *azureuser1*.
* Обновите *узел* на *<GRANDPARENT_SPACE>. <APP_NAME>. <HOST_SUFFIX*>, в этом примере *dev.bikesharingweb.fedcab0987.EUS.azds.IO*.

## <a name="create-a-new-branch-for-code-changes"></a>Создание новой ветви для изменения кода

Перейдите к `BikeSharingApp/` и создайте новую ветвь под названием *Bike-Images*.

```cmd
cd dev-spaces/samples/BikeSharingApp/
git checkout -b bike-images
```

Измените [велосипеды/server.js][bikes-server-js] , чтобы удалить строки 232 и 233:

```javascript
    // Hard code image url *FIX ME*
    theBike.imageUrl = "/static/logo.svg";
```

Теперь раздел должен выглядеть следующим образом:

```javascript
    var theBike = result;
    theBike.id = theBike._id;
    delete theBike._id;
```

Сохраните файл, а затем используйте `git add` и `git commit` для размещения изменений.

```cmd
git add Bikes/server.js 
git commit -m "Removing hard coded imageUrl from /bikes/:id route"
```

## <a name="push-your-changes"></a>Отправка изменений

Используйте `git push` для отправки новой ветви в разветвленный репозиторий:

```cmd
git push origin bike-images
```

После завершения отправки перейдите к разветвленному репозиторию на сайте GitHub, чтобы создать запрос на включение внесенных изменений с *главной* ветвью в разветвленном репозитории в качестве базовой ветви по сравнению с ветвью " *образы велосипедов* ".

После открытия запроса на вытягивание перейдите на вкладку *действия* . Убедитесь, что новое действие началось и создает службу *велосипедов* .

## <a name="view-the-child-space-with-your-changes"></a>Просмотр дочернего пространства с вашими изменениями

После завершения действия вы увидите комментарий с URL-адресом нового дочернего пространства на основе изменений в запросе на вытягивание.

> [!div class="mx-imgBorder"]
> ![URL-адрес действия GitHub](../media/github-actions/github-action-url.png)

Перейдите к службе *бикешарингвеб* , открыв URL-адрес из комментария. Выберите пользователя *Aurelia Briggs (customer)* , а затем выберите велосипед, который нужно сдать в прокат. Убедитесь, что вы больше не видите изображение заполнителя для велосипеда.

При объединении изменений в *главную* ветвь в вилке выполняется другое действие для перестроения и запуска всего приложения в родительском пространстве разработки. В этом примере родительское пространство — *dev*. Это действие настраивается в [. GitHub/Workflows/bikesharing. yml][github-action-bikesharing-yaml].

## <a name="clean-up-your-azure-resources"></a>Очистка ресурсов Azure

```azurecli
az group delete --name MyResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о принципах работы Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Принцип работы Azure Dev Spaces](../how-dev-spaces-works.md)

[azure-cli-installed]: /cli/azure/install-azure-cli
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[bikes-server-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/Bikes/server.js#L232-L233
[bike-sharing-gh]: https://github.com/Azure/dev-spaces/
[bike-sharing-values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml
[github-actions-beta-signup]: https://github.com/features/actions
[github-action-yaml]: https://github.com/Azure/dev-spaces/blob/master/.github/workflows/bikes.yml
[github-action-bikesharing-yaml]: https://github.com/Azure/dev-spaces/blob/master/.github/workflows/bikesharing.yml
[helm-installed]: https://helm.sh/docs/intro/install/
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[sp-acr]: ../../container-registry/container-registry-auth-service-principal.md
[sp-aks]: ../../aks/kubernetes-service-principal.md
