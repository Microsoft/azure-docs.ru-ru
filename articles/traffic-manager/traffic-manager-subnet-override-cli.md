---
title: Переопределение подсети диспетчера трафика Azure с помощью Azure CLI | Документация Майкрософт
description: Эта статья поможет вам понять, как можно использовать переопределение подсети диспетчера трафика для переопределения метода маршрутизации профиля диспетчера трафика, чтобы направить трафик в конечную точку на основе IP-адреса конечного пользователя через предопределенный диапазон IP-адресов для сопоставления конечных точек.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.topic: how-to
ms.service: traffic-manager
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 2e289728c7fde9b98256d079d45067aba1d4d805
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102211334"
---
# <a name="traffic-manager-subnet-override-using-azure-cli"></a>Переопределение подсети диспетчера трафика с помощью Azure CLI

Переопределение подсети диспетчера трафика позволяет изменить метод маршрутизации профиля.  Добавление переопределения направляет трафик, основанный на IP-адресе конечного пользователя, с предопределенным диапазоном IP-адресов для сопоставления конечных точек. 

## <a name="how-subnet-override-works"></a>Как работает переопределение подсети

Когда переопределение подсети добавляется в профиль диспетчера трафика, диспетчер трафика сначала проверяет, есть ли переопределение подсети для IP-адреса конечного пользователя. Если он найден, запрос DNS пользователя будет перенаправлен в соответствующую конечную точку.  Если сопоставление не найдено, диспетчер трафика будет возвращаться к исходному методу маршрутизации профиля. 

Диапазоны IP-адресов могут быть указаны либо в виде диапазонов CIDR (например, 1.2.3.0/24), либо в качестве диапазонов адресов (например, 1.2.3.4-5.6.7.8). Диапазоны IP-адресов, связанные с каждой конечной точкой, должны быть уникальными для этой конечной точки. Любое перекрытие диапазонов IP-адресов между разными конечными точками приведет к тому, что профиль будет отклонен диспетчером трафика.

Существуют два типа профилей маршрутизации, которые поддерживают переопределения подсетей.

* **География** . Если диспетчер трафика находит переопределение подсети для IP-адреса DNS-запроса, он направляет запрос в конечную точку, независимо от работоспособности конечной точки.
* **Производительность** . Если диспетчер трафика находит переопределение подсети для IP-адреса DNS-запроса, он будет направлять трафик в конечную точку только в том случае, если он исправен.  Диспетчер трафика будет передаваться на эвристику маршрутизации производительности, если конечная точка переопределения подсети находится в неработоспособном состоянии.

## <a name="create-a-traffic-manager-subnet-override"></a>Создание переопределения подсети диспетчера трафика

Чтобы создать переопределение подсети диспетчера трафика, можно использовать Azure CLI, чтобы добавить подсети для переопределения в конечную точку диспетчера трафика.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="update-the-traffic-manager-endpoint-with-subnet-override"></a>Обновите конечную точку диспетчера трафика с переопределением подсети.
Используйте Azure CLI для обновления конечной точки с помощью команды [AZ Network трафика — обновление конечной точки диспетчера](/cli/azure/network/traffic-manager/endpoint#az-network-traffic-manager-endpoint-update).

```azurecli-interactive
### Add a range of IPs ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 1.2.3.4-5.6.7.8 \
    --type AzureEndpoints

### Add a subnet ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 9.10.11.0:24 \
    --type AzureEndpoints
```

Диапазоны IP-адресов можно удалить, выполнив [Обновление конечной точки AZ Network Configuration Manager](/cli/azure/network/traffic-manager/endpoint#az-network-traffic-manager-endpoint-update) с параметром **--Remove** .

```azurecli-interactive
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --remove subnets \
    --type AzureEndpoints
```

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения в статье [Методы маршрутизации трафика диспетчером трафика](traffic-manager-routing-methods.md).

Сведения о [методе маршрутизации трафика подсети](./traffic-manager-routing-methods.md#subnet-traffic-routing-method)