---
title: Запуск и завершение службы Kubernetes Azure (AKS)
description: Узнайте, как запустить кластер Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 09/24/2020
author: palma21
ms.openlocfilehash: 94edf35cc16d4967449af15797f6ecccba60be4b
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102181097"
---
# <a name="stop-and-start-an-azure-kubernetes-service-aks-cluster-preview"></a>Останавливает и запускает кластер Azure Kubernetes Service (AKS) (Предварительная версия)

Рабочие нагрузки AKS могут не выполняться непрерывно, например, в кластере разработки, который используется только в рабочее время. Это ведет к тому, что кластер Azure Kubernetes Service (AKS) может простаивать, не выполняя никаких дополнительных системных компонентов. Можно уменьшить объем ресурсов кластера путем [масштабирования всех `User` пулов узлов до 0](scale-cluster.md#scale-user-node-pools-to-0), но [ `System` пул](use-system-pools.md) по-прежнему требуется для запуска системных компонентов во время работы кластера. Чтобы оптимизировать затраты в течение этих периодов, можно полностью отключить (завершить) кластер. Это действие полностью останавливает свою плоскость управления и узлы агентов, позволяя экономить все затраты на вычисление, сохраняя при этом все объекты и состояние кластера, сохраненные для запуска. Затем вы можете выбрать, где вы оставили после выходных или чтобы ваш кластер выполнялся только во время выполнения пакетных заданий.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].


### <a name="limitations"></a>Ограничения

При использовании функции запуска и окончания кластера действуют следующие ограничения.

- Эта функция поддерживается только для кластеров с поддержкой масштабируемых наборов виртуальных машин.
- Состояние кластера остановленного кластера AKS сохраняется в течение 12 месяцев. Если кластер остановлен более 12 месяцев, состояние кластера восстановить нельзя. Дополнительные сведения см. в разделе [политики поддержки AKS](support-policies.md).
- Во время действия предварительной версии необходимо отключить Автомасштабирование кластера (ЦС) перед попыткой его отключения.
- Вы можете запустить или удалить остановленный кластер AKS. Чтобы выполнить любые операции, такие как масштабирование или обновление, сначала запустите кластер.

### <a name="install-the-aks-preview-azure-cli"></a>Установка `aks-preview` Azure CLI 

Также требуется расширение *AKS-preview* Azure CLI версии 0.4.64 или более поздней. Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] . Или установите все доступные обновления с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

### <a name="register-the-startstoppreview-preview-feature"></a>Регистрация `StartStopPreview` функции предварительной версии

Чтобы использовать функцию "запуск/завершение кластера", необходимо включить `StartStopPreview` флаг компонента в подписке.

Зарегистрируйте `StartStopPreview` флаг компонента с помощью команды [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "StartStopPreview"
```

Через несколько минут отобразится состояние *Registered* (Зарегистрировано). Проверьте состояние регистрации с помощью команды [AZ Feature List][az-feature-list] .

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/StartStopPreview')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. ContainerService* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="stop-an-aks-cluster"></a>Завершение кластера AKS

Вы можете использовать `az aks stop` команду, чтобы прерывать работающие узлы кластера AKS и плоскости управления. В следующем примере останавливается кластер с именем *myAKSCluster*:

```azurecli-interactive
az aks stop --name myAKSCluster --resource-group myResourceGroup
```

Вы можете проверить, не остановлен ли кластер, выполнив команду [AZ AKS][az-aks-show] -OUTPUT и убедившись, `powerState` что показанные `Stopped` ниже выходные данные.

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Stopped"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

Если `provisioningState` показывает `Stopping` , что кластер еще не полностью остановлен.

> [!IMPORTANT]
> Если вы используете [бюджеты прерываний Pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) , операция остановки может занять больше времени, так как процесс стока займет большее время.


## <a name="start-an-aks-cluster"></a>Запуск кластера AKS

Вы можете использовать `az aks start` команду, чтобы запустить остановленные узлы кластера AKS и плоскость управления. Кластер перезапускается с предыдущим состоянием плоскости управления и количеством узлов агентов.  
В следующем примере запускается кластер с именем *myAKSCluster*:

```azurecli-interactive
az aks start --name myAKSCluster --resource-group myResourceGroup
```

Вы можете проверить, запущен ли кластер, выполнив команду [AZ AKS Показать][az-aks-show] и убедившись, `powerState` что показанные `Running` ниже выходные данные.

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Running"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

Если `provisioningState` показывает `Starting` , что кластер еще не был полностью запущен.


## <a name="next-steps"></a>Дальнейшие действия

- Чтобы узнать, как масштабировать `User` Пулы до 0, см. раздел [масштабирование `User` пулов до 0](scale-cluster.md#scale-user-node-pools-to-0).
- Сведения о том, как сократить расходы с помощью экземпляров смесевых, см. в разделе [Добавление пула узлов смесевых цветов в AKS](spot-node-pool.md).
- Дополнительные сведения о политиках поддержки AKS см. в разделе [политики поддержки AKS](support-policies.md).

<!-- LINKS - external -->

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-show]: /cli/azure/aks#az_aks_show
