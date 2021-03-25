---
title: Краткое руководство. Развертывание приложения Hello World в Сетке Azure Service Fabric
description: В этом кратком руководстве показано, как развернуть приложение Сетки Service Fabric в Сетке Azure Service Fabric.
author: georgewallace
ms.author: gwallace
ms.date: 11/27/2018
ms.topic: quickstart
ms.openlocfilehash: c89cc1972a554cac85ce2a258873f6c810e45167
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102217726"
---
# <a name="quickstart-deploy-hello-world-to-service-fabric-mesh"></a>Краткое руководство. Развертывание приложения Hello World в Сетке Service Fabric

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

[Сетка Service Fabric](service-fabric-mesh-overview.md) упрощает создание и управление приложениями для микрослужб в Azure без необходимости предоставлять виртуальные машины. В этом кратком руководстве вы создадите приложение Hello World в Azure и опубликуете его в Интернете. Эта операция выполняется при помощи одной команды. Всего за несколько минут вы увидите это представление в своем браузере.

![Приложение Hello World в браузере][sfm-app-browser]

[!INCLUDE [preview note](./includes/include-preview-note.md)]

Если у вас еще нет учетной записи Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="set-up-service-fabric-mesh-cli"></a>Настройка CLI для Сетки Service Fabric 
Для выполнения инструкций этого краткого руководства можно использовать Azure Cloud Shell или локальный экземпляр Azure CLI. Установите модуль расширения интерфейса командной строки службы "Сетка Azure Service Fabric", выполнив следующие [инструкции](service-fabric-mesh-howto-setup-cli.md).

## <a name="sign-in-to-azure"></a>Вход в Azure
Войдите в Azure и настройте подписку.

```azurecli-interactive
az login
az account set --subscription "<subscriptionID>"
```

## <a name="create-resource-group"></a>Создать группу ресурсов
Создайте группу ресурсов, в которую будет развертываться приложение. Можно использовать существующую группу ресурсов и пропустить этот шаг. 

```azurecli-interactive
az group create --name myResourceGroup --location eastus 
```

## <a name="deploy-the-application"></a>Развертывание приложения

>[!NOTE]
> Начиная с 2 ноября 2020 года [ограничения скорости скачивания](https://docs.docker.com/docker-hub/download-rate-limit/) применяются к анонимным и прошедшим проверку подлинности запросам к Docker Hub из учетных записей бесплатных планов Docker по IP-адресу. 
> 
> Эти шаблоны используют общедоступные образы из Docker Hub. Обратите внимание, что скорость может быть ограничена. Дополнительные сведения см. в разделе [Проверка подлинности с помощью Docker Hub](../container-registry/buffer-gate-public-content.md#authenticate-with-docker-hub).

Создайте приложение в группе ресурсов с помощью команды `az mesh deployment create`.  Выполните следующую команду:

```azurecli-interactive
az mesh deployment create --resource-group myResourceGroup --template-uri https://raw.githubusercontent.com/Azure-Samples/service-fabric-mesh/master/templates/helloworld/helloworld.linux.json --parameters "{'location': {'value': 'eastus'}}" 
```

Предыдущая команда развертывает приложение Linux, используя [шаблон linux.json](https://raw.githubusercontent.com/Azure-Samples/service-fabric-mesh/master/templates/helloworld/helloworld.linux.json). Если необходимо развернуть приложение Windows, используйте [шаблон windows.json](https://raw.githubusercontent.com/Azure-Samples/service-fabric-mesh/master/templates/helloworld/helloworld.windows.json). Образы контейнера для ОС Windows больше, чем образы контейнера для Linux, поэтому может потребоваться больше времени для развертывания.

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
      "value": "[reference('helloWorldGateway').ipAddress]",
      "type": "string"
    }
  }
```

## <a name="open-the-application"></a>Запуск приложения
После успешного развертывания приложения скопируйте общедоступный IP-адрес службы конечной точки из выхода CLI. Откройте IP-адрес в веб-браузере. Отображается веб-страница с логотипом Сетки Azure Service Fabric.

## <a name="check-the-application-details"></a>Проверка сведений о приложении
Состояние приложения можно проверить с помощью команды `az mesh app show`. Эта команда предоставляет полезную информацию, которую можно отслеживать.

Имя приложения в этом кратком руководстве — `helloWorldApp`, для сбора сведений о приложении, выполните следующую команду.

```azurecli-interactive
az mesh app show --resource-group myResourceGroup --name helloWorldApp
```

## <a name="see-the-application-logs"></a>Обзор журналов приложений

Изучите журналы для развернутого приложения используя команду `az mesh code-package-log get`.

```azurecli-interactive
az mesh code-package-log get --resource-group myResourceGroup --application-name helloWorldApp --service-name helloWorldService --replica-name 0 --code-package-name helloWorldCode
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда все готово для удаления приложения, запустите команду [​​az group delete][az-group-delete], чтобы удалить группу ресурсов, а также ресурсы приложения и сети, которые она содержит.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о создании и развертывании приложений службы "Сетка Service Fabric" доступны в руководстве.
> [!div class="nextstepaction"]
> [Создание и развертывание веб-приложения на базе нескольких служб](service-fabric-mesh-tutorial-create-dotnetcore.md)

<!-- Images -->
[sfm-app-browser]: ./media/service-fabric-mesh-quickstart-deploy-container/HelloWorld.png

<!-- Links / Internal -->
[az-group-delete]: /cli/azure/group
[azure-cli-install]: /cli/azure/install-azure-cli