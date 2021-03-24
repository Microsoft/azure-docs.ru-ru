---
title: Краткое руководство. Создание Брандмауэра Azure и групп IP-адресов с помощью шаблона Resource Manager
description: В этом кратком руководстве показано, как создать Брандмауэр Azure и группу IP-адресов с помощью шаблона Azure Resource Manager (шаблона ARM).
services: firewall
author: vhorne
ms.service: firewall
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 7966cbe44fc8984cf3c3d9e2d08a11bf599ff158
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89079138"
---
# <a name="quickstart-create-an-azure-firewall-and-ip-groups---arm-template"></a>Краткое руководство. Создание Брандмауэра Azure и групп IP-адресов с помощью шаблона ARM

В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager (шаблона ARM) развернуть Брандмауэр Azure с примерами групп IP-адресов, используемых в правиле сети и приложения. Группа IP-адресов — это ресурс верхнего уровня, позволяющий определять и группировать IP-адреса, диапазоны и подсети в один объект. Это полезно для управления IP-адресами в правилах Брандмауэра Azure. Вы можете либо вручную ввести IP-адреса, либо импортировать их из файла.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-ipgroups-and-linux-jumpbox%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="review-the-template"></a>Изучение шаблона

Этот шаблон создает Брандмауэр Azure и группы IP-адресов, а также необходимые ресурсы для поддержки Брандмауэра Azure.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-azurefirewall-create-with-ipgroups-and-linux-jumpbox).

:::code language="json" source="~/quickstart-templates/101-azurefirewall-create-with-ipgroups-and-linux-jumpbox/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Network/ipGroups**](/azure/templates/microsoft.network/ipGroups)
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)
- [**Microsoft.Network/networkSecurityGroups;**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон ARM в Azure:

1. Выберите элемент **Развертывание в Azure**, чтобы войти в Azure и открыть шаблон. Шаблон создает Брандмауэр Azure, сетевую инфраструктуру и две виртуальные машины.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-ipgroups-and-linux-jumpbox%2Fazuredeploy.json)

2. На портале на странице **Создание Брандмауэра Azure с группами IP-адресов** введите или выберите следующие значения:
   - Подписка: выберите из существующих подписок. 
   - Группа ресурсов:  выберите из существующих групп ресурсов или выберите **Создать** и нажмите кнопку **ОК**.
   - Расположение. Выберите расположение
   - Имя виртуальной сети: введите имя для новой виртуальной сети (VNet). 
   - Имя группы IP-адресов 1: введите имя для группы IP-адресов 1. 
   - Имя группы IP-адресов 2: введите имя для группы IP-адресов 2. 
   - Имя пользователя администратора: введите имя учетной записи пользователя с правами администратора. 
   - Аутентификация: выберите sshPublicKey или пароль. 
   - Пароль администратора: введите пароль администратора или ключ.

3. Установите флажок **Я принимаю указанные выше условия** и нажмите кнопку **Приобрести**. Для развертывания может потребоваться 10 минут или более.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

На портале Azure ознакомьтесь с развернутыми ресурсами, особенно правилами брандмауэра, использующими группы IP-адресов.

:::image type="content" source="media/quick-create-ipgroup-template/ipgroups.png" alt-text="Группы IP-адресов.":::

:::image type="content" source="media/quick-create-ipgroup-template/network-rule.png" alt-text="Правила сети.":::

Дополнительные сведения о синтаксисе JSON и свойствах для брандмауэра в шаблоне см. в статье [Справочник по шаблонам Microsoft.Network azureFirewalls](/azure/templates/Microsoft.Network/2019-11-01/azureFirewalls).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью брандмауэра, удалите группу ресурсов. При этом будут удалены брандмауэры и все связанные с ним ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Развертывание и настройка Брандмауэра Azure в гибридной сети с помощью портала Azure](tutorial-hybrid-portal.md)