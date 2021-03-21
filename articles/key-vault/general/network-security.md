---
title: Настройка брандмауэров и виртуальных сетей Azure Key Vault — Azure Key Vault
description: Пошаговые инструкции для настройки брандмауэров и виртуальных сетей Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: 13ead70f278f12866dbe00b53c487aebcc43304f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101742642"
---
# <a name="configure-azure-key-vault-firewalls-and-virtual-networks"></a>Настройка брандмауэров и виртуальных сетей Azure Key Vault

В этой статье содержатся рекомендации по настройке брандмауэра Azure Key Vault. В этом документе подробно рассматриваются различные конфигурации брандмауэра Key Vault и предоставляются пошаговые инструкции по настройке Azure Key Vault для работы с другими приложениями и службами Azure.

Дополнительные сведения см. в статье [Конечные точки служб для виртуальной сети для Azure Key Vault](overview-vnet-service-endpoints.md).

## <a name="firewall-settings"></a>Параметры брандмауэра

В этом разделе будут рассмотрены различные способы настройки брандмауэра Azure Key Vault.

### <a name="key-vault-firewall-disabled-default"></a>Брандмауэр Key Vault отключен (по умолчанию)

По умолчанию при создании нового хранилища ключей брандмауэр Azure Key Vault отключен. Все приложения и службы Azure имеют доступ к хранилищу ключей и отправляют запросы в хранилище ключей. Обратите внимание, что эта конфигурация не означает, что любой пользователь сможет выполнять операции в хранилище ключей. Доступ к хранилищу ключей по-прежнему ограничен секретами, ключами и сертификатами, хранящимися в хранилище ключей, что требует от Azure Active Directory разрешений в политике проверки подлинности и доступа. Дополнительные сведения о проверке подлинности в хранилище ключей см. в документе с описанием проверки подлинности в хранилище ключей [здесь](./authentication-fundamentals.md).

### <a name="key-vault-firewall-enabled-trusted-services-only"></a>Брандмауэр Key Vault включен (только доверенные службы)

При включении брандмауэра Key Vault вы получите параметр "Разрешить доверенным службам Майкрософт обходить этот брандмауэр". Список доверенных служб содержит не все службы Azure. Например, Azure DevOps не находится в списке доверенных служб. **Это не означает, что службы, которые не отображаются в списке доверенных служб, не являются доверенными или небезопасны.** Список доверенных служб содержит службы, в которых Майкрософт контролирует весь код, выполняемый в службе. Так как пользователи могут писать пользовательский код в таких службах Azure, как Azure DevOps, корпорация Майкрософт не предоставляет возможность создать утверждение параметров для этой службы. Более того, только то, что служба отображается в списке доверенных служб, не означает, что она разрешена для всех сценариев.

