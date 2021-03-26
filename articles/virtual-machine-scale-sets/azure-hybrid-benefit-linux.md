---
title: Преимущество гибридного использования Azure для масштабируемых наборов виртуальных машин Linux
description: Узнайте, как Преимущество гибридного использования Azure можно применить к масштабируемому набору виртуальных машин, чтобы сэкономить деньги на виртуальных машинах Linux, работающих в Azure.
services: virtual-machine-scale-sets
documentationcenter: ''
author: mathapli
manager: rochakm
ms.service: virtual-machine-scale-sets
ms.subservice: linux
ms.collection: linux
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 03/20/2021
ms.author: mathapli
ms.openlocfilehash: 09e41b4071ad02120f7d303b32bc87bc0ba39e2c
ms.sourcegitcommit: 44edde1ae2ff6c157432eee85829e28740c6950d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105549303"
---
# <a name="azure-hybrid-benefit-for-linux-virtual-machine-scale-set-public-preview"></a>Преимущество гибридного использования Azure для масштабируемого набора виртуальных машин Linux (общедоступная Предварительная версия)

**Преимущество гибридного использования Azure для масштабируемого набора виртуальных машин Linux теперь находится в общедоступной предварительной версии**. Преимущество АХБ может помочь снизить затраты на выполнение [масштабируемых наборов виртуальных машин](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview)RHEL и SLES.

С этим преимуществом вы платите только за стоимость инфраструктуры масштабируемого набора. Это преимущество доступно для всех образов RHEL и SLES Marketplace с оплатой по мере использования (PAYG).


