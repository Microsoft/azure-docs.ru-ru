---
title: Использование планового обслуживания для кластера Azure Kubernetes Service (AKS) (Предварительная версия)
titleSuffix: Azure Kubernetes Service
description: Узнайте, как использовать плановое обслуживание в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 03/03/2021
ms.author: qpetraroia
author: qpetraroia
ms.openlocfilehash: f5c85f371dbe0fe3488c1ca6ff806f114153d3a7
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102639190"
---
# <a name="use-planned-maintenance-to-schedule-maintenance-windows-for-your-azure-kubernetes-service-aks-cluster-preview"></a>Использование планового обслуживания для планирования периодов обслуживания для кластера Azure Kubernetes Service (AKS) (Предварительная версия)

В кластере AKS регулярное обслуживание выполняется автоматически. По умолчанию эта работа может быть выполнена в любое время. Плановое обслуживание позволяет планировать еженедельные периоды обслуживания, которые обновляют плоскость управления и сокращают влияние рабочей нагрузки. По расписанию все операции обслуживания будут выполняться во время выбранного окна. Вы можете запланировать один или несколько еженедельных окон в кластере, указав день или диапазон времени в конкретный день. Периоды обслуживания настраиваются с помощью Azure CLI.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="limitations"></a>Ограничения

При использовании планового обслуживания действуют следующие ограничения.

- AKS оставляет за собой право разбивать эти окна на несрочные и экстренные исправления.
- Выполнение операций обслуживания считается *наиболее оптимальным* и не гарантируется в пределах указанного окна.
- Обновления не могут быть заблокированы более семи дней.

### <a name="install-aks-preview-cli-extension"></a>Установка расширения интерфейса командной строки предварительной версии AKS

Также требуется расширение *AKS-preview* Azure CLI версии 0.5.4 или более поздней. Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] . Или установите все доступные обновления с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="allow-maintenance-on-every-monday-at-100am-to-200am"></a>Разрешить обслуживание каждый понедельник в 1:8:00 до 2:8:00

Для добавления периода обслуживания можно использовать `az aks maintenanceconfiguration add` команду.

> [!IMPORTANT]
> Периоды планового обслуживания указываются в формате UTC.

```azurecli-interactive
az aks maintenanceconfiguration add -g MyResourceGroup --cluster-name myAKSCluster --name default --weekday Monday  --start-hour 1
```

В следующем примере выходных данных показано окно обслуживания от 1:8:00 до 2:8:00 каждый понедельник.

```json
{- Finished ..
  "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/default",
  "name": "default",
  "notAllowedTime": null,
  "resourceGroup": "MyResourceGroup",
  "systemData": null,
  "timeInWeek": [
    {
      "day": "Monday",
      "hourSlots": [
        1
      ]
    }
  ],
  "type": null
}
```

Чтобы разрешить обслуживание в любое время в течение дня, не указывайте параметр *Start-Hour* . Например, следующая команда задает для периода обслуживания полный день каждый понедельник:

```azurecli-interactive
az aks maintenanceconfiguration add -g MyResourceGroup --cluster-name myAKSCluster --name default --weekday Monday
```

## <a name="add-a-maintenance-configuration-with-a-json-file"></a>Добавление конфигурации обслуживания с помощью JSON-файла

Можно также использовать JSON-файл, чтобы создать период обслуживания вместо использования параметров. Создайте `test.json` файл со следующим содержимым:

```json
  {
        "timeInWeek": [
          {
            "day": "Tuesday",
            "hour_slots": [
              1,
              2
            ]
          },
          {
            "day": "Wednesday",
            "hour_slots": [
              1,
              6
            ]
          }
        ],
        "notAllowedTime": [
          {
            "start": "2021-05-26T03:00:00Z",
            "end": "2021-05-30T012:00:00Z"
          }
        ]
}
```

В приведенном выше JSON-файле указываются окна обслуживания каждый вторник в 1:8:00-3:8:00 и каждую среду в 1:8:00 – 2:8:00 и в 6:8:00-7:8:00. Кроме того, существует исключение из *2021-05-26T03:00:00Z* до *2021-05-30T12:00:00Z* , где обслуживание не разрешено, даже если оно перекрывается с периодом обслуживания. Следующая команда добавляет окна обслуживания из `test.json` .

```azurecli-interactive
az aks maintenanceconfiguration add -g MyResourceGroup --cluster-name myAKSCluster --name default --config-file ./test.json
```

## <a name="update-an-existing-maintenance-window"></a>Обновление существующего периода обслуживания

Чтобы обновить существующую конфигурацию обслуживания, используйте `az aks maintenanceconfiguration update` команду.

```azurecli-interactive
az aks maintenanceconfiguration update -g MyResourceGroup --cluster-name myAKSCluster --name default --weekday Monday  --start-hour 1
```

## <a name="list-all-maintenance-windows-in-an-existing-cluster"></a>Вывод списка всех периодов обслуживания в существующем кластере

Чтобы просмотреть все текущие окна конфигурации обслуживания в кластере AKS, используйте `az aks maintenanceconfiguration list` команду.

```azurecli-interactive
az aks maintenanceconfiguration list -g MyResourceGroup --cluster-name myAKSCluster
```

В выходных данных ниже видно, что для myAKSCluster настроено два окна обслуживания. Одно окно находится в понедельниках в 1:8:00, а другое окно — в пятницу в 4:8:00.

```json
[
  {
    "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/default",
    "name": "default",
    "notAllowedTime": null,
    "resourceGroup": "MyResourceGroup",
    "systemData": null,
    "timeInWeek": [
      {
        "day": "Monday",
        "hourSlots": [
          1
        ]
      }
    ],
    "type": null
  },
  {
    "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/testConfiguration",
    "name": "testConfiguration",
    "notAllowedTime": null,
    "resourceGroup": "MyResourceGroup",
    "systemData": null,
    "timeInWeek": [
      {
        "day": "Friday",
        "hourSlots": [
          4
        ]
      }
    ],
    "type": null
  }
]
```

## <a name="show-a-specific-maintenance-configuration-window-in-an-aks-cluster"></a>Отображение конкретного окна конфигурации обслуживания в кластере AKS

Чтобы просмотреть конкретное окно настройки обслуживания в кластере AKS, используйте `az aks maintenanceconfiguration show` команду.

```azurecli-interactive
az aks maintenanceconfiguration show -g MyResourceGroup --cluster-name myAKSCluster --name default
```

В следующем примере выходных данных показано окно обслуживания для *по умолчанию*:

```json
{
  "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/default",
  "name": "default",
  "notAllowedTime": null,
  "resourceGroup": "MyResourceGroup",
  "systemData": null,
  "timeInWeek": [
    {
      "day": "Monday",
      "hourSlots": [
        1
      ]
    }
  ],
  "type": null
}
```

## <a name="delete-a-certain-maintenance-configuration-window-in-an-existing-aks-cluster"></a>Удаление определенного окна конфигурации обслуживания в существующем кластере AKS

Чтобы удалить определенное окно конфигурации обслуживания в кластере AKS, используйте `az aks maintenanceconfiguration delete` команду.

```azurecli-interactive
az aks maintenanceconfiguration delete -g MyResourceGroup --cluster-name myAKSCluster --name default
```

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы приступить к обновлению кластера AKS, см. статью [Обновление кластера AKS][aks-upgrade] .


<!-- LINKS - Internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-aks-install-cli]: /cli/azure/aks?view=azure-cli-latest#az-aks-install-cli&preserve-view=true
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register
[aks-upgrade]: upgrade-cluster.md
