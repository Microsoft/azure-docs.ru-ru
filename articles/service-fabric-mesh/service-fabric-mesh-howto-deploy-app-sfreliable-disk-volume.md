---
title: Service Fabric надежного тома диска с Service Fabricной сеткой
description: Узнайте, как сохранить состояние в приложении "Сетка Azure Service Fabric" путем подключения тома надежного диска Service Fabric в контейнере с помощью Azure CLI.
author: ashishnegi
ms.topic: conceptual
ms.date: 12/03/2018
ms.author: asnegi
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: ac65693f2513338695e07cd8a19acb13333e7281
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99625792"
---
# <a name="mount-highly-available-service-fabric-reliable-disk-based-volume-in-a-service-fabric-mesh-application"></a>Подключение тома надежного диска высокой доступности Service Fabric в приложении "Сетка Azure Service Fabric" 

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Распространенный способ сохранения состояния в приложениях-контейнерах — использовать удаленное хранилище, такое как хранилище файлов Azure, или базу данных, такую как Azure Cosmos DB. Это влечет за собой значительную сетевую задержку при доступе к удаленному хранилищу для чтения и записи.

В этой статье показано, как сохранить состояние на надежном диске высокой доступности Service Fabric, подключив том в контейнере приложения "Сетка Azure Service Fabric".
Надежный диск Service Fabric предоставляет тома для локальных операций чтения с репликацией записей в кластере Service Fabric для обеспечения высокой доступности. Это устраняет необходимость в сетевых вызовах для чтения и уменьшает задержку сети для операций записи. Если контейнер перезапустится или переместится на другой узел, новый экземпляр контейнера будет рассматривать прежний том как устарелый. Это одновременно обеспечивает и эффективность, и высокую доступность.

В этом примере приложение Counter содержит службу ASP.NET Core с веб-страницей, на которой в браузере отображается значение счетчика.

`counterService` периодически считывает значение счетчика из файла, увеличивает его и записывает обратно в файл. Этот файл хранится в папке, подключенной к тому на надежном диске Service Fabric.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этой задачи можно использовать Azure Cloud Shell или локальный экземпляр Azure CLI. Чтобы использовать Azure CLI в рамках работы с этой статьей, убедитесь, что `az --version` возвращает по крайней мере версию `azure-cli (2.0.43)`.  Установите (или обновите) модуль расширения интерфейса командной строки службы "Сетка Azure Service Fabric", выполнив эти [инструкции](service-fabric-mesh-howto-setup-cli.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в Azure и настройте подписку.

```azurecli-interactive
az login
az account set --subscription "<subscriptionID>"
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов, в которую будет развертываться приложение. Следующая команда создает группу ресурсов с именем `myResourceGroup` в расположении в восточной части США. Если вы измените имя группы ресурсов в приведенной ниже команде, не забудьте изменить его во всех последующих командах.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="deploy-the-template"></a>Развертывание шаблона

>[!NOTE]
> Начиная с 2 ноября 2020 года [ограничения скорости скачивания](https://docs.docker.com/docker-hub/download-rate-limit/) применяются к анонимным и прошедшим проверку подлинности запросам к Docker Hub из учетных записей бесплатных планов Docker по IP-адресу. 
> 
> Этот шаблон использует общедоступные образы из DOCKER Hub. Обратите внимание, что скорость может быть ограничена. Дополнительные сведения см. в разделе [Проверка подлинности с помощью Docker Hub](../container-registry/buffer-gate-public-content.md#authenticate-with-docker-hub).

Следующая команда развертывает приложение Linux с помощью [шаблона counter.sfreliablevolume.linux.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/counter/counter.sfreliablevolume.linux.json). Чтобы развернуть приложение Windows, используйте [шаблон counter.sfreliablevolume.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/counter/counter.sfreliablevolume.windows.json). Имейте в виду, что развертывание больших образов контейнеров может занять больше времени.

```azurecli-interactive
az mesh deployment create --resource-group myResourceGroup --template-uri https://raw.githubusercontent.com/Azure-Samples/service-fabric-mesh/master/templates/counter/counter.sfreliablevolume.linux.json
```

Можно также просмотреть состояние развертывания с помощью команды:

```azurecli-interactive
az deployment group show --name counter.sfreliablevolume.linux --resource-group myResourceGroup
```

Обратите внимание на имя ресурса шлюза, который имеет тип ресурса `Microsoft.ServiceFabricMesh/gateways`. Он будет использоваться при получении общедоступного IP-адреса приложения.

## <a name="open-the-application"></a>Запуск приложения

Когда приложение будет успешно развернуто, получите IP-адрес ресурса шлюза для него. Используйте имя шлюза, которое вы записали в разделе, приведенном выше.
```azurecli-interactive
az mesh gateway show --resource-group myResourceGroup --name counterGateway
```

В выходных данных должно содержаться свойство `ipAddress`, которое является общедоступным IP-адресом для конечной точки службы. Откройте его в веб-браузере. Отобразится веб-страница со значением счетчика, которое обновляется каждую секунду.

## <a name="verify-that-the-application-is-able-to-use-the-volume"></a>Проверка доступности тома для приложения

Приложение создает файл `counter.txt` в томе, расположенном в папке `counter/counterService`. Этот файл содержит значение счетчика, отображаемое на веб-странице.

## <a name="delete-the-resources"></a>Удаление ресурсов

Регулярно удаляйте ресурсы, которые больше не используются в Azure. Чтобы удалить ресурсы, относящиеся к этому примеру, удалите группу ресурсов, в которой они были развернуты (при этом будут удалены все элементы, связанные с этой группой ресурсов), с помощью следующей команды:

```azurecli-interactive
az group delete --resource-group myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

- Просмотрите пример приложения с надежным производительным диском Service Fabric на [сайте GitHub](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/counter).
- Узнайте больше о модели ресурсов Service Fabric из раздела [Introduction to Service Fabric Resource Model](service-fabric-mesh-service-fabric-resources.md) (Общие сведения о модели ресурсов Service Fabric).
- Узнайте больше о службе "Сетка Service Fabric" из раздела [Что такое Сетка Service Fabric?](service-fabric-mesh-overview.md)