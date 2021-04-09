---
title: Руководство. Развертывание приложения в службе "Сетка Azure Service Fabric"
description: В этом руководстве вы узнаете, как развертывать приложение в Сетке Service Fabric с помощью шаблона.
author: georgewallace
ms.topic: tutorial
ms.date: 01/11/2019
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: 589e881eb48daf7da9cd2a934b14acfcc76dc5f9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99625421"
---
# <a name="tutorial-deploy-an-application-to-service-fabric-mesh-using-a-template"></a>Руководство. Развертывание приложения в Сетке Service Fabric с помощью шаблона

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство представляет первую часть цикла. Вы узнаете, как развернуть приложение сетки Azure Service Fabric с помощью шаблона.  Приложение состоит из службы веб-интерфейса ASP.NET и службы серверной части веб-API ASP.NET Core, которые находятся в Docker Hub.  Вы извлечете образы двух контейнеров из Docker Hub и затем отправите их в свой частный реестр. Затем вы создадите шаблон Azure RM для приложения и развернете приложение из реестра контейнеров в Сетку Service Fabric. Когда вы закончите, у вас будет простое приложение списка дел в Сетке Service Fabric.

В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание экземпляра частного реестра контейнеров Azure
> * Отправка образа контейнера в реестр
> * Создание файлов шаблона и параметров RM
> * Развертывание приложения в Сетке Service Fabric

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * Развертывание приложения в Сетке Service Fabric с помощью шаблона
> * [Масштабирование служб в приложении, работающем в Сетке Service Fabric](service-fabric-mesh-tutorial-template-scale-services.md)
> * [Обновление приложения, работающего в Сетке Service Fabric](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * [Удаление приложения](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет Azure подписки до начала работы, можно [создать бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* [Установка Docker](service-fabric-mesh-howto-setup-developer-environment-sdk.md#install-docker)

* [Установите Azure CLI и интерфейса командной строки Сетки Azure Service Fabric локально](service-fabric-mesh-howto-setup-cli.md#install-the-azure-service-fabric-mesh-cli).

## <a name="create-a-container-registry"></a>Создание реестра контейнеров

Образы контейнеров, связанные со службами в приложении Сетки Service Fabric, должны храниться в реестре контейнеров.  В этом руководстве используется экземпляр реестра контейнеров Azure (ACR). 

Чтобы создать экземпляр ACR, выполните следующие действия.  Если вас уже есть экземпляр ACR, переходите к следующему шагу.

### <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в Azure и настройте активную подписку.

```azurecli
az login
az account set --subscription "<subscriptionName>"
```

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. С помощью следующей команды создайте группу ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

### <a name="create-the-container-registry"></a>Создание реестра контейнеров

Создайте экземпляр ACR с помощью команды `az acr create`. Имя реестра должно быть уникальным в пределах Azure и содержать от 5 до 50 буквенно-цифровых символов. В следующем примере используется имя *myContainerRegistry*. Если отобразится сообщение о том, что это имя уже используется, выберите другое имя.

```azurecli
az acr create --resource-group myResourceGroup --name myContainerRegistry --sku Basic
```

После создания реестра выходные данные выглядят так:

```json
{
  "adminUserEnabled": false,
  "creationDate": "2018-09-13T19:43:57.388203+00:00",
  "id": "/subscriptions/<subscription>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry",
  "location": "eastus",
  "loginServer": "mycontainerregistry.azurecr.io",
  "name": "myContainerRegistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

## <a name="push-the-images-to-azure-container-registry"></a>Принудительная отправка образов в Реестр контейнеров Azure

В этом руководстве используется пример приложения списка дел.  Образы контейнера для [WebFrontEnd](https://hub.docker.com/r/seabreeze/azure-mesh-todo-webfrontend/) и службы [ToDoService](https://hub.docker.com/r/seabreeze/azure-mesh-todo-service/) можно найти в центре Docker. Дополнительные сведения о том, как создать приложение в Visual Studio, см. в статье [Руководство по созданию, отладке, развертыванию и обновлению приложения на базе нескольких служб в Сетке Service Fabric](service-fabric-mesh-tutorial-create-dotnetcore.md). Сетка Service Fabric может запускать контейнеры Docker Windows или Linux.  Если вы работаете с контейнерами Linux, выберите **Переключиться на контейнеры Linux** в Docker.  Если вы работаете с контейнерами Windows, выберите **Переключиться на контейнеры Windows** в Docker.

Чтобы отправить образ в экземпляр ACR, сначала нужно получить образ контейнера. Если у вас еще нет локальных образов контейнера, используйте команду [docker pull](https://docs.docker.com/engine/reference/commandline/pull/), чтобы извлечь образы [WebFrontEnd](https://hub.docker.com/r/seabreeze/azure-mesh-todo-webfrontend/) и [ToDoService](https://hub.docker.com/r/seabreeze/azure-mesh-todo-service/) из Docker Hub.

>[!NOTE]
> Начиная с 2 ноября 2020 года [ограничения скорости скачивания](https://docs.docker.com/docker-hub/download-rate-limit/) применяются к анонимным и прошедшим проверку подлинности запросам к Docker Hub из учетных записей бесплатных планов Docker по IP-адресу. 
> 
> Эти команды используют общедоступные образы из Docker Hub. Обратите внимание, что скорость может быть ограничена. Дополнительные сведения см. в разделе [Проверка подлинности с помощью Docker Hub](../container-registry/buffer-gate-public-content.md#authenticate-with-docker-hub).

Извлеките образы Windows:

```bash
docker pull seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker pull seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

Прежде чем отправить образ в реестр, нужно добавить в него тег с полным именем сервера входа для ACR.

Выполните следующую команду для получения полного имени сервера входа для экземпляра ACR.

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table

AcrLoginServer
---------------------------------
mycontainerregistry.azurecr.io
```

Присвойте образу Docker тег с помощью команды [docker tag](https://docs.docker.com/engine/reference/commandline/tag/). В следующем примере теги присваиваются образам seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709 и seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709. Если вы используете другие образы, подставьте их имена в следующую команду.

```bash
docker tag seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709 mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker tag seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709 mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

Войдите в Реестр контейнеров Azure.

```azurecli
az acr login -n myContainerRegistry
```

Воспользуйтесь командой [docker push](https://docs.docker.com/engine/reference/commandline/push/) для принудительной отправки образа в экземпляр ACR:

```bash
docker push mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker push mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

### <a name="list-container-images"></a>Список образов контейнеров

Чтобы просмотреть репозитории в вашем экземпляре ACR, используйте следующую команду:

```azurecli
az acr repository list --name myContainerRegistry --output table

Result
-------------------------------
seabreeze/azure-mesh-todo-webfrontend
seabreeze/azure-mesh-todo-service
```

В следующем примере перечисляются теги в репозитории **azure-mesh-todo-service**.

```azurecli
az acr repository show-tags --name myContainerRegistry --repository seabreeze/azure-mesh-todo-service --output table

Result
--------
1.0-nanoserver-1709
```

Приведенные выше выходные данные подтверждают наличие `azure-mesh-todo-service:1.0-nanoserver-1709` в частном реестре контейнеров.  Также проверьте наличие `azure-mesh-todo-webfrontend:1.0-nanoserver-1709`.

## <a name="retrieve-credentials-for-the-registry"></a>Получение учетных данных для реестра

> [!IMPORTANT]
> Добавлять администратора для экземпляра ACR не рекомендуется для рабочих сценариев. Здесь это делается для удобства. В рабочих сценариях используйте [субъект-службу](../container-registry/container-registry-auth-service-principal.md) для проверки подлинности пользователя и системы.

Чтобы развернуть экземпляр контейнера из реестра, который был создан с помощью шаблона, во время развертывания необходимо указать учетные данные реестра. Сначала добавьте администратора в реестр, выполнив следующую команду:

```azurecli
az acr update --name myContainerRegistry --admin-enabled true
```

Затем получите имя входа сервера, имя пользователя и пароль для реестра с помощью следующих команд:

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
az acr credential show --name myContainerRegistry --query username
az acr credential show --name myContainerRegistry --query "passwords[0].value"
```

Используйте значения имени входа сервера, имени пользователя и пароля ACR, возвращенные при создании файлов шаблона и параметров RM в следующем разделе.

## <a name="download-and-explore-the-template-and-parameters-files"></a>Скачивание и изучение файлов шаблона и параметров

Приложение Сетки Service Fabric — это ресурс Azure, который можно развернуть и контролировать с помощью шаблонов Azure Resource Manager (RM). Если вы не знакомы с основными понятиями, связанными с развертыванием решений Azure и управлением ими, см. [обзор Azure Resource Manager](../azure-resource-manager/management/overview.md) и [Сведения о структуре и синтаксисе шаблонов RM](../azure-resource-manager/templates/template-syntax.md).

В этом руководстве используется пример приложения списка дел.  Вместо создания новых файлов шаблона и параметров скачайте файлы [шаблона развертывания mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) и [параметров mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).

### <a name="parameters"></a>Параметры
Если в шаблоне есть значения, которые изменятся после развертывания приложения или которые вы хотели бы изменять для разных развертываний (если планируете повторно использовать этот шаблон), рекомендуется параметризировать эти значения. Лучше всего создать раздел "parameters" в верхней части шаблона развертывания и указать в нем имена и свойства параметров, которые встречаются ниже в шаблоне развертывания. Каждое определение параметра включает *type*, *defaultValue* и дополнительный раздел *metadata* с *description*.

Раздел параметров определяется в верхней части шаблона развертывания, сразу над разделом *recources*:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      ...
    },
    "resources": [
```

В [шаблоне развертывания mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) объявляются следующие параметры: location, registryPassword, registryUserName, registryServer, frontEndImage, serviceImage, frontEndCpu, serviceCpu, frontEndMemory, serviceMemory, frontEndReplicaCount, serviceReplicaCount.  Описания для каждого параметра можно найти в файле шаблона развертывания.

Эти параметры используются в файле [параметров mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).  Использование отдельного файла параметров позволяет менять значения параметров в разных развертываниях без обновления самого шаблона развертывания.

### <a name="overview-of-the-application-and-services"></a>Обзор служб и приложения

Службы задаются в шаблоне как свойства ресурса приложения.  Приложения развертываются в частной сети, которая объявляется как ресурс в шаблоне.  Службы могут использовать тома для сохранения данных, которые объявляются как ресурсы в шаблоне.  Для каждой службы указывается тип операционной системы, пакеты кода, количество реплик и сеть в качестве свойств.  Для каждого пакета кода укажите образ контейнера, конечные точки, ресурсы памяти и ЦП и учетные данные репозитория образа. В общих чертах шаблон приложения Сетки Service Fabric с несколькими службами выглядит как:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    ...
  },
  "resources": [
    {
      "apiVersion": "2018-09-01-preview",
      "name": "MyMeshApplication",
      "type": "Microsoft.ServiceFabricMesh/applications",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/networks/MeshAppNetwork",
        "Microsoft.ServiceFabricMesh/volumes/ServiceAVolume"
      ],
      "properties": {
        "services": [
          {
            "name": "ServiceA",
            "properties": {
              "description": "ServiceA description.",
              "osType": "Linux",
              "codePackages": [
                {
                  "name": "ServiceA",
                  "image": "[parameters('frontEndImage')]",
                  "volumeRefs": [
                    {
                      "name": "[resourceId('Microsoft.ServiceFabricMesh/volumes', 'ServiceAVolume')]",
                      "destinationPath": "/app/data"
                    }
                  ],
                  "endpoints": [
                    {
                      "name": "ServiceAListener",
                      "port": 80
                    }
                  ],
                  "resources": {
                    "requests": {
                      "cpu": "[parameters('frontEndCpu')]",
                      "memoryInGB": "[parameters('frontEndMemory')]"
                    }
                  },
                  "imageRegistryCredential": {
                    "server": "[parameters('registryServer')]",
                    "username": "[parameters('registryUserName')]",
                    "password": "[parameters('registryPassword')]"
                  }
                }
              ],
              "replicaCount": "[parameters('frontEndReplicaCount')]",
              "networkRefs": [
                {
                  "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'MeshAppNetwork')]"
                }
              ]
            }
          },
          {
            "name": "ServiceB",
            ...
          }
        ],
        "description": "Application description."
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "MeshAppNetwork",
      "type": "Microsoft.ServiceFabricMesh/networks",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "MeshAppNetwork description.",
        "addressPrefix": "10.0.0.4/22",
        "ingressConfig": {
          "layer4": [
            {
              "name": "ServiceAIngress",
              "publicPort": "20001",
              "applicationName": "MyMeshApplication",
              "serviceName": "ServiceA",
              "endpointName": "ServiceAListener"
            }
          ]
        }
      }
    },
    {
      "apiVersion": "2018-09-01-preview",
      "name": "ServiceAVolume",
      "type": "Microsoft.ServiceFabricMesh/volumes",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "Azure Files storage volume for counter App.",
        "provider": "SFAzureFile",
        "azureFileParameters": {
          "shareName": "[parameters('fileShareName')]",
          "accountName": "[parameters('storageAccountName')]",
          "accountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
  ]
}
```

Детали приложения списка дел вы найдете в файле [шаблона развертывания mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json).

## <a name="deploy-the-application-to-service-fabric-mesh"></a>Развертывание приложения в Сетке Service Fabric
Создайте приложение и связанные ресурсы, выполнив приведенную ниже команду, и укажите учетные данные из предыдущего шага [Извлечение учетных данных для реестра](#retrieve-credentials-for-the-registry).

В файле параметров измените следующие значения:

|Параметр|Значение|
|---|---|
|location|Регион для развертывания приложения.  Например, "eastus".|
|registryPassword|Пароль, полученный ранее в шаге [Извлечение учетных данных для реестра](#retrieve-credentials-for-the-registry). Этот параметр в шаблоне является защищенной строкой и не будет отображаться в состоянии развертывания или командах `az mesh service show`.|
|registryUserName|Имя пользователя, полученное в шаге [Извлечение учетных данных для реестра](#retrieve-credentials-for-the-registry).|
|registryServer|Имя сервера регистра, полученное в шаге [Извлечение учетных данных для реестра](#retrieve-credentials-for-the-registry).|
|frontEndImage|Образ контейнера для службы внешнего интерфейса.  Например, `<myregistry>.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709`.|
|serviceImage|Образ контейнера для службы серверной части.  Например, `<myregistry>.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709`.|

Для развертывания приложения выполните следующую команду:

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

Эта команда создает такой фрагмент кода JSON. В разделе ```outputs``` выходных данных JSON скопируйте свойство ```publicIPAddress```.

```json
"outputs": {
    "publicIPAddress": {
    "type": "String",
    "value": "40.83.78.216"
    }
}
```

Эти сведения поступают из раздела ```outputs``` шаблона ARM. Как показано ниже, этот раздел ссылается на ресурс шлюза, чтобы получить общедоступный IP-адрес. 

```json
  "outputs": {
    "publicIPAddress": {
      "value": "[reference('todolistappGateway').ipAddress]",
      "type": "string"
    }
  }
```

## <a name="open-the-application"></a>Запуск приложения

После успешного развертывания приложения получите общедоступный IP-адрес конечной точки службы. Команда развертывания вернет общедоступный IP-адрес конечной точки службы. При необходимости вы можете запросить сетевой ресурс, чтобы найти общедоступный IP-адрес конечной точки службы. Имя сетевого ресурса для этого приложения — `todolistappNetwork`. Получить сведения о нем можно с помощью следующей команды. 

```azurecli
az mesh gateway show --resource-group myResourceGroup --name todolistappGateway
```

Откройте IP-адрес в веб-браузере.

## <a name="check-application-status"></a>Проверка состояния приложения

Состояние приложения можно проверить с помощью команды app show. Имя развернутого приложения — todolistapp, поэтому получите сведения с помощью этого имени.

```azurecli
az mesh app show --resource-group myResourceGroup --name todolistapp
```

Изучите журналы для развернутого приложения используя команду `az mesh code-package-log get`.
```azurecli
az mesh code-package-log get --resource-group myResourceGroup --application-name todolistapp --service-name WebFrontEnd --replica-name 0 --code-package-name WebFrontEnd
```

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Создание частного реестра контейнеров.
> * Отправка образа контейнера в реестр
> * Создание файлов шаблона и параметров
> * Развертывание приложения в Сетке Service Fabric

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Масштабирование приложения, работающего в Сетке Service Fabric](service-fabric-mesh-tutorial-template-scale-services.md)