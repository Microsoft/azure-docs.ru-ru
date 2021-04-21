---
title: Создание участника в службе "Блокчейн Azure" с помощью Azure PowerShell
description: Создание участника блокчейн-консорциума в службе "Блокчейн Azure" с помощью Azure PowerShell.
ms.reviewer: ravastra
ms.date: 9/22/2020
ms.topic: quickstart
ms.custom:
- references_regions
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 9087bda9c1979a98f4bbf9f8343d012c4cf3098c
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529639"
---
# <a name="quickstart-create-an-azure-blockchain-service-blockchain-member-using-azure-powershell"></a>Краткое руководство. Создание участника блокчейна в службе "Блокчейн Azure" с помощью Azure PowerShell

В этом кратком руководстве показано, как развернуть нового участника блокчейн-сети и консорциум в службе "Блокчейн Azure" с помощью Azure PowerShell.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Так как модуль PowerShell **Az.Blockchain** предоставляется в предварительной версии, его нужно установить отдельно от модуля Az PowerShell с помощью командлета `Install-Module`. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Blockchain
```

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

## <a name="register-resource-provider"></a>Регистрация поставщика ресурсов

Если вы впервые используете службу "Блокчейн Azure", зарегистрируйте поставщик ресурсов **Microsoft.Blockchain**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Blockchain
```

## <a name="choose-a-specific-azure-subscription"></a>Выбор требуемой подписки Azure

Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите требуемую подписку с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="define-variables"></a>Определение переменных

Некоторый фрагменты данных будут многократно использоваться. Создайте переменные для хранения информации.

```azurepowershell-interactive
# Name of resource group used throughout this article
$resourceGroupName = 'myResourceGroup'

# Azure region
$location = 'eastus'
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../../azure-resource-manager/management/overview.md) с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов на основе имени в переменной `$resourceGroupName` в регионе, указанном в переменной `$location`.

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

## <a name="create-a-blockchain-member"></a>Создание участника блокчейн-сети

Элемент в службе "Azure Блокчейн" — это блокчейн-узел в частной сети блокчейн-консорциума.
При подготовке элемента можно создать сеть консорциума или присоединиться к ней. Для сети консорциума требуется хотя бы один элемент. Количество элементов блокчейн-сети, необходимых участникам консирцуима, зависит от вашего сценария. У участников консорциума может быть один или несколько элементов блокчейн-сети или участники могут использовать их совместно. Дополнительные сведения о консорциумах см. в разделе [Консорциум службы "Блокчейн Azure"](consortium.md).

Для этого вам нужно передать несколько параметров и свойств. Замените примеры параметров своими значениями.

```azurepowershell-interactive
$passwd = Read-Host -Prompt 'Enter the members default transaction node password' -AsSecureString
$csPasswd = Read-Host -Prompt 'Enter  the consortium account password' -AsSecureString

$memberParams = @{
  Name = 'myblockchainmember'
  ResourceGroupName = $resourceGroupName
  Consortium = 'myconsortium'
  ConsortiumManagementAccountPassword = $csPasswd
  Location = $location
  Password = $passwd
  Protocol = 'Quorum'
  Sku = 'S0'
}
New-AzBlockchainMember @memberParams
```

| Параметр | Описание |
|---------|-------------|
| **ResourceGroupName** | Имя группы ресурсов, в которой создаются ресурсы службы "Блокчейн Azure". Выберите группу ресурсов, созданную при работе с предыдущим разделом.
| **Имя** | Уникальное имя, идентифицирующее участника блокчейн-сети в службе "Блокчейн Azure". Это имя используется в адресе общедоступной конечной точки. Например, `myblockchainmember.blockchain.azure.com`.
| **Местоположение** | Регион Azure, в котором создается участник блокчейн-сети. Например, `westus2`. Выберите расположение, наиболее близкое к пользователям или другим приложениям Azure. Компоненты могут быть недоступны в некоторых регионах. Диспетчер данных блокчейна Azure доступен в таких регионах Azure: восточная часть США и Западная Европа.
| **Пароль** | Пароль для узла транзакций по умолчанию участника. Этот пароль используется для обычной проверки подлинности при подключении к общедоступной конечной точке узла транзакций участника блокчейна.
| **протокол**; | Протокол блокчейна. В настоящее время поддерживается протокол _Quorum_.
| **Consortium** | Имя консорциума, который создается или к которому присоединяется участник. Дополнительные сведения о консорциумах см. в разделе [Консорциум службы "Блокчейн Azure"](consortium.md).
| **ConsortiumManagementAccountPassword** | Пароль учетной записи консорциума, также называется паролем учетной записи участника. Пароль учетной записи участника используется для шифрования закрытого ключа для учетной записи Ethereum, созданного для вашего участника. Учетная запись участника и ее пароль используются для управления консорциумом.
| **SKU** | Тип уровня. **S0** для уровня "Стандартный" или **B0** для уровня "Базовый". Используйте уровень _Базовый_ для разработки, тестирования и подтверждения концепций. Используйте уровень _Стандартный_ для рабочих развертываний. Если вы работаете с Диспетчером данных блокчейна или отправляете большой объем частных транзакций, используйте ценовую категорию _Стандартный_. Изменение ценовой категории "Базовый" и "Стандартный" после создания элемента не поддерживается.

Для создания участника блокчейн-сети и вспомогательных ресурсов требуется около 10 минут.

## <a name="clean-up-resources"></a>Очистка ресурсов

Данные о созданном участнике блокчейн-сети можно использовать для работы со следующим кратким руководством или учебником. Если ресурсы больше не нужны, вы можете удалить их. Для этого удалите группу ресурсов `myResourceGroup`, которую создали для этого краткого руководства.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этой статьи, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы развернули участника службы "Блокчейн Azure" и новый консорциум. Ознакомьтесь со следующим кратким руководством, чтобы использовать комплект SDK службы "Блокчейн Azure" для Ethereum для присоединения к элементу в службе "Блокчейн Azure".

> [!div class="nextstepaction"]
> [Краткое руководство. Подключение к сети консорциума службы "Блокчейн Azure" с помощью Visual Studio Code](connect-vscode.md)
