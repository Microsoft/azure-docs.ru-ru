---
title: Краткое руководство. Создание Брандмауэра Azure и политики брандмауэра — шаблон Resource Manager
description: В этом кратком руководстве показано, как развернуть брандмауэр Azure и политику брандмауэра.
services: firewall-manager
author: vhorne
ms.author: victorh
ms.date: 02/17/2021
ms.topic: quickstart
ms.service: firewall-manager
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 43853d9e0b955167905af4777d533114a1d1f2ba
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529865"
---
# <a name="quickstart-create-an-azure-firewall-and-a-firewall-policy---arm-template"></a>Краткое руководство. Создание Брандмауэра Azure и политики брандмауэра — шаблон ARM

В этом кратком руководстве показано, как создать Брандмауэр Azure и политику брандмауэра с помощью шаблона Azure Resource Manager (шаблона ARM). Политика брандмауэра имеет правило приложения, которое разрешает подключения к `www.microsoft.com`, и правило, которое разрешает подключения к Центру обновления Windows с помощью тега полного доменного имени **WindowsUpdate**. Правило сети разрешает подключения UDP к серверу времени по адресу 13.86.101.172.

Кроме того, в правилах для определения **исходных** IP-адресов используются группы IP-адресов.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Дополнительные сведения см. в статье [Что такое Диспетчер брандмауэра Azure?](overview.md)

Дополнительные сведения см. в статье [Что такое Брандмауэр Azure?](../firewall/overview.md)

Дополнительные сведения см. в статье [Группы IP-адресов в Брандмауэре Azure](../firewall/ip-groups.md).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="review-the-template"></a>Изучение шаблона

Этот шаблон позволяет создать центральную виртуальную сеть со всеми необходимыми ресурсами для поддержки этого сценария.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups/).

:::code language="json" source="~/quickstart-templates/101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Network/ipGroups**](/azure/templates/microsoft.network/ipGroups)
- [**Microsoft.Network/firewallPolicies**](/azure/templates/microsoft.network/firewallPolicies)
- [**Microsoft.Network/firewallPolicies/ruleCollectionGroups**](/azure/templates/microsoft.network/firewallPolicies/ruleCollectionGroups)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон ARM в Azure:

1. Выберите элемент **Развертывание в Azure**, чтобы войти в Azure и открыть шаблон. Шаблон создает Брандмауэр Azure, виртуальную глобальную сеть и виртуальный концентратор, сетевую инфраструктуру и две виртуальные машины.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups%2Fazuredeploy.json)

2. На портале на странице **Create a Firewall and FirewallPolicy with Rules and Ipgroups** (Создание Брандмауэра и политики брандмауэра с использованием правил и групп IP-адресов) введите или выберите следующие значения:
   - Подписка. Выберите одну из существующих подписок.
   - Группа ресурсов:  выберите из существующих групп ресурсов или выберите **Создать** и нажмите кнопку **ОК**.
   - Регион. Выберите регион.
   - Имя брандмауэра. Укажите имя брандмауэра.

3. Выберите **Просмотр и создание**, а затем щелкните **Создать**. Для развертывания может потребоваться 10 минут или более.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

После завершения развертывания вы увидите следующие ресурсы.

:::image type="content" source="media/quick-firewall-policy/qs-deployed-resources.png" alt-text="Развертываемые ресурсы":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью брандмауэра, удалите группу ресурсов. При этом будут удалены брандмауэры и все связанные с ним ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Общие сведения о политика Диспетчера брандмауэра Azure](policy-overview.md)
