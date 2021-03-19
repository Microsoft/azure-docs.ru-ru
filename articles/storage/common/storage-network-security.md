---
title: Настройка брандмауэров службы хранилища Azure и виртуальных сетей | Документация Майкрософт
description: Настройка многоуровневой сетевой безопасности для учетной записи хранения с помощью брандмауэров службы хранилища Azure и виртуальной сети Azure.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 03/16/2021
ms.author: normesta
ms.reviewer: santoshc
ms.subservice: common
ms.openlocfilehash: 3d71a7ad2507909dacf54e7f1c49b6e768033113
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104600485"
---
# <a name="configure-azure-storage-firewalls-and-virtual-networks"></a>Настройка брандмауэров службы хранилища Azure и виртуальных сетей

Служба хранилища Azure предоставляет многоуровневую модель безопасности. Эта модель позволяет защищать и контролировать уровень доступа к учетным записям хранения, которые требуются приложениям и корпоративным средам, в зависимости от типа и подмножества используемых сетей или ресурсов. При настройке сетевых правил только приложения, запрашивающие данные через указанный набор сетей или через указанный набор ресурсов Azure, могут получить доступ к учетной записи хранения. Вы можете ограничить доступ к учетной записи хранения запросами, источником которых являются указанные IP-адреса, диапазоны IP-адресов, подсети в виртуальной сети Azure или экземпляры ресурсов некоторых служб Azure.

Учетные записи хранения имеют общедоступную конечную точку, доступную через Интернет. Вы также можете создать [частные конечные точки для учетной записи хранения](storage-private-endpoints.md), которые назначают учетной записи хранения частный IP-адрес из вашей виртуальной сети, а также защищают весь трафик между виртуальной сетью и учетной записью хранения с помощью приватного канала. Брандмауэр службы хранилища Azure предоставляет контроль доступа для общедоступной конечной точки вашей учетной записи хранения. Вы также можете использовать брандмауэр, чтобы блокировать доступ через общедоступную конечную точку при использовании частных конечных точек. Конфигурация брандмауэра службы хранилища также позволяет выбрать доверенные службы платформы Azure для безопасного доступа к учетной записи хранения.

При этом приложению, которое обращается к учетной записи хранения, когда действуют сетевые правила, по-прежнему необходима надлежащая авторизация для выполнения запроса. Авторизация обеспечивается учетными данными Azure Active Directory для больших двоичных объектов и очередей, действительным ключом доступа к учетной записи или маркером SAS.

> [!IMPORTANT]
> Включение правил брандмауэра для учетной записи хранения по умолчанию блокирует входящие запросы данных, за исключением запросов от служб, работающих внутри виртуальной сети Azure (VNet), или запросов, поступающих из разрешенных общедоступных IP-адресов. Запросы от других служб Azure, в том числе портала Azure, служб метрики и ведения журналов, блокируются.
>
> Вы можете предоставить доступ службам Azure, работающим в рамках виртуальной сети, разрешая трафик из подсети, в которой размещен экземпляр службы. Можно также включить ограниченное количество сценариев с помощью механизма исключений, описанного ниже. Доступ к данным учетной записи хранения через портал Azure возможен только с компьютера, находящегося в пределах настроенной доверенной границы (на основе протокола IP или VNet).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="scenarios"></a>Сценарии

Чтобы защитить учетную запись хранения, необходимо сначала настроить правило, по умолчанию запрещающее доступ к трафику из всех сетей (включая интернет-трафик) в общедоступной конечной точке. Затем следует настроить правила, предоставляющие доступ к трафику из определенных виртуальных сетей. Можно также настроить правила, чтобы предоставить доступ к трафику из выбранных диапазонов общедоступных IP-адресов Интернета, разрешив подключения от конкретных клиентов в Интернете или локальных клиентах. Такая конфигурация позволяет создать для приложений границу в виде безопасной сети.

Вы можете сочетать правила брандмауэра, разрешающие доступ из конкретных виртуальных сетей, и правила, разрешающие доступ из диапазонов общедоступных IP-адресов, в одной и той же учетной записи хранения. Правила брандмауэра службы хранилища могут применяться к существующим учетным записям хранения, а также во время создания учетных записей хранения.

Правила брандмауэра службы хранилища применяются к общедоступной конечной точке учетной записи хранения. Вам не нужно задавать никакие правила доступа брандмауэра, чтобы разрешить трафик для частных конечных точек учетной записи хранения. Процесс утверждения создания частной конечной точки предоставляет неявный доступ к трафику из подсети, в которой размещается эта частная конечная точка.

Сетевые правила применяются ко всем сетевым протоколам в службе хранилища Azure, включая протоколы RESTFUL и SMB. Для доступа к данным с помощью таких инструментов, как портал Azure, Обозреватель службы хранилища и AZCopy, необходимо настроить явные правила сети.

