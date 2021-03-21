---
title: Получение уведомлений об обслуживании с помощью интерфейса командной строки
description: Просмотрите уведомления об обслуживании для виртуальных машин, работающих в Azure, и запустите самостоятельное обслуживание с помощью Azure CLI.
author: shants123
ms.service: virtual-machines
ms.subservice: maintenance-control
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: cd042ce09533cbefe37cb2e4d311a3857e3dfdec
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102552411"
---
# <a name="handling-planned-maintenance-notifications-using-the-azure-cli"></a>Обработка уведомлений о плановом обслуживании с помощью Azure CLI

**Эта статья относится к виртуальным машинам под управлением Linux и Windows.**

С помощью интерфейса командной строки можно узнать, когда запланировано [обслуживание](maintenance-notifications.md)виртуальных машин. Сведения о плановом обслуживании доступны [на](/cli/azure/vm#az-vm-get-instance-view)
 
Сведения об обслуживании возвращаются, только если имеется запланированное обслуживание. 

```azurecli-interactive
az vm get-instance-view -n myVM -g myResourceGroup --query instanceView.maintenanceRedeployStatus
```

Выходные данные
```
      "maintenanceRedeployStatus": {
      "additionalProperties": {},
      "isCustomerInitiatedMaintenanceAllowed": true,
      "lastOperationMessage": null,
      "lastOperationResultCode": "None",
      "maintenanceWindowEndTime": "2018-06-04T16:30:00+00:00",
      "maintenanceWindowStartTime": "2018-05-21T16:30:00+00:00",
      "preMaintenanceWindowEndTime": "2018-05-19T12:30:00+00:00",
      "preMaintenanceWindowStartTime": "2018-05-14T12:30:00+00:00"
```

## <a name="start-maintenance"></a>Запуск обслуживания

Следующий вызов начнет обслуживание на виртуальной машине, если для параметра задано значение `IsCustomerInitiatedMaintenanceAllowed` true.

```azurecli-interactive
az vm perform-maintenance -g myResourceGroup -n myVM 
```

## <a name="classic-deployments"></a>Классические развертывания

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

Если у вас все еще есть устаревшие виртуальные машины, развернутые с использованием классической модели развертывания, вы можете отправить к ним запрос и запустить их обслуживание с помощью Azure CLI.

Чтобы убедиться, находитесь ли вы в правильном режиме для работы с классической виртуальной машиной, введите:

```
azure config mode asm
```

Чтобы узнать состояние обслуживания виртуальной машины с именем *myVM*, введите:

```
azure vm show myVM 
``` 

Чтобы начать обслуживание классической виртуальной машины с именем *myVM* в службе *myService* и развертывании *myDeployment*, введите команду:

```
azure compute virtual-machine initiate-maintenance --service-name myService --name myDeployment --virtual-machine-name myVM
```

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете управлять плановым обслуживанием с помощью [Azure PowerShell](maintenance-notifications-powershell.md) или [портала](maintenance-notifications-portal.md).
