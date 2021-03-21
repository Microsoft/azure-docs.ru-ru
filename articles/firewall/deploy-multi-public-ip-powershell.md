---
title: Развертывание брандмауэра Azure с несколькими общедоступными IP-адресами с помощью PowerShell
description: Из этой статьи вы узнаете, как развернуть брандмауэр Azure с несколькими общедоступными IP-адресами с помощью Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 05/06/2020
ms.author: victorh
ms.openlocfilehash: cbad025a0d0c4d679ea9cdc7557c81b5145798fd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96023675"
---
# <a name="deploy-an-azure-firewall-with-multiple-public-ip-addresses-using-azure-powershell"></a>Развертывание Брандмауэра Azure с использованием нескольких общедоступных IP-адресов с помощью Azure PowerShell

Эта функция позволяет выполнять следующие сценарии.

- **DNAT**. Позволяет преобразовать несколько стандартных экземпляров порта для внутренних серверов. Например, если существует два общедоступных IP-адреса, то для обоих IP-адреса можно преобразовать TCP-порт 3389 (RDP).
- **SNAT**. Дополнительные порты доступны для исходящих SNAT подключений, что снижает вероятность нехватки SNAT портов. На данный момент Брандмауэр Azure случайно выбирает общедоступные IP-адреса источника, которые можно использовать для подключения. Если в сети установлена любая фильтрация, необходимо разрешить все публичные IP-адреса, которые связанные с брандмауэром. Чтобы упростить эту настройку, можно использовать [префикс общедоступного IP-адреса](../virtual-network/public-ip-address-prefix.md).
 
Портал Azure, Azure PowerShell, Azure CLI, REST и шаблоны позволяют использовать брандмауэр Azure с несколькими общедоступными IP-адресами. Вы можете развернуть брандмауэр Azure, содержащий до 250 общедоступных IP-адресов.

В следующих примерах Azure PowerShell показано, как можно настроить, добавить и удалить общедоступные IP-адреса для брандмауэра Azure.

> [!NOTE]
> Вы не можете удалить первую конфигурацию с страницы конфигурации общедоступного IP-адреса брандмауэра Azure. Если вы хотите изменить IP-адрес, можно использовать Azure PowerShell.

## <a name="create-a-firewall-with-two-or-more-public-ip-addresses"></a>Создание брандмауэра с двумя или более общедоступными IP-адресами

В этом примере создается брандмауэр, подключенный к *виртуальной сети виртуальных сетей с* двумя общедоступными IP-адресами.

```azurepowershell
$rgName = "resourceGroupName"

$vnet = Get-AzVirtualNetwork `
  -Name "vnet" `
  -ResourceGroupName $rgName

$pip1 = New-AzPublicIpAddress `
  -Name "AzFwPublicIp1" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

$pip2 = New-AzPublicIpAddress `
  -Name "AzFwPublicIp2" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

New-AzFirewall `
  -Name "azFw" `
  -ResourceGroupName $rgName `
  -Location centralus `
  -VirtualNetwork $vnet `
  -PublicIpAddress @($pip1, $pip2)
```

## <a name="add-a-public-ip-address-to-an-existing-firewall"></a>Добавление общедоступного IP-адреса в существующий брандмауэр

В этом примере общедоступный IP-адрес *azFwPublicIp1* подключен к брандмауэру.

```azurepowershell
$pip = New-AzPublicIpAddress `
  -Name "azFwPublicIp1" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

$azFw = Get-AzFirewall `
  -Name "AzureFirewall" `
  -ResourceGroupName "rg"

$azFw.AddPublicIpAddress($pip)

$azFw | Set-AzFirewall
```

## <a name="remove-a-public-ip-address-from-an-existing-firewall"></a>Удаление общедоступного IP-адреса из существующего брандмауэра

В этом примере общедоступный IP-адрес *azFwPublicIp1* отсоединяется от брандмауэра.

```azurepowershell
$pip = Get-AzPublicIpAddress `
  -Name "azFwPublicIp1" `
  -ResourceGroupName "rg"

$azFw = Get-AzFirewall `
  -Name "AzureFirewall" `
  -ResourceGroupName "rg"

$azFw.RemovePublicIpAddress($pip)

$azFw | Set-AzFirewall
```

## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство. Создание Брандмауэра Azure с несколькими общедоступными IP-адресами, используя шаблон Resource Manager](quick-create-multiple-ip-template.md)
