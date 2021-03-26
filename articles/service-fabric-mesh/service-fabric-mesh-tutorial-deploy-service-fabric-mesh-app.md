---
title: Руководство. Развертывание приложения Сетки Service Fabric
description: Узнайте, как использовать Visual Studio, чтобы опубликовать приложение Сетки Azure Service Fabric, состоящее из веб-сайта ASP.NET Core, который обменивается данными с северной веб-службой.
author: georgewallace
ms.topic: tutorial
ms.date: 09/18/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: 713f8dcb3d3b3d30fecbea4bb6b50cc4e47d451d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102216757"
---
# <a name="tutorial-deploy-a-service-fabric-mesh-application"></a>Руководство по Развертывание приложения Сетки Service Fabric.

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство является третьей частью цикла, в котором показано, как публиковать веб-приложение Сетки Azure Service Fabric непосредственно из Visual Studio.

Из этого руководства вы узнаете, как выполнить следующие задачи:
> [!div class="checklist"]
> * Опубликуйте приложение в Azure с помощью Visual Studio.
> * Проверка состояния развертывания приложения
> * Обзор всех приложений, развернутых сейчас в подписке

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Создание приложения Сетки Service Fabric в Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Отладка приложения Сетки Service Fabric, выполняющегося в локальном кластере разработки](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * Развертывание приложения Сетки Service Fabric
> * [Обновление приложения Сетки Service Fabric](service-fabric-mesh-tutorial-upgrade.md)
> * [Удаление ресурсов Сетки Service Fabric](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет Azure подписки до начала работы, можно [создать бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Убедитесь, что имеется [настроенная среда разработки](service-fabric-mesh-howto-setup-developer-environment-sdk.md), которая включает установку среды выполнения Service Fabric, пакет SDK, Docker и Visual Studio 2017.

## <a name="download-the-to-do-sample-application"></a>Загрузка образца приложения

Если вы не создавали пример приложения [во второй части этой серии руководств](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md), можете скачать его. В окне терминала выполните следующую команду, чтобы клонировать репозиторий с примером приложения на локальный компьютер.

```
git clone https://github.com/azure-samples/service-fabric-mesh
```

Приложение находится в каталоге `src\todolistapp`.

## <a name="publish-to-azure"></a>Публикация в Azure

Чтобы опубликовать проект Сетки Service Fabric в Azure, щелкните правой кнопкой мыши **todolistapp** в Visual Studio и выберите **Опубликовать...**

Дальше появится диалоговое окно **Публикация приложения Service Fabric**.

![Диалоговое окно публикации службы "Сетка Service Fabric" в Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

Выберите учетную запись Azure и подписку. Выберите **расположение**. В этой статье используется **восточная часть США**.

В разделе **Группа ресурсов** выберите **\<Create New Resource Group...>** . Откроется диалоговое окно, в котором создайте новую группу ресурсов. В этой статье используется расположение **восточная часть США** и имена группы **sfmeshTutorial1RG** (если в организации несколько человек используют одну и ту же подписку, выберите уникальное имя группы).  Нажмите клавишу **Создать**, чтобы создать группу ресурсов и вернитесь в диалоговое окно публикации.

![Диалоговое окно новой группы ресурсов службы "Сетка Service Fabric" в Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-new-resource-group-dialog.png)

Вернитесь в диалоговое окно **Опубликовать приложение Service Fabric** и в разделе **Реестр контейнеров Azure** выберите **\<Create New Container Registry...>** . В диалоговом окне **Создать реестр контейнеров** используйте уникальное имя для параметра **Имя реестра контейнеров**. Укажите **Расположение** (в этом руководстве используется **восточная часть США**). Выберите в раскрывающемся списке **группу ресурсов**, созданную на предыдущем шаге, например **sfmeshTutorial1RG**. Для **номера SKU** установите значение **Базовый**, а затем щелкните **Создать**, чтобы создать частный реестр контейнеров Azure и вернуться в диалоговое окно публикации.

![Диалоговое окно нового реестра контейнеров службы "Сетка Service Fabric" в Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-new-container-registry-dialog.png)

Если возникает ошибка, что поставщик ресурсов не зарегистрирован для вашей подписки, можно его зарегистрировать. Сначала определите, доступен ли поставщик ресурсов для вашей подписки.

```Powershell
Get-AzureRmResourceProvider -ListAvailable
```

Если поставщик реестра контейнеров (`Microsoft.ContainerRegistry`) доступен, зарегистрируйте его с помощью PowerShell.

```Powershell
Connect-AzureRmAccount
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ContainerRegistry
```

Чтобы развернуть приложение Service Fabric в Azure, в диалоговом окне публикации нажмите клавишу **Опубликовать**.

При первой публикации в Azure образ Docker помещается в службу "Реестр контейнеров Azure" (ACR), на что требуется определенное время в зависимости от размера образа. Следующие публикации того же проекта будут проходить быстрее. Ход выполнения процесса развертывания можно отследить, выбрав панель **Средства Service Fabric** в окне **Вывод** в Visual Studio. После завершения развертывания выходные данные в области **Средства Service Fabric** будут отображать IP-адрес и порт приложения в виде URL-адреса.

```
Packaging Application...
Building Images...
Web1 -> C:\Code\ServiceFabricMeshApp\ToDoService\bin\Any CPU\Release\netcoreapp2.0\ToDoService.dll
Uploading the images to Azure Container Registry...
Deploying application to remote endpoint...
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

Откройте веб-браузер и перейдите к URL-адресу, чтобы увидеть работу сайта в Azure.

## <a name="set-up-service-fabric-mesh-cli"></a>Настройка CLI для Сетки Service Fabric

Чтобы выполнить остальные шаги, можно использовать Azure Cloud Shell или локальный экземпляр Azure CLI. Установите модуль расширения интерфейса командной строки службы "Сетка Azure Service Fabric", выполнив следующие [инструкции](service-fabric-mesh-howto-setup-cli.md).

## <a name="check-application-deployment-status"></a>Проверка состояния развертывания приложения

На этом этапе приложение развернуто. Можно проверить его состояние с помощью команды `app show`. 

Имя приложения для учебного приложения — `todolistapp`. Соберите сведения о приложении с помощью следующей команды.

```azurecli-interactive
az mesh app show --resource-group $rg --name todolistapp
```

## <a name="get-the-ip-address-of-your-deployment"></a>Получение IP-адреса развертывания

Чтобы получить IP-адрес своего приложения, используйте следующую команду.
  
```azurecli-interactive
az mesh gateway show --resource-group myResourceGroup --name todolistappGateway
```

## <a name="see-all-applications-currently-deployed-to-your-subscription"></a>Обзор всех приложений, развернутых сейчас в подписке

Чтобы получить список приложений, которые развернуты в подписке, можно использовать команду app list.

```azurecli-interactive
az mesh app list --output table
```

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:
> [!div class="checklist"]
> * Публикация приложения в Azure.
> * Проверка состояния развертывания приложения
> * Обзор всех приложений, развернутых сейчас в вашей подписке

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Обновление приложения Сетки Service Fabric](service-fabric-mesh-tutorial-upgrade.md)

[azure-cli-install]: /cli/azure/install-azure-cli
