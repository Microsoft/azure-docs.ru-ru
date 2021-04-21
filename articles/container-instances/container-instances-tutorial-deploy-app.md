---
title: Учебник. Развертывание приложения-контейнера в экземпляр контейнеров
description: Руководство по службе "Экземпляры контейнеров Azure" — часть 3 из 3. Развертывание приложения-контейнера в службе "Экземпляры контейнеров Azure"
ms.topic: tutorial
ms.date: 03/21/2018
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 2d12b0bc91cf844001b770c8a7e534ad6b1b9b2d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763889"
---
# <a name="tutorial-deploy-a-container-application-to-azure-container-instances"></a>Руководство. Развертывание приложения-контейнера в службе "Экземпляры контейнеров Azure"

Это последняя из трех частей руководства. С помощью предыдущих частей мы [создали образ контейнера](container-instances-tutorial-prepare-app.md), который затем [передали в реестр контейнеров Azure](container-instances-tutorial-prepare-acr.md). Эта часть завершает серию руководств. Мы развернем контейнер в службе "Экземпляры контейнеров Azure".

Изучив это руководство, вы:

> [!div class="checklist"]
> * развернете контейнер в службе "Экземпляры контейнеров Azure" из реестра контейнеров Azure;
> * просмотрите выполняющееся приложение в браузере;
> * отобразите журналы контейнера.

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="deploy-the-container-using-the-azure-cli"></a>Развертывание контейнера с помощью Azure CLI

В этом разделе с помощью Azure CLI вы развернете образ, созданный в [первом руководстве](container-instances-tutorial-prepare-app.md) и переданный в реестр контейнеров Azure во [втором руководстве](container-instances-tutorial-prepare-acr.md). Обязательно выполните инструкции в этих руководствах, прежде чем продолжить.

### <a name="get-registry-credentials"></a>Получение учетных данных реестра

При развертывании образа, который размещен в частном реестре контейнеров Azure, например созданного во [втором учебнике](container-instances-tutorial-prepare-acr.md), нужно предоставить учетные данные для доступа к реестру. 

Для большинства сценариев рекомендуется создать и настроить субъект-службу Azure Active Directory с разрешениями на *извлечение* из вашего реестра. Примеры скриптов для создания субъекта-службы с необходимыми разрешениями см. в статье [Authenticate with Azure Container Registry from Azure Container Instances](../container-registry/container-registry-auth-aci.md) (Проверка подлинности с помощью Реестра контейнеров Azure из службы "Экземпляры контейнеров Azure"). Запишите *идентификатор* и *пароль субъекта-службы*. Эти учетные данные пригодятся для доступа к реестру во время развертывания контейнера.

Также необходимо получить полное имя сервера для входа в реестр контейнеров (замените `<acrName>` именем реестра):

```azurecli
az acr show --name <acrName> --query loginServer
```

### <a name="deploy-container"></a>Развертывание контейнера

Теперь разверните контейнер с помощью команды [az container create][az-container-create]. Замените `<acrLoginServer>` значениями, полученными посредством предыдущих команд. Замените `<service-principal-ID>` и `<service-principal-password>` идентификатором и паролем субъекта-службы, который вы создали для доступа к реестру. Замените `<aciDnsLabel>` необходимым DNS-именем.

```azurecli
az container create --resource-group myResourceGroup --name aci-tutorial-app --image <acrLoginServer>/aci-tutorial-app:v1 --cpu 1 --memory 1 --registry-login-server <acrLoginServer> --registry-username <service-principal-ID> --registry-password <service-principal-password> --dns-name-label <aciDnsLabel> --ports 80
```

В течение нескольких секунд вы должны получить исходный ответ Azure. Значение `--dns-name-label` должно быть уникальным в пределах региона Azure, в котором создается экземпляр контейнера. Если при выполнении команды появится сообщение об ошибке **Метка DNS-имени**, измените значение в предыдущей команде.

### <a name="verify-deployment-progress"></a>Проверка хода выполнения развертывания

Чтобы просмотреть состояние развертывания, используйте команду [az container show][az-container-show].

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query instanceView.state
```

Повторяйте выполнение команды [az container show][az-container-show], пока состояние не изменится с *Ожидание* на *Выполняется*, что должно занять не больше минуты. Если контейнер находится в состоянии *Выполняется*, перейдите к следующему шагу.

## <a name="view-the-application-and-container-logs"></a>Просмотр приложения и журналов контейнера

После успешного развертывания отобразите полное доменное имя контейнера (FQDN) с помощью команды [az container show][az-container-show]:

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query ipAddress.fqdn
```

Пример:
```output
"aci-demo.eastus.azurecontainer.io"
```

Чтобы увидеть работающее приложение, перейдите к полученному DNS-имени в любом браузере:

![Приложение Hello World в браузере][aci-app-browser]

Также можно просмотреть выходные данные журнала контейнера:

```azurecli
az container logs --resource-group myResourceGroup --name aci-tutorial-app
```

Выходные данные примера:

```output
listening on port 80
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET / HTTP/1.1&quot; 200 1663 &quot;-&quot; &quot;Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://aci-demo.eastus.azurecontainer.io/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы, созданные в этом руководстве, больше не требуются, можно выполнить команду [az group delete][az-group-delete], чтобы удалить группу ресурсов и все ресурсы, которые она содержит. Эта команда удаляет созданный вами реестр контейнеров, а также запущенный контейнер и все связанные с ним ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого руководства мы завершили процесс развертывания контейнера в службе "Экземпляры контейнеров Azure". Были выполнены следующие действия:

> [!div class="checklist"]
> * развертывание контейнера из реестра контейнеров Azure с помощью Azure CLI;
> * Просмотр приложения в браузере
> * Просмотр журналов контейнера

Теперь, когда вы изучили основные принципы работы, узнайте больше о службе "Экземпляры контейнеров Azure", например, о том, как работают группы контейнеров.

> [!div class="nextstepaction"]
> [Группы контейнеров в службе "Экземпляры контейнеров Azure"](container-instances-container-groups.md)

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/aci-app-browser.png

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-container-show]: /cli/azure/container#az_container_show
[az-group-delete]: /cli/azure/group#az_group_delete
[azure-cli-install]: /cli/azure/install-azure-cli
[prepare-app]: ./container-instances-tutorial-prepare-app.md