Как только правила сети применены, они распространяются на все запросы. Токены SAS, позволяющие получить доступ к конкретному IP-адресу, служат для ограничения доступа владельца токена, а не предоставляют доступ к каким-либо новым ресурсам, не указанным в настроенных сетевых правилах.

Правила сети не влияют на трафик дисков виртуальных машин (включая операции подключения и отключения, а также дисковые операции ввода-вывода). Сетевые правила обеспечивают безопасность во время доступа REST к страничным BLOB-объектам.

Классические учетные записи хранения не поддерживают брандмауэры и виртуальные сети.

Вы можете использовать неуправляемые диски в учетных записях хранения с правилами сети, применяемыми для резервного копирования и восстановления виртуальных машин, создав исключение. Этот процесс описан в разделе " [Управление исключениями](#manage-exceptions) " этой статьи. Исключения брандмауэра не применяются к управляемым дискам, так как ими уже управляет Azure.

## <a name="change-the-default-network-access-rule"></a>Изменение сетевого правила доступа по умолчанию

По умолчанию учетные записи хранения принимают запросы на подключение от клиентов в сети. Для ограничения доступа из выбранных сетей необходимо сначала изменить действие по умолчанию.

> [!WARNING]
> Изменение сетевых правил может повлиять на возможность подключения приложений к службе хранилища Azure. Когда для сетевого правила по умолчанию указано значение **deny**, доступ к данным блокируется полностью, если только не применяются специальные сетевые правила, **разрешающие** доступ. Предоставьте доступ для разрешенных сетей с помощью сетевых правил, прежде чем изменить правила по умолчанию и запретить доступ.

### <a name="managing-default-network-access-rules"></a>Управление сетевыми правилами доступа по умолчанию

Вы можете управлять сетевыми правилами доступа по умолчанию для учетных записей хранения с помощью портала Azure, PowerShell или CLI версии 2.

#### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к учетной записи хранения, которую нужно защитить.

2. Выберите в меню Параметры пункт **Сетевые подключения**.

3. Чтобы запретить доступ по умолчанию, разрешите доступ в разделе **Выбранные сети**. Чтобы разрешить передачу трафика из всех сетей, разрешите доступ в разделе **Все сети**.

4. Выберите **Сохранить**, чтобы применить изменения.

<a id="powershell"></a>

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Установите [Azure PowerShell](/powershell/azure/install-Az-ps) и [выполните вход](/powershell/azure/authenticate-azureps).

2. Отобразите состояние правила по умолчанию для учетной записи хранения.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").DefaultAction
    ```

3. Настройте в правиле по умолчанию запрет сетевого доступа по умолчанию.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Deny
    ```

4. Настройте в правиле по умолчанию разрешение сетевого доступа по умолчанию.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Allow
    ```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Установите [Azure CLI](/cli/azure/install-azure-cli) и [выполните вход](/cli/azure/authenticate-azure-cli).

2. Отобразите состояние правила по умолчанию для учетной записи хранения.

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.defaultAction
    ```

3. Настройте в правиле по умолчанию запрет сетевого доступа по умолчанию.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Deny
    ```

4. Настройте в правиле по умолчанию разрешение сетевого доступа по умолчанию.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Allow
    ```
---

## <a name="grant-access-from-a-virtual-network"></a>Предоставление доступа из виртуальной сети

Вы можете настроить учетные записи хранения таким образом, чтобы они разрешали доступ только из определенных подсетей. Разрешенные подсети могут относиться к виртуальной сети как в той же, так и в другой подписке, включая подписки, принадлежащие другому клиенту Azure Active Directory.

Включите [конечную точку службы](../../virtual-network/virtual-network-service-endpoints-overview.md) для службы хранилища Azure, входящей в нужную виртуальную сеть. Конечная точка службы направляет трафик из виртуальной сети в службу хранилища Azure по оптимальному пути. В каждом запросе также передаются удостоверения подсети и виртуальной сети. Затем администраторы могут настроить сетевые правила для учетной записи хранения, разрешающие получение запросов из определенных подсетей в виртуальной сети. Клиенты, которым предоставлен доступ по этим сетевым правилам, по прежнему должны выполнять требования авторизации учетной записи хранения, чтобы получать доступ к данным.

