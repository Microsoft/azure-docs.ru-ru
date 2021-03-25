---
title: Создание и включение плана Защиты от атак DDoS Azure с помощью шаблона Azure Resource Manager (шаблона ARM).
description: Узнайте, как создать и включить план Защиты от атак DDoS Azure с помощью шаблона Azure Resource Manager (шаблона ARM).
services: ddos-protection
documentationcenter: na
author: mumian
ms.service: ddos-protection
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.custom: subject-armqs
ms.author: jgao
ms.date: 01/14/2021
ms.openlocfilehash: 75d6c484a0f1d6325aaa7894d8902ff78cadbd74
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99092506"
---
# <a name="quickstart-create-an-azure-ddos-protection-standard-using-arm-template"></a>Краткое руководство. Создание Защиты от атак DDoS Azure категории "Стандартный" с помощью шаблона ARM

В этом кратком руководстве описывается, как с помощью шаблона Azure Resource Manager (шаблона ARM) создать план Защиты от атак DDoS и виртуальную сеть, а затем применить его к этой виртуальной сети. План Защиты от атак DDoS Azure категории "Стандартный" определяет набор виртуальных сетей в нескольких подписках, для которых включена защита от атак DDoS. Вы можете настроить один план защиты от атак DDoS для своей организации и привязать к нему виртуальные сети из нескольких подписок.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-create-and-enable-ddos-protection-plans%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-create-and-enable-ddos-protection-plans).

:::code language="json" source="~/quickstart-templates/101-create-and-enable-ddos-protection-plans/azuredeploy.json":::

Шаблон определяет два ресурса:

- [Microsoft.Network/ddosProtectionPlans](/azure/templates/microsoft.network/ddosprotectionplans)
- [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)

## <a name="deploy-the-template"></a>Развертывание шаблона

Шаблон в этом примере создает новую группу ресурсов, план защиты от атак DDoS и виртуальную сеть.

1. Чтобы войти в Azure и открыть шаблон, нажмите кнопку **Развертывание в Azure**.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-create-and-enable-ddos-protection-plans%2Fazuredeploy.json)

1. Введите значения для создания новой группы ресурсов, плана защиты от атак DDoS и виртуальной сети.

    :::image type="content" source="media/manage-ddos-protection-template/ddos-template.png" alt-text="Шаблон быстрого запуска для защиты от атак DDoS":::

    - **Подписка**: Имя подписки Azure, в которой будут развертываться ресурсы.
    - **Группа ресурсов.** Выберите существующую группу ресурсов или создайте новую.
    - **Регион**. Регион, в котором развертывается группа ресурсов, например "Восточная часть США".
    - **Имя плана защиты от атак DDoS**. Имя нового плана защиты от атак DDoS.
    - **Имя виртуальной сети**. Создает имя новой виртуальной сети.
    - **Расположение.** Здесь используется тот же регион, что и для группы ресурсов, в которой будут развернуты ресурсы.
    - **Префикс адресов виртуальной сети** Сохраните значение по умолчанию или введите нужный адрес виртуальной сети.
    - **Префикс подсети.** Сохраните значение по умолчанию или введите нужный адрес виртуальной подсети.
    - **Включить план защиты от атак DDoS.** По умолчанию здесь указано значение `true` для включение плана защиты от атак DDoS.

1. Выберите **Review + create** (Просмотреть и создать).
1. Убедитесь, что проверка шаблона успешно пройдена, и щелкните **Создать**, чтобы начать развертывание.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Чтобы скопировать команду Azure CLI или Azure PowerShell, щелкните кнопку **Копировать**. Кнопка **Попробовать** открывает окно Azure Cloud Shell, где можно выполнить эту команду.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
az network ddos-protection show \
  --resource-group MyResourceGroup \
  --name MyDdosProtectionPlan
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzDdosProtectionPlan -ResourceGroupName 'MyResourceGroup' -Name 'MyDdosProtectionPlan'
```

---

В выходных данных отобразятся новые ресурсы.

# <a name="cli"></a>[CLI](#tab/CLI)

```Output
{
  "etag": "W/\"abcdefgh-1111-2222-bbbb-987654321098\"",
  "id": "/subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/ddosProtectionPlans/MyDdosProtectionPlan",
  "location": "eastus",
  "name": "MyDdosProtectionPlan",
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
  "resourceGuid": null,
  "tags": null,
  "type": "Microsoft.Network/ddosProtectionPlans",
  "virtualNetworks": [
    {
      "id": "/subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet",
      "resourceGroup": "MyResourceGroup"
    }
  ]
}
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```Output
Name              : MyDdosProtectionPlan
Id                : /subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/ddosProtectionPlans/MyDdosProtectionPlan
Etag              : W/"abcdefgh-1111-2222-bbbb-987654321098"
ProvisioningState : Succeeded
VirtualNetworks   : [
                      {
                        "Id": "/subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet"
                      }
                    ]
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда все будет готово, ресурсы можно удалить. Эта команда удаляет группу ресурсов и все содержащиеся в ней ресурсы.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
az group delete --name MyResourceGroup
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'MyResourceGroup'
```

---

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как просмотреть и настроить данные телеметрии для плана защиты от атак DDoS, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Настройка телеметрии защиты от атак DDoS и просмотр данных телеметрии](telemetry.md)