Чтобы определить, находится ли служба, которую вы пытаетесь использовать, в списке доверенных служб, см. документ [здесь](./overview-vnet-service-endpoints.md#trusted-services).

### <a name="key-vault-firewall-enabled-ipv4-addresses-and-ranges---static-ips"></a>Брандмауэр Key Vault включен (адреса и диапазоны IPv4 — статические IP-адреса)

Если вы хотите уполномочить определенную службу для доступа к хранилищу ключей через брандмауэр Key Vault, можно добавить IP-адрес в список разрешений брандмауэра хранилища ключей. Эта конфигурация лучше подходит для служб, использующих статические IP-адреса или известные диапазоны. Для этого случая существует ограничение в 1000 диапазонов CIDR.

Чтобы разрешить IP-адрес или диапазон адресов для ресурса Azure, например веб-приложению или приложению логики, выполните следующие действия.

1. Войдите на портал Azure.
1. Выберите ресурс (определенный экземпляр службы).
1. Просмотрите сведения в колонке "Свойства" в разделе "Параметры".
1. Найдите поле "IP-адрес".
1. Скопируйте это значение или диапазон и введите его в список разрешений брандмауэра Key Vault.

Чтобы разрешить всю службу Azure в брандмауэре Key Vault, используйте список общедоступных IP-адресов центра обработки данных для Azure [здесь](https://www.microsoft.com/download/details.aspx?id=41653). Найдите IP-адреса, связанные со службой, которые вы хотите использовать в нужном регионе, и добавьте эти IP-адреса в брандмауэр Key Vault, выполнив описанные выше действия.

### <a name="key-vault-firewall-enabled-virtual-networks---dynamic-ips"></a>Брандмауэр Key Vault включен (виртуальные сети — динамические IP-адреса)

Если вы пытаетесь разрешить ресурс Azure, например виртуальную машину, в хранилище ключей, возможно, вы не сможете использовать статические IP-адреса и не хотите разрешать всем IP-адресам виртуальных машин Azure доступ к хранилищу ключей.

В этом случае необходимо создать ресурс в виртуальной сети, а затем разрешить трафик из конкретной виртуальной сети и подсети для доступа к хранилищу ключей. Для этого сделайте следующее.

1. Войдите на портал Azure.
1. Выберите хранилище ключей, которое нужно настроить.
1. Перейдите на вкладку "Сеть".
1. Выберите "+Добавить существующую виртуальную сеть"
1. Выберите виртуальную сеть и подсеть, которые вы хотите разрешить в брандмауэре Key Vault.

### <a name="key-vault-firewall-enabled-private-link"></a>Брандмауэр Key Vault включен (Приватный канал)

Чтобы узнать, как настроить подключение к Приватному каналу в хранилище ключей, ознакомьтесь с документом [здесь](./private-link-service.md).

> [!IMPORTANT]
> Когда правила брандмауэра начнут действовать, пользователи смогут выполнять запросы на операции [плоскости данных](secure-your-key-vault.md#data-plane-access-control) в Key Vault только из разрешенных виртуальных сетей или диапазонов IPv4-адресов. Это относится и к получению доступа к Key Vault с портала Azure. Пользователь сможет перейти в хранилище ключей с портала Azure, но не сможет получить список ключей, секретов и сертификатов, если клиентский компьютер не включен в список разрешенных. Это также влияет на выбор хранилища ключей другими службами Azure. Пользователи смогут просматривать список хранилищ ключей, но не список ключей, если правила брандмауэра запрещают доступ их клиентским компьютерам.

> [!NOTE]
> Следует учитывать следующие ограничения конфигурации:
> * Допускается не более 127 правил виртуальной сети и 127 правил IPv4. 
> * Правила IP-сети можно применять только в общедоступных IP-адресах. Диапазоны IP-адресов, зарезервированные для частных сетей (как определено в документе RFC 1918), запрещено использовать в правилах IP. К частным сетям относятся адреса, начинающиеся с **10.** , **172.16-31** и **192.168.** 
> * Сейчас поддерживаются только IPV4-адреса.

## <a name="use-the-azure-portal"></a>Использование портала Azure

Брандмауэры и виртуальные сети Key Vault можно настроить с помощью портала Azure следующим образом:

1. Перейдите к хранилищу ключей, которое нужно защитить.
2. Щелкните **Сеть** и выберите вкладку **Брандмауэры и виртуальные сети**.
3. В разделе **Разрешить доступ из** щелкните **Выбранные сети**.
4. Чтобы добавить имеющиеся виртуальные сети в правила брандмауэров и виртуальных сетей, выберите **+ Добавить существующие виртуальные сети**.
5. В новой открывшейся колонке выберите подписку, виртуальные сети и подсети, которым нужно предоставить доступ к этому хранилищу ключей. Если в выбранных вами виртуальных сетях и подсетях нет включенных конечных точек службы, подтвердите, что вы хотите включить конечные точки службы, и нажмите кнопку **Включить**. Эта настройка вступит в силу в течение 15 минут.
6. В разделе **IP-сети** добавьте диапазоны IPv4-адресов, введя их с использованием [нотации CIDR](https://tools.ietf.org/html/rfc4632), или укажите отдельные IP-адреса.
7. Если вы хотите разрешить доверенным службам Майкрософт обход брандмауэра Key Vault, выберите "Да". Полный список текущих доверенных служб Key Vault см. по следующей ссылке. [Доверенные службы Azure Key Vault](./overview-vnet-service-endpoints.md#trusted-services)
7. Щелкните **Сохранить**.

Вы также можете добавить новые виртуальные сети и подсети, а потом включить конечные точки службы для них, выбрав **+ Добавить новую виртуальную сеть**. Затем следуйте инструкциям на экране.

## <a name="use-the-azure-cli"></a>Использование Azure CLI 

Брандмауэры и виртуальные сети Key Vault можно настроить с помощью Azure CLI следующим образом:

1. [Установите Azure CLI](/cli/azure/install-azure-cli) и [войдите в систему](/cli/azure/authenticate-azure-cli).

2. Перечислите доступные правила виртуальной сети. Если вы не настроили каких-либо правил для этого хранилища ключей, список будет пуст.
   ```azurecli
   az keyvault network-rule list --resource-group myresourcegroup --name mykeyvault
   ```

3. Включите конечную точку службы для Key Vault в имеющейся виртуальной сети и подсети.
   ```azurecli
   az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.KeyVault"
   ```

4. Добавьте сетевое правило для виртуальной сети и подсети.
   ```azurecli
   subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
   az keyvault network-rule add --resource-group "demo9311" --name "demo9311premium" --subnet $subnetid
   ```

5. Добавьте диапазон IP-адресов, из которого необходимо разрешать трафик.
   ```azurecli
   az keyvault network-rule add --resource-group "myresourcegroup" --name "mykeyvault" --ip-address "191.10.18.0/24"
   ```

6. Если это хранилище ключей должно быть доступно для каких-либо доверенных служб, задайте для параметра `bypass` значение `AzureServices`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mykeyvault" --bypass AzureServices
   ```

7. Включите правила сети, установив для действия по умолчанию значение `Deny`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mekeyvault" --default-action Deny
   ```

## <a name="use-azure-powershell"></a>Использование Azure PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Брандмауэры и виртуальные сети Key Vault можно настроить с помощью PowerShell следующим образом:

1. Установите последнюю версию [Azure PowerShell](/powershell/azure/install-az-ps) и [выполните вход](/powershell/azure/authenticate-azureps).

2. Перечислите доступные правила виртуальной сети. Если вы не настроили каких-либо правил для этого хранилища ключей, список будет пуст.
   ```powershell
   (Get-AzKeyVault -VaultName "mykeyvault").NetworkAcls
   ```

3. Включите конечную точку службы для Key Vault в имеющейся виртуальной сети и подсети.
   ```powershell
   Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.1.1.0/24" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
   ```

4. Добавьте сетевое правило для виртуальной сети и подсети.
   ```powershell
   $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -VirtualNetworkResourceId $subnet.Id
   ```

5. Добавьте диапазон IP-адресов, из которого необходимо разрешать трафик.
   ```powershell
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -IpAddressRange "16.17.18.0/24"
   ```

6. Если это хранилище ключей должно быть доступно для каких-либо доверенных служб, задайте для параметра `bypass` значение `AzureServices`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -Bypass AzureServices
   ```

7. Включите правила сети, установив для действия по умолчанию значение `Deny`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -DefaultAction Deny
   ```

## <a name="references"></a>Ссылки
* Справочник по шаблонам ARM: [Справочник по шаблону ARM Azure Key Vault](/azure/templates/Microsoft.KeyVault/vaults).
* Команда Azure CLI: [az keyvault network-rule](/cli/azure/keyvault/network-rule).
* Командлеты Azure PowerShell: [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet).

## <a name="next-steps"></a>Дальнейшие действия

* [Конечные точки служб для виртуальной сети для Key Vault](overview-vnet-service-endpoints.md)
* [Обзор системы безопасности Azure Key Vault](security-overview.md)