Каждая учетная запись хранения поддерживает до 200 правил виртуальной сети, которые могут сочетаться с [правилами IP-сети](#grant-access-from-an-internet-ip-range).

### <a name="available-virtual-network-regions"></a>Доступные регионы виртуальной сети

Как правило, конечные точки служб работают между виртуальными сетями и экземплярами служб в одном регионе Azure. При использовании конечных точек служб со службой хранилища Azure область применения расширяется и включает [связанный регион](../../best-practices-availability-paired-regions.md). Конечные точки служб обеспечивают непрерывность работы при отработке регионального отказа, а также доступ на чтение к экземплярам геоизбыточного хранилища (RA-GRS). Правила сети, предоставляющие доступ к учетной записи хранения из виртуальной сети, также предоставляют доступ к любому экземпляру RA-GRS.

Планируя аварийное восстановление на случай регионального сбоя, следует заранее создать виртуальные сети в связанном регионе. Включите конечные точки для службы хранилища Azure с применением сетевых правил, разрешающих доступ из этих альтернативных виртуальных сетей. Затем примените эти правила к учетным записям своего геоизбыточного хранилища.

> [!NOTE]
> Конечные точки службы не применяются к трафику за пределами региона виртуальной сети и заданного связанного региона. Сетевые правила, предоставляющие доступ к учетным записям хранения из виртуальных сетей, применяются только в основном регионе учетной записи хранения или в заданном связанном регионе.

### <a name="required-permissions"></a>Необходимые разрешения

Чтобы применить правило виртуальной сети к учетной записи хранения, у пользователя должны быть соответствующие разрешения для добавляемых подсетей. Применение правила может выполнять [участник учетной записи хранения](../../role-based-access-control/built-in-roles.md#storage-account-contributor) или пользователь, которому было предоставлено разрешение на `Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftnetwork) с помощью настраиваемой роли Azure.

Учетная запись хранения и виртуальные сети, которым предоставлен доступ, могут находиться в разных подписках, включая подписки, являющиеся частью другого клиента Azure AD.

> [!NOTE]
> Настройка правил, предоставляющих доступ к подсетям в виртуальных сетях, которые являются частью другого клиента Azure Active Directory, в настоящее время поддерживаются только с помощью PowerShell, интерфейса командной строки и интерфейсов REST API. Такие правила нельзя настроить с помощью портала Azure, хотя на портале их можно просматривать.

### <a name="managing-virtual-network-rules"></a>Управление правилами виртуальной сети

Правилами виртуальной сети для учетных записей хранения можно управлять с помощью портала Azure, PowerShell или CLI версии 2.

#### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к учетной записи хранения, которую нужно защитить.

2. Выберите в меню Параметры пункт **Сетевые подключения**.

3. Убедитесь, что вы разрешили доступ в разделе **Выбранные сети**.

4. Чтобы предоставить доступ к виртуальной сети с помощью нового правила сети, в разделе **виртуальные сети** выберите **Добавить существующую виртуальную сеть**, выберите **Параметры виртуальных сетей** и **подсетей** , а затем нажмите кнопку **Добавить**. Чтобы создать новую виртуальную сеть и предоставить ей доступ, выберите **Добавить новую виртуальную сеть**. Укажите сведения, необходимые для создания новой виртуальной сети, а затем нажмите кнопку **создать**.

    > [!NOTE]
    > Если конечная точка службы для службы хранилища Azure еще не настроена для выбранной виртуальной сети и подсетей, ее можно настроить в ходе этой операции.
    >
    > В настоящее время при создании правила можно выбирать только из виртуальных сетей, принадлежащих одному и тому же клиенту Azure Active Directory. Чтобы предоставить доступ к подсети в виртуальной сети, принадлежащей другому клиенту, используйте PowerShell, интерфейс командной строки или интерфейсы REST API.

5. Чтобы удалить правило виртуальной сети или подсети, выберите **...** , чтобы открыть контекстное меню для виртуальной сети или подсети, и выберите **Удалить**.

6. Нажмите кнопку **сохранить** , чтобы применить изменения.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Установите [Azure PowerShell](/powershell/azure/install-Az-ps) и [выполните вход](/powershell/azure/authenticate-azureps).

2. Выведите список правил виртуальной сети.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").VirtualNetworkRules
    ```

3. Включите конечную точку службы для службы хранилища Azure в имеющейся виртуальной сети и подсети.

    ```powershell
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.0.0.0/24" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    ```

4. Добавьте сетевое правило для виртуальной сети и подсети.

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

    > [!TIP]
    > Чтобы добавить сетевое правило для подсети, принадлежащей другому клиенту Azure AD, используйте полный параметр **VirtualNetworkResourceId** в форме "/подписки/ИД_подписки/группыРесурсов/Имя_группыРесурсов/поставщики/Microsoft.Network/виртуальныеСети/имя_виртуальнойСети/подсети/имя_подсети".

5. Удалите сетевое правило для виртуальной сети и подсети.

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе сетевые правила не будут действовать.

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Установите [Azure CLI](/cli/azure/install-azure-cli) и [выполните вход](/cli/azure/authenticate-azure-cli).

2. Выведите список правил виртуальной сети.

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query virtualNetworkRules
    ```

3. Включите конечную точку службы для службы хранилища Azure в имеющейся виртуальной сети и подсети.

    ```azurecli
    az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.Storage"
    ```

4. Добавьте сетевое правило для виртуальной сети и подсети.

    ```azurecli
    subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

    > [!TIP]
    > Чтобы добавить правило для подсети, принадлежащей другому клиенту Azure AD, используйте полный идентификатор подсети в формате "/Subscriptions/ \<subscription-ID\> /ResourceGroups/ \<resourceGroup-Name\> /провидерс/Микрософт.Нетворк/виртуалнетворкс/ \<vNet-name\> /субнетс/ \<subnet-name\> ".
    >
    > Чтобы получить идентификатор подсети для виртуальной сети, принадлежащей другому клиенту Azure AD, можно использовать параметр **subscription**.

5. Удалите сетевое правило для виртуальной сети и подсети.

    ```azurecli
    subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе сетевые правила не будут действовать.

---

## <a name="grant-access-from-an-internet-ip-range"></a>Предоставление доступа из диапазона IP-адресов в Интернете

Правила IP-сети можно использовать для предоставления доступа к конкретным общедоступным диапазонам IP-адресов путем создания правил IP-сети. Каждая учетная запись хранения поддерживает до 200 правил. Эти правила предоставляют доступ к конкретным службам Интернета и локальным сетям и блокируют общий Интернет-трафик.

К диапазонам IP-адресов применяются следующие ограничения.

- Правила IP-сети разрешены только для **общедоступных** IP-адресов. 

  Диапазоны IP-адресов, зарезервированные для частных сетей (как определено в документе [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3)), запрещено использовать в правилах IP. К частным сетям относятся адреса, начинающиеся с _10.*_ , _172.16.*_  - _172.31.*_ и _192.168.*_ .

- Необходимо указать допустимые диапазоны адресов Интернета, используя [нотацию CIDR](https://tools.ietf.org/html/rfc4632) в формате *16.17.18.0/24* или как отдельные IP-адреса, например *16.17.18.19*. 

- Небольшие адреса, использующие размеры с префиксом /31 или /32, не поддерживаются. Эти диапазоны следует настраивать с помощью отдельных правил для IP-адресов. 

- Для настройки правил брандмауэра службы хранилища поддерживаются только IPv4-адреса.

Правила IP-сети нельзя использовать в следующих случаях:

- Чтобы ограничить доступ к клиентам в том же регионе Azure, что и учетная запись хранения.
  
  Правила IP-сети не затрагивают запросы, исходящие из того же региона Azure, к которому относится учетная запись хранения. Используйте [правила виртуальной сети](#grant-access-from-a-virtual-network) для разрешения запросов из того же региона. 

- Для ограничения доступа клиентов в [парном регионе](../../best-practices-availability-paired-regions.md) , который находится в виртуальной сети с конечной точкой службы.

- Чтобы ограничить доступ к службам Azure, развернутым в том же регионе, что и учетная запись хранения.

  Службы, развернутые в том же регионе, что и учетная запись хранения, используют для обмена данными частные IP-адреса Azure. Поэтому нельзя ограничить доступ к определенным службам Azure на основе диапазона общедоступных исходящих IP-адресов.

### <a name="configuring-access-from-on-premises-networks"></a>Настройка доступа из локальных сетей

Чтобы предоставить доступ из вашей локальной сети к учетной записи хранения в правиле IP-сети, необходимо определить IP-адреса для Интернета, которые используются в вашей сети. За помощью обращайтесь к администратору сети.

Если вы используете [ExpressRoute](../../expressroute/expressroute-introduction.md) для локальной среды, для общедоступного пиринга или пиринга Майкрософт, то вам необходимо определить используемые IP-адреса NAT. Для общедоступного пиринга в каждом канале ExpressRoute по умолчанию используется два IP-адреса NAT для трафика служб Azure, когда он входит в основную магистральную сеть Microsoft Azure. Для пиринга Майкрософт используются IP-адреса NAT, предоставленные либо клиентом, либо поставщиком услуг. Чтобы разрешить доступ к ресурсам служб, необходимо разрешить эти общедоступные IP-адреса в настройках брандмауэра IP-адресов ресурсов. Чтобы найти IP-адреса канала ExpressRoute для общедоступного пиринга, [отправьте запрос по ExpressRoute в службу поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) через портал Azure. Узнайте больше о [NAT для общедоступного пиринга и пиринга Майкрософт](../../expressroute/expressroute-nat.md#nat-requirements-for-azure-public-peering).

### <a name="managing-ip-network-rules"></a>Управление правилами IP-сети

Правилами IP-сети для учетных записей хранения можно управлять с помощью портала Azure, PowerShell или CLI версии 2.

#### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к учетной записи хранения, которую нужно защитить.

2. Выберите в меню Параметры пункт **Сетевые подключения**.

3. Убедитесь, что вы разрешили доступ в разделе **Выбранные сети**.

4. Чтобы предоставить доступ к диапазону IP-адресов в Интернете, введите IP-адрес или диапазон адресов (в формате CIDR) в разделе **Брандмауэр** > **Диапазон адресов**.

5. Чтобы удалить правило IP-сети, щелкните значок корзины рядом с диапазоном адресов.

6. Выберите **Сохранить**, чтобы применить изменения.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Установите [Azure PowerShell](/powershell/azure/install-Az-ps) и [выполните вход](/powershell/azure/authenticate-azureps).

2. Выведите список правил IP-сети.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").IPRules
    ```

3. Добавьте правило сети для отдельного IP-адреса.

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

4. Добавьте правило сети для диапазона IP-адресов.

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

5. Удалите правило сети для отдельного IP-адреса.

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

6. Удалите правило сети для диапазона IP-адресов.

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе сетевые правила не будут действовать.

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Установите [Azure CLI](/cli/azure/install-azure-cli) и [выполните вход](/cli/azure/authenticate-azure-cli).

1. Выведите список правил IP-сети.

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query ipRules
    ```

2. Добавьте правило сети для отдельного IP-адреса.

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

3. Добавьте правило сети для диапазона IP-адресов.

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

4. Удалите правило сети для отдельного IP-адреса.

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

5. Удалите правило сети для диапазона IP-адресов.

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе сетевые правила не будут действовать.

---

<a id="grant-access-specific-instances"></a>

## <a name="grant-access-from-azure-resource-instances-preview"></a>Предоставление доступа из экземпляров ресурсов Azure (Предварительная версия)

В некоторых случаях приложение может зависеть от ресурсов Azure, которые не могут быть изолированы с помощью виртуальной сети или правила IP-адресов. Тем не менее вы по-прежнему хотите защитить и ограничить доступ к учетной записи хранения только ресурсами Azure вашего приложения. Вы можете настроить учетные записи хранения, чтобы разрешить доступ к определенным экземплярам ресурсов некоторых служб Azure, создав правило экземпляра ресурса. 

Типы операций, которые может выполнять экземпляр ресурса в данных учетной записи хранения, определяются [назначениями ролей Azure](storage-auth-aad.md#assign-azure-roles-for-access-rights) экземпляра ресурса. Экземпляры ресурсов должны находиться в том же клиенте, что и учетная запись хранения, но они могут принадлежать любой подписке в клиенте.

> [!NOTE]
> Эта функция доступна в общедоступной предварительной версии и доступна во всех регионах общедоступного облака.

> [!NOTE]
> Правила экземпляра ресурса в настоящее время поддерживаются только для Azure синапсе. Поддержка других служб Azure, перечисленных в разделе « [доверенный доступ на основе управляемого системой удостоверения](#trusted-access-system-assigned-managed-identity) » этой статьи, будет доступна в ближайшие недели.


### <a name="portal"></a>[Портал](#tab/azure-portal)

Вы можете добавить или удалить правила сети ресурсов в портал Azure.

1. Чтобы начать, войдите на [портал Azure](https://portal.azure.com/).

2. Найдите учетную запись хранения и отобразите общие сведения о ней.

3. Выберите **сеть** , чтобы отобразить страницу конфигурации сети.

4. В раскрывающемся списке **тип ресурса** выберите тип ресурса для экземпляра ресурса. 

5. В раскрывающемся списке **имя экземпляра** выберите экземпляр ресурса. Также можно выбрать включение всех экземпляров ресурсов в активный клиент, подписку или группу ресурсов.

6. Выберите **Сохранить**, чтобы применить изменения. Экземпляр ресурса отображается в разделе " **экземпляры ресурсов** " страницы "Параметры сети". 

Чтобы удалить экземпляр ресурса, щелкните значок удаления ( :::image type="icon" source="media/storage-network-security/delete-icon.png"::: ) рядом с экземпляром ресурса.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для добавления или удаления правил сети ресурсов можно использовать команды PowerShell.

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе сетевые правила не будут действовать.

#### <a name="install-the-preview-module"></a>Установка модуля предварительной версии

Установите последнюю версию модуля PowershellGet. Затем закройте и снова откройте консоль PowerShell.

```powershell
install-Module PowerShellGet –Repository PSGallery –Force  
```

Установите **AZ. Storage,** выполнив предварительную версию модуля.

```powershell
Install-Module Az.Storage -Repository PsGallery -RequiredVersion 3.0.1-preview -AllowClobber -AllowPrerelease -Force 
```

Дополнительные сведения об установке модулей PowerShell см. [в статье Установка модуля Azure PowerShell](/powershell/azure/install-az-ps) .

#### <a name="grant-access"></a>Предоставление доступа

Добавьте сетевое правило, предоставляющее доступ из экземпляра ресурса.

```powershell
$resourceId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory"
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Add-AzStorageAccountNetworkRule -ResourceGroupName $resourceGroupName -Name $accountName -TenantId $tenantId -ResourceId $resourceId

```

Одновременное указание нескольких экземпляров ресурсов путем изменения набора правил сети.

```powershell
$resourceId1 = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory"
$resourceId2 = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/mySQLServer"
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Update-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName -ResourceAccessRule (@{ResourceId=$resourceId1;TenantId=$tenantId},@{ResourceId=$resourceId2;TenantId=$tenantId}) 
```

#### <a name="remove-access"></a>Запрет доступа

Удаление сетевого правила, предоставляющего доступ из экземпляра ресурса.

```powershell
$resourceId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory"
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Remove-AzStorageAccountNetworkRule -ResourceGroupName $resourceGroupName -Name $accountName -TenantId $tenantId -ResourceId $resourceId  
```

Удалите все правила сети, которые предоставляют доступ к экземплярам ресурсов.

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Update-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName -ResourceAccessRule @()  
```

#### <a name="view-a-list-of-allowed-resource-instances"></a>Просмотр списка разрешенных экземпляров ресурсов

Просмотрите полный список экземпляров ресурсов, которым был предоставлен доступ к учетной записи хранения.

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

$rule = Get-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName
$rule.ResourceAccessRules 
```

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для добавления или удаления правил сети ресурсов можно использовать команды Azure CLI.

#### <a name="install-the-preview-extension"></a>Установка расширения предварительной версии

1. Откройте [Azure Cloud Shell](../../cloud-shell/overview.md) или, если вы [установили](/cli/azure/install-azure-cli) Azure CLI локально, командное консольное приложение (например, Windows PowerShell).

2. Затем убедитесь, что установленная версия Azure CLI `2.13.0` или более поздняя, используя следующую команду.

   ```azurecli
   az --version
   ```

   Если ваша версия Azure CLI ниже чем `2.13.0`, установите более позднюю версию. Подробнее см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

3. Введите следующую команду, чтобы установить расширение предварительной версии.

   ```azurecli
   az extension add -n storage-preview
   ```

#### <a name="grant-access"></a>Предоставление доступа

Добавьте сетевое правило, предоставляющее доступ из экземпляра ресурса.

```azurecli
az storage account network-rule add \
    --resource-id /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Synapse/workspaces/testworkspace \
    --tenant-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    -g myResourceGroup \
    --account-name mystorageaccount
```

#### <a name="remove-access"></a>Запрет доступа

Удаление сетевого правила, предоставляющего доступ из экземпляра ресурса.

```azurecli
az storage account network-rule remove \
    --resource-id /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Synapse/workspaces/testworkspace \
    --tenant-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    -g myResourceGroup \
    --account-name mystorageaccount
```

#### <a name="view-a-list-of-allowed-resource-instances"></a>Просмотр списка разрешенных экземпляров ресурсов

Просмотрите полный список экземпляров ресурсов, которым был предоставлен доступ к учетной записи хранения.

```azurecli
az storage account network-rule list \
    -g myResourceGroup \
    --account-name mystorageaccount
```

---

<a id="exceptions"></a>
<a id="trusted-microsoft-services"></a>

## <a name="grant-access-to-trusted-azure-services"></a>Предоставление доступа к доверенным службам Azure 

Некоторые службы Azure работают с сетями, которые не могут быть добавлены в правила сети. Вы можете предоставить подмножеству таких доверенных служб Azure доступ к учетной записи хранения, сохраняя сетевые правила для других приложений. Затем эти доверенные службы будут использовать строгую проверку подлинности для безопасного подключения к учетной записи хранения.

Вы можете предоставить доступ к доверенным службам Azure, создав исключение правила сети. Пошаговые инструкции см. в разделе [Управление исключениями](#manage-exceptions) этой статьи.

При предоставлении доступа к доверенным службам Azure вы предоставляете следующие типы доступа:

- Доверенный доступ для операций SELECT к ресурсам, зарегистрированным в вашей подписке.
- Доверенный доступ к ресурсам на основе управляемого системой удостоверения.

<a id="trusted-access-resources-in-subscription"></a>

### <a name="trusted-access-for-resources-registered-in-your-subscription"></a>Доверенный доступ для ресурсов, зарегистрированных в вашей подписке

Ресурсы некоторых служб **при регистрации в вашей подписке** могут получить доступ к вашей учетной записи хранения **в той же подписке** для выполнения некоторых операций, например для записи в журналы или резервного копирования.  В следующей таблице описаны все службы и разрешенные операции. 

| Служба                  | Имя поставщика ресурсов     | Разрешенные операции                 |
|:------------------------ |:-------------------------- |:---------------------------------- |
| Azure Backup             | Microsoft.RecoveryServices | Резервное копирование и восстановление неуправляемых дисков в виртуальных машинах IAAS. (Не требуется для управляемых дисков.) [Подробнее](../../backup/backup-overview.md). |
| Azure Data Box           | Microsoft.DataBox          | Позволяет импортировать данные в Azure с помощью Data Box. [Подробнее](../../databox/data-box-overview.md). |
| Azure DevTest Labs       | Microsoft.DevTestLab       | Создание пользовательских образов и установка артефактов. [Подробнее](../../devtest-labs/devtest-lab-overview.md). |
| Сетка событий Azure         | Microsoft.EventGrid        | Включение публикации событий в хранилище BLOB-объектов и предоставление службе "Сетка событий" разрешения на публикацию в хранилище очередей. См. дополнительные сведения о [событиях хранилища BLOB-объектов](../../event-grid/overview.md#event-sources) и [публикации в хранилище очередей](../../event-grid/event-handlers.md). |
| Центры событий Azure         | Microsoft.EventHub         | Архивация данных с помощью функции "Сбор" в Центрах событий. [Подробнее](../../event-hubs/event-hubs-capture-overview.md) |
| Служба синхронизации файлов Azure          | Microsoft.StorageSync      | Позволяет преобразовать локальный файловый сервер в кэш для общих папок Azure. Таким образом создается возможность многосайтовой синхронизации, быстрого аварийного восстановления и резервного копирования на стороне облака. [Дополнительные сведения](../files/storage-sync-files-planning.md) |
| Azure HDInsight          | Microsoft.HDInsight        | Подготавливает начальное содержимое файловой системы по умолчанию для нового кластера HDInsight. [Подробнее](../../hdinsight/hdinsight-hadoop-use-blob-storage.md). |
| Импорт и экспорт Azure      | Microsoft.ImportExport     | Позволяет импортировать данные в службу хранилища Azure или экспортировать данные из хранилища Azure с помощью службы импорта и экспорта хранилища Azure. [Подробнее](../../import-export/storage-import-export-service.md).  |
| Azure Monitor            | Microsoft.Insights         | Позволяет записывать в защищенную учетную запись хранения данные мониторинга, в том числе журналы ресурсов, журналы входа и аудита Azure Active Directory, а также журналы Microsoft Intune. [Подробнее](../../azure-monitor/roles-permissions-security.md). |
| Сеть Azure         | Microsoft.Network.          | Хранение и анализ журналов сетевого трафика, в том числе с помощью наблюдателя за сетями и служб Аналитика трафика. [Подробнее](../../network-watcher/network-watcher-nsg-flow-logging-overview.md). |
| Azure Site Recovery      | Microsoft.SiteRecovery     | Включение репликации для аварийного восстановления виртуальных машин IaaS Azure при использовании кэша, источника или целевых учетных записей хранения с поддержкой брандмауэра.  [Подробнее](../../site-recovery/azure-to-azure-tutorial-enable-replication.md). |

<a id="trusted-access-system-assigned-managed-identity"></a>

### <a name="trusted-access-based-on-system-assigned-managed-identity"></a>Доверенный доступ на основе управляемого системой удостоверения

В следующей таблице перечислены службы, которые могут иметь доступ к данным учетной записи хранения, если экземплярам этих служб предоставлено соответствующее разрешение. Чтобы предоставить разрешение, необходимо явно [назначить роль Azure](storage-auth-aad.md#assign-azure-roles-for-access-rights) [управляемому удостоверению, назначенному системой](../../active-directory/managed-identities-azure-resources/overview.md) , для каждого экземпляра ресурса. В таком случае область доступа для этого экземпляра соответствует роли Azure, назначенной управляемому удостоверению. 

> [!TIP]
> Рекомендуемый способ предоставления доступа к конкретным ресурсам — использование правил экземпляра ресурса. Сведения о предоставлении доступа к конкретным экземплярам ресурсов см. в разделе [предоставление доступа из экземпляров ресурсов Azure (Предварительная версия)](#grant-access-specific-instances) этой статьи.


| Служба                        | Имя поставщика ресурсов                 | Назначение            |
| :----------------------------- | :------------------------------------- | :----------------- |
| Служба "Управление API Azure"           | Microsoft.ApiManagement/service        | Включает доступ службы управления API к учетным записям хранения за брандмауэром с помощью политик. [Подробнее](../../api-management/api-management-authentication-policies.md#use-managed-identity-in-send-request-policy). |
| Когнитивный поиск Azure         | Microsoft.Search/searchServices        | Разрешает службам Когнитивного поиска доступ к учетным записям хранения для индексирования, обработки и выполнения запросов. |
| Azure Cognitive Services       | Microsoft. Когнитивесервице/учетные записи    | Разрешает Cognitive Services доступ к учетным записям хранения. |
| Задачи Реестра контейнеров Azure | Microsoft.ContainerRegistry/registries | Задачи Реестра контейнеров Azure могут получать доступ к учетным записям хранения при создании образов контейнеров. |
| Фабрика данных Azure             | Microsoft.DataFactory/factories;        | Разрешает доступ к учетным записям хранения через среду ADF. |
| Azure Data Share               | Microsoft.DataShare/accounts           | Разрешает доступ к учетным записям хранения через Data Share. |
| Azure DevTest Labs             | Microsoft.DevTestLab/labs              | Разрешает доступ к учетным записям хранения через DevTest Labs. |
| Центр Интернета вещей Azure                  | Microsoft.Devices/IotHubs              | Позволяет записывать данные из центра Интернета вещей в хранилище BLOB-объектов. [Дополнительные сведения](../../iot-hub/virtual-network-support.md#egress-connectivity-to-storage-account-endpoints-for-routing) |
| Azure Logic Apps               | Microsoft.Logic/workflows              | Позволяет приложениям логики получать доступ к учетным записям хранения. [Подробнее](../../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity). |
| Служба "Машинное обучение Azure" | Microsoft.MachineLearningServices      | Авторизованные рабочие области Машинного обучения Azure записывают выходные данные эксперимента, модели и журналы в хранилище BLOB-объектов и читают данные. [Подробнее](../../machine-learning/how-to-network-security-overview.md#secure-the-workspace-and-associated-resources). |
| Службы мультимедиа Azure           | Microsoft.Media/mediaservices          | Разрешает доступ к учетным записям хранения через службы мультимедиа. |
| Служба "Миграция Azure"                  | Microsoft. Migrate/мигратепрожектс      | Разрешает доступ к учетным записям хранения с помощью службы "миграция Azure". |
| Azure Purview                  | Microsoft. зрения/учетные записи             | Разрешает зрения доступ к учетным записям хранения. |
| Удаленная отрисовка Azure         | Microsoft. Микседреалити/Ремотерендерингаккаунтс | Разрешает доступ к учетным записям хранения через удаленную визуализацию. |
| Azure Site Recovery            | Microsoft.RecoveryServices/vaults      | Разрешает доступ к учетным записям хранения с помощью Site Recovery. |
| База данных SQL Azure             | Microsoft.Sql                          | Позволяет [записывать](../../azure-sql/database/audit-write-storage-account-behind-vnet-firewall.md) данные аудита в учетные записи хранения за брандмауэром. |
| Azure Synapse Analytics        | Microsoft.Sql                          | Позволяет импортировать и экспортировать данные из конкретных баз данных SQL с помощью инструкции COPY или Polybase (в выделенном пуле), а также `openrowset` функции и внешних таблиц в бессерверном пуле. [Подробнее](../../azure-sql/database/vnet-service-endpoint-rule-overview.md). |
| Azure Stream Analytics         | Microsoft.StreamAnalytics              | Позволяет записывать данные из задания потоковой передачи в хранилище BLOB-объектов. [Подробнее](../../stream-analytics/blob-output-managed-identity.md). |
| Azure Synapse Analytics        | Microsoft.Synapse/workspaces           | Обеспечивает доступ к данным в службе хранилища Azure из Azure синапсе Analytics. |

## <a name="grant-access-to-storage-analytics"></a>Предоставление доступа к аналитике хранилища

В некоторых случаях для чтения метрик и журналов ресурсов требуется доступ из-за пределов границ сети. При настройке доверенных служб для доступа к учетной записи хранения можно разрешить доступ для чтения к файлам журнала, таблицам метрик или обоим параметрам, создав исключение правила сети. Пошаговые инструкции см. в разделе **Управление исключениями** ниже. Дополнительные сведения о работе с аналитикой хранилища см. в статье [использование аналитики службы хранилища Azure для сбора данных журналов и метрик](./storage-analytics.md). 

<a id="manage-exceptions"></a>

## <a name="manage-exceptions"></a>управление исключениями.

Исключениями из правил сети можно управлять с помощью портала Azure, PowerShell или Azure CLI v2.

#### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к учетной записи хранения, которую нужно защитить.

2. Выберите в меню Параметры пункт **Сетевые подключения**.

3. Убедитесь, что вы разрешили доступ в разделе **Выбранные сети**.

4. В разделе **Исключения** выберите те исключения, которые нужно предоставить.

5. Выберите **Сохранить**, чтобы применить изменения.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Установите [Azure PowerShell](/powershell/azure/install-Az-ps) и [выполните вход](/powershell/azure/authenticate-azureps).

2. Отобразите исключения для сетевых правил учетной записи хранения.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount").Bypass
    ```

3. Настройте исключения для сетевых правил учетной записи хранения.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass AzureServices,Metrics,Logging
    ```

4. Удалите исключения для сетевых правил учетной записи хранения.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass None
    ```

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе исключение не будет действовать.

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Установите [Azure CLI](/cli/azure/install-azure-cli) и [выполните вход](/cli/azure/authenticate-azure-cli).

2. Отобразите исключения для сетевых правил учетной записи хранения.

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.bypass
    ```

3. Настройте исключения для сетевых правил учетной записи хранения.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass Logging Metrics AzureServices
    ```

4. Удалите исключения для сетевых правил учетной записи хранения.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass None
    ```

> [!IMPORTANT]
> Обязательно [укажите для правила по умолчанию](#change-the-default-network-access-rule) значение **deny**, иначе исключение не будет действовать.

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о конечных точках службы сети Azure см. в статье [Конечные точки службы виртуальной сети](../../virtual-network/virtual-network-service-endpoints-overview.md).

Более подробную информацию о безопасности службы хранилища Azure см. в статье [Руководство по безопасности службы хранилища Azure](../blobs/security-recommendations.md).