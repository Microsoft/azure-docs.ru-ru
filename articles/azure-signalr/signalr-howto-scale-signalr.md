---
title: Масштабирование экземпляра службы Azure SignalR
description: Узнайте, как масштабировать экземпляр службы Azure SignalR, чтобы добавить или уменьшить емкость с помощью портал Azure или Azure CLI.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 9/9/2020
ms.author: zhshang
ms.custom: devx-track-azurecli
ms.openlocfilehash: bb01f2f96e5db4c94e759b114818360e6084255f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89595773"
---
# <a name="how-to-scale-an-azure-signalr-service-instance"></a>Как масштабировать экземпляр службы Azure SignalR?
В этой статье показано, как масштабировать экземпляр службы Azure SignalR. Существует два сценария масштабирования, увеличения и уменьшения масштаба.

* Увеличение [масштаба](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling): получение дополнительных единиц, подключений, сообщений и т. д. Вы можете увеличить масштаб, изменив ценовую категорию с Free на Standard.
* [Масштабное развертывание](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling): увеличение числа блоков SignalR. Можно масштабировать до 100 единиц. Существует ограниченный вариант единиц измерения для масштабирования: 1, 2, 5, 10, 20, 50 и 100 для одного экземпляра службы SignalR.

Для применения параметров масштабирования требуется несколько минут. В редких случаях применение может занять около 30 минут. Они не нуждаются в изменении кода или повторном развертывании серверного приложения.

Сведения о ценах и емкости индивидуальных сигнальных служб см. в разделе [сведения о ценах на службу Azure SignalR](https://azure.microsoft.com/pricing/details/signalr-service/).  

> [!NOTE]
> При изменении службы SignalR с уровня " **бесплатный** " на " **стандартный** " или наоборот, IP-адрес общедоступной службы будет изменен, а для распространения изменений на DNS-серверы через весь Интернет обычно потребуется 30-60 минут. Служба может быть недоступна до обновления DNS. Как правило, не рекомендуется слишком часто изменять ценовую категорию.


## <a name="scale-on-azure-portal"></a>Масштабирование на портал Azure

1. В браузере откройте [портал Azure](https://portal.azure.com).

2. На странице службы SignalR в меню слева выберите **масштаб**.
   
3. Выберите ценовую категорию и нажмите кнопку **выбрать**. Задайте число единиц для уровня " **стандартный** ".
   
    ![Масштабирование на портале](./media/signalr-howto-scale/signalr-howto-scale.png)

4. Выберите команду **Сохранить**.

## <a name="scale-using-azure-cli"></a>Масштабирование с помощью интерфейса командной строки Azure

Этот скрипт создает новый ресурс службы SignalR уровня " **бесплатный** " и новую группу ресурсов, а затем масштабирует его до уровня " **стандартный** ". 

```azurecli-interactive
#!/bin/bash

# Generate a unique suffix for the service name
let randomNum=$RANDOM*$RANDOM

# Generate a unique service and group name with the suffix
SignalRName=SignalRTestSvc$randomNum
#resource name must be lowercase
mySignalRSvcName=${SignalRName,,}
myResourceGroupName=$SignalRName"Group"

# Create resource group 
az group create --name $myResourceGroupName --location eastus

# Create the Azure SignalR Service resource
az signalr create \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Free_F1 \
  --service-mode Default

# Scale up to Standard Tier, and scale out to 50 units
az signalr update \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Standard_S1 \
  --unit-count 50
```

Запишите фактическое имя, созданное для новой группы ресурсов. Это имя группы ресурсов вам понадобится для удаления групп ресурсов.

[!INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="compare-pricing-tiers"></a>Сравнение ценовых категорий

Подробные сведения, такие как входящие сообщения и подключения для каждой ценовой категории, см. в разделе [сведения о ценах на службу SignalR](https://azure.microsoft.com/pricing/details/signalr-service/).

Таблица ограничений, квот и ограничений службы на каждом уровне см. в разделе [ограничения службы SignalR](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-signalr-service-limits).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как масштабировать один экземпляр службы SignalR.

Для сценариев масштабирования, сегментирования и перекрестных регионов также поддерживаются несколько конечных точек.

> [!div class="nextstepaction"]
> [масштабируемая служба SignalR с несколькими экземплярами](./signalr-howto-scale-multi-instances.md)