>[!NOTE]
> В этой статье описывается Преимущество гибридного использования Azure для Linux VMSS. Здесь доступен отдельный раздел [ [АХБ for Linux VMS](https://docs.microsoft.com/azure/virtual-machines/linux/azure-hybrid-benefit-linux), который уже доступен для клиентов Azure с ноября 2020.

## <a name="benefit-description"></a>Описание преимущества
Гибридная служба Azure позволяет использовать существующие лицензии на облачный доступ из Red Hat или SUSE и гибкого преобразования экземпляров масштабируемых наборов виртуальных машин в выставление счетов с использованием собственной подписки (BYOS). 

Масштабируемый набор виртуальных машин, развернутый из образов PAYG Marketplace, будет взимать плату за инфраструктуру и программное обеспечение. При использовании АХБ экземпляры PAYG можно преобразовать в модель выставления счетов BYOS без повторного развертывания.

:::image type="content" source="./media/azure-hybrid-benefit-linux/azure-hybrid-benefit-linux-cost.png" alt-text="Преимущество гибридного использования Azure визуализации затрат на виртуальных машинах Linux.":::

## <a name="scope-of-azure-hybrid-benefit-eligibility-for-linux"></a>Область действия Преимущество гибридного использования Azure для Linux
Преимущество гибридного использования Azure доступны для всех образов RHEL и SLES PAYG из Azure Marketplace. Это преимущество еще не доступно для образов RHEL или SLES BYOS или пользовательских образов из Azure Marketplace.

Экземпляры выделенных узлов Azure и гибридные преимущества SQL не подходят для Преимущество гибридного использования Azure, если вы уже используете преимущества виртуальных машин Linux.

## <a name="get-started"></a>Начало работы

### <a name="red-hat-customers"></a>Клиенты Red Hat

Преимущество гибридного использования Azure для RHEL доступна для клиентов Red Hat, отвечающих обоим критериям:

- Наличие активных или неиспользуемых подписок RHEL, доступных для использования в Azure
- Включить одну или несколько подписок для использования в Azure с помощью программы [Cloud Access для Red Hat](https://www.redhat.com/en/technologies/cloud-computing/cloud-access)

> [!IMPORTANT]
> Убедитесь, что в программе [доступа к облаку](https://www.redhat.com/en/technologies/cloud-computing/cloud-access) включена правильная подписка.

Чтобы начать использовать преимущество для Red Hat, сделайте следующее:

1. Включите подходящие подписки RHEL в Azure с помощью [интерфейса клиента Red Hat Cloud Access](https://access.redhat.com/management/cloud).

   Для подписок Azure, которые вы предоставляете в процессе включения облачного доступа Red Hat, будет разрешено использовать функцию Преимущество гибридного использования Azure.
1. Примените Преимущество гибридного использования Azure к любому существующему и новому масштабируемому набору виртуальных машин RHEL PAYG. Для реализации преимущества можно использовать портал Azure или Azure CLI.
1. Следуйте рекомендуемым [дальнейшим действиям](https://access.redhat.com/articles/5419341) по настройке источников обновлений для ВИРТУАЛЬНЫХ машин RHEL и правилам соответствия подписок RHEL.


### <a name="suse-customers"></a>Клиенты SUSE

Чтобы приступить к использованию преимущества для SUSE, сделайте следующее:

1. Зарегистрируйтесь в программе общедоступного облака SUSE.
1. Примените преимущество к вновь созданному или существующему масштабируемому набору виртуальных машин с помощью портал Azure или Azure CLI.
1. Зарегистрируйте виртуальные машины, которые получают преимущество с помощью отдельного источника обновлений.


## <a name="enable-and-disable-the-benefit-on-azure-portal"></a>Включение и отключение преимущества на портале Azure 
Интерфейс портала для включения и отключения АХБ в масштабируемом наборе виртуальных машин **в настоящее время недоступен**.

## <a name="enable-and-disable-the-benefit-using-azure-cli"></a>Включение и отключение преимущества с помощью Azure CLI

`az vmss update`Для обновления существующих виртуальных машин можно использовать команду. Для виртуальных машин RHEL выполните команду с `--license-type` параметром `RHEL_BYOS` . Для виртуальных машин SLES выполните команду с `--license-type` параметром `SLES_BYOS` .

### <a name="cli-example-to-enable-the-benefit"></a>Пример интерфейса командной строки для включения преимущества
```azurecli
# This will enable the benefit on a RHEL VMSS
az vmss update --resource-group myResourceGroup --name myVmName --license-type RHEL_BYOS

# This will enable the benefit on a SLES VMSS
az vmss update --resource-group myResourceGroup --name myVmName --license-type SLES_BYOS
```
### <a name="cli-example-to-disable-the-benefit"></a>Пример интерфейса командной строки для отключения преимущества
Чтобы отключить преимущество, используйте `--license-type` значение `None` :

```azurecli
# This will disable the benefit on a VM
az vmss update -g myResourceGroup -n myVmName --license-type None
```

>[!NOTE]
> Масштабируемые наборы имеют ["политику обновления"](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) , которая определяет, как виртуальные машины обновляются с использованием последней модели масштабируемого набора. Таким образом, если VMSS имеет политику автоматического обновления, преимущество АХБ будет применяться автоматически при обновлении экземпляров виртуальных машин. Если VMSS имеет политику обновления "чередующегося", основанную на запланированных обновлениях, будет применен АХБ.
В случае политики обновления "вручную" потребуется выполнить обновление каждой существующей виртуальной машины вручную.  

### <a name="cli-example-to-upgrade-virtual-machine-scale-set-instances-in-case-of-manual-upgrade-policy"></a>Пример интерфейса командной строки для обновления экземпляров масштабируемого набора виртуальных машин в случае политики обновления вручную 
```azurecli
# This will bring VMSS instances up to date with latest VMSS model 
az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
```

## <a name="apply-the-azure-hybrid-benefit-at-virtual-machine-scale-set-create-time"></a>Применить Преимущество гибридного использования Azure на этапе создания масштабируемого набора виртуальных машин 
В дополнение к применению Преимущество гибридного использования Azure к существующему масштабируемому набору виртуальных машин с оплатой по мере использования, его можно вызвать во время создания масштабируемого набора виртуальных машин. Преимущества этого срифолд:
- Вы можете подготавливать экземпляры масштабируемых наборов виртуальных машин PAYG и BYOS с помощью одного и того же образа и процесса.
- Он позволяет вносить изменения в будущие режимы лицензирования, что недоступно для BYOS-образа.
- Экземпляры масштабируемых наборов виртуальных машин по умолчанию будут подключены к инфраструктуре обновления Red Hat (RHUI), чтобы убедиться в том, что они остаются в актуальном состоянии и защищены. Обновленный механизм можно изменить после развертывания в любое время.

### <a name="cli-example-to-create-virtual-machine-scale-set-with-ahb-benefit"></a>Пример интерфейса командной строки для создания масштабируемого набора виртуальных машин с помощью преимущества АХБ
```azurecli
# This will enable the benefit while creating RHEL VMSS
az vmss create --name myVmName --resource-group myResourceGroup --vnet-name myVnet --subnet mySubnet  --image myRedHatImageURN --admin-username myAdminUserName --admin-password myPassword --instance-count myInstanceCount --license-type RHEL_BYOS 

# This will enable the benefit while creating RHEL VMSS
az vmss create --name myVmName --resource-group myResourceGroup --vnet-name myVnet --subnet mySubnet  --image myRedHatImageURN --admin-username myAdminUserName --admin-password myPassword --instance-count myInstanceCount --license-type SLES_BYOS
```

## <a name="next-steps"></a>Дальнейшие действия
* [Узнайте, как создавать и обновлять виртуальные машины, а также добавлять типы лицензий (RHEL_BYOS, SLES_BYOS) для Преимущество гибридного использования Azure с помощью Azure CLI](/cli/azure/vmss)
