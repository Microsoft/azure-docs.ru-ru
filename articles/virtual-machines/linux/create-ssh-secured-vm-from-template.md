---
title: Создание виртуальной машины Linux в Azure с помощью шаблона
description: Как использовать Azure CLI для создания виртуальной машины Linux с помощью шаблона Resource Manager
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 03/22/2019
ms.author: cynthn
ms.openlocfilehash: a4e1bf56df52717255d2bae0a38186335d922ff1
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102554689"
---
# <a name="how-to-create-a-linux-virtual-machine-with-azure-resource-manager-templates"></a>Как создать виртуальную машину Linux с помощью шаблонов Azure Resource Manager

Узнайте, как создать виртуальную машину Linux с помощью шаблона Azure Resource Manager и Azure CLI из Azure Cloud Shell. Сведения о создании виртуальной машины Windows см. в статье [Создание виртуальной машины Windows на основе шаблона диспетчер ресурсов](../windows/ps-template.md).

Альтернативой является развертывание шаблона из портал Azure. Чтобы открыть шаблон на портале, нажмите кнопку **развернуть в Azure** .

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-sshkey%2Fazuredeploy.json)

## <a name="templates-overview"></a>Обзор шаблонов

Шаблоны Azure Resource Manager — это файлы JSON, которые определяют инфраструктуру и конфигурацию решения Azure. Этот шаблон можно использовать, чтобы повторно развертывать решение на протяжении всего его жизненного цикла и гарантировать, что ресурсы развертываются в согласованном состоянии. Дополнительные сведения о формате шаблона и его создании см. в разделе [Краткое руководство. Создание и развертывание шаблонов Azure Resource Manager с помощью портал Azure](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md). См. дополнительные сведения о синтаксисе JSON при [определении ресурсов в шаблонах Azure Resource Manager](/azure/templates/microsoft.compute/allversions).

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создание виртуальной машины Azure обычно включает в себя два этапа:

1. Создайте группу ресурсов. Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Группу ресурсов следует создавать до виртуальной машины.
1. Создайте виртуальную машину.

В следующем примере создается виртуальная машина на основе шаблона быстрого запуска [Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json). Для этого развертывания разрешена только проверка подлинности по протоколу SSH. При запросе укажите значение собственного открытого ключа SSH, например содержимое *~/.ssh/id_rsa.pub*. Если вам необходимо создать пару ключей SSH, см. сведения в статье [Как создать и использовать пару из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](mac-create-ssh-keys.md). Ниже приведена копия шаблона:

[!code-json[create-linux-vm](~/quickstart-templates/101-vm-sshkey/azuredeploy.json)]

Чтобы запустить скрипт CLI, выберите " **попробовать** ", чтобы открыть Azure Cloud Shell. Чтобы вставить скрипт, щелкните оболочку правой кнопкой мыши и выберите команду **Вставить**:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
echo "Enter the project name (used for generating resource names):" &&
read projectName &&
echo "Enter the administrator username:" &&
read username &&
echo "Enter the SSH public key:" &&
read key &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json --parameters projectName=$projectName adminUsername=$username adminPublicKey="$key" &&
az vm show --resource-group $resourceGroupName --name "$projectName-vm" --show-details --query publicIps --output tsv
```

Последняя команда Azure CLI отображает общедоступный IP-адрес созданной виртуальной машины. Для подключения к виртуальной машине требуется общедоступный IP-адрес. См. следующий раздел этой статьи.

В предыдущем примере указан шаблон, хранящийся в GitHub. Вы также можете скачать или создать шаблон и указать локальный путь с параметром `--template-file`.

Ниже приведены некоторые дополнительные ресурсы.

- Чтобы узнать, как создавать шаблоны Resource Manager, ознакомьтесь с [документацией по Azure Resource Manager](../../azure-resource-manager/index.yml).
- Схемы виртуальных машин Azure см. в [справочнике по шаблонам Azure](/azure/templates/microsoft.compute/allversions).
- Дополнительные примеры шаблонов виртуальных машин см. в статье [шаблоны](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Compute&pageNumber=1&sort=Popular)быстрого запуска Azure.

## <a name="connect-to-virtual-machine"></a>Подключение к виртуальной машине

После этого можно подключиться по SSH к виртуальной машине в обычном режиме. Укажите собственный открытый IP-адрес из предыдущей команды:

```bash
ssh <adminUsername>@<ipAddress>
```

## <a name="next-steps"></a>Дальнейшие действия

В этом примере вы создали базовую виртуальную машину Linux. Дополнительные диспетчер ресурсов шаблонов, включающих платформы приложений, или создании более сложных сред см. в статье шаблоны быстрого запуска [Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Compute&pageNumber=1&sort=Popular).

Дополнительные сведения о создании шаблонов, синтаксисе и свойствах JSON для типов ресурсов, которые вы развернули, см. в этих статьях:

- [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups)
- [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
- [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
- [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces)
- [Microsoft. COMPUTE/virtualMachines](/azure/templates/microsoft.compute/virtualmachines)
