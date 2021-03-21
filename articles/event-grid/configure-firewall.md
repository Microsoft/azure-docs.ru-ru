---
title: Настройка брандмауэра IP для разделов или доменов службы "Сетка событий Azure"
description: В этой статье описывается, как настроить параметры брандмауэра для разделов и доменов сетки событий.
ms.topic: conceptual
ms.date: 03/02/2021
ms.openlocfilehash: 178b9d84ea8b2e0f764f7584526db8dbcf5284f3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031843"
---
# <a name="configure-ip-firewall-for-azure-event-grid-topics-or-domains"></a>Настройка брандмауэра IP для разделов или доменов службы "Сетка событий Azure" 
По умолчанию раздел и домен доступны через Интернет, если запрос сопровождается действительной проверкой подлинности и авторизацией. С помощью брандмауэра для IP-адресов такой доступ можно дополнительно ограничить набором или диапазоном IPv4-адресов в нотации [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). Издатели, исходящие из любого другого IP-адреса, будут отклонены и получат ответ 403 (запрещено). Дополнительные сведения о функциях сетевой безопасности, поддерживаемых службой "Сетка событий", см. в разделе [Сетевая безопасность для сетки событий](network-security.md).

В этой статье описывается, как настроить параметры брандмауэра IP для разделов и доменов службы "Сетка событий Azure".

## <a name="use-azure-portal"></a>Использование портала Azure
В этом разделе показано, как использовать портал Azure для создания правил брандмауэра для входящих IP-адресов. Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания правил IP для входящего трафика для **доменов**. 

1. В [портал Azure](https://portal.azure.com)перейдите к разделу или домену сетки событий и перейдите на вкладку **Сетевые подключения** .
2. Выберите **общедоступные сети** , чтобы разрешить всем сетям, включая Интернет, доступ к ресурсу. 

    Трафик можно ограничить с помощью правил брандмауэра на основе IP-адресов. Укажите один адрес IPv4 или диапазон IP-адресов в нотации, зависящей от класса междоменной маршрутизации (CIDR). 

    ![Снимок экрана, на котором отображается страница "доступ к общедоступной сети" с выбранным параметром "общедоступные сети".](./media/configure-firewall/public-networks-page.png)
3. Выберите **частные конечные точки только** , чтобы разрешить доступ к этому ресурсу только частным конечным точкам. Используйте вкладку **подключения к частной конечной точке** на этой странице для управления подключениями. 

    ![Страница общедоступных сетей](./media/configure-firewall/private-endpoints-page.png)
4. На панели инструментов щелкните **Сохранить**. 



## <a name="use-azure-cli"></a>Использование Azure CLI
В этом разделе показано, как использовать команды Azure CLI для создания разделов с правилами входящих IP-адресов. Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания правил IP для входящего трафика для **доменов**. 


### <a name="enable-or-disable-public-network-access"></a>Включение или отключение доступа к общедоступной сети
По умолчанию доступ к общедоступной сети включен для разделов и доменов. Его также можно включить явным образом или отключить. Трафик можно ограничить, настроив правила брандмауэра для входящих IP-адресов. 

#### <a name="enable-public-network-access-while-creating-a-topic"></a>Включить доступ к общедоступной сети при создании раздела

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access enabled
```


#### <a name="disable-public-network-access-while-creating-a-topic"></a>Отключение доступа к общедоступной сети при создании раздела

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access disabled
```

> [!NOTE]
> Если для раздела или домена отключен доступ к общедоступной сети, трафик через общедоступный Интернет не разрешается. Доступ к этим ресурсам будет разрешен только для подключений к частным конечным точкам. 


#### <a name="enable-public-network-access-for-an-existing-topic"></a>Включить доступ к общедоступной сети для существующего раздела

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled 
```

#### <a name="disable-public-network-access-for-an-existing-topic"></a>Отключить доступ к общедоступной сети для существующего раздела 

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access disabled
```

### <a name="create-a-topic-with-single-inbound-ip-rule"></a>Создание раздела с единым правилом IP для входящего трафика
Следующий пример команды CLI создает раздел сетки событий с правилами входящего трафика IP. 

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR or CIDR MASK> allow 
```

### <a name="create-a-topic-with-multiple-inbound-ip-rules"></a>Создание раздела с несколькими правилами входящего трафика IP

Приведенный ниже пример команды CLI создает сетку событий два правила IP-адреса для входящего трафика за один шаг: 

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR 1 or CIDR MASK 1> allow \
    --inbound-ip-rules <IP ADDR 2 or CIDR MASK 2> allow
```

### <a name="update-an-existing-topic-to-add-inbound-ip-rules"></a>Обновление существующего раздела для добавления правил IP-адресов входящего трафика
В этом примере сначала создается таблица "Сетка событий", а затем в отдельной команде добавляются правила IP-адресов для входящего трафика для раздела. Он также обновляет правила IP-адресов входящего трафика, которые были заданы во второй команде. 

```azurecli-interactive

# create the event grid topic first
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location

# add inbound IP rules to an existing topic
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR or CIDR MASK> allow

# later, update topic with additional ip rules
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR 1 or CIDR MASK 1> allow \
    --inbound-ip-rules <IP ADDR 2 or CIDR MASK 2> allow
```

### <a name="remove-an-inbound-ip-rule"></a>Удаление правила IP-адреса для входящего трафика
Следующая команда удаляет второе правило, созданное на предыдущем шаге, указывая только первое правило при обновлении параметра. 

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR 1 or CIDR MASK 1> allow
```


## <a name="use-powershell"></a>Использование PowerShell
В этом разделе показано, как использовать команды Azure PowerShell для создания разделов сетки событий Azure с правилами брандмауэра для входящих IP-адресов. Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания правил IP для входящего трафика для **доменов**. 

### <a name="prerequisites"></a>Предварительные условия
Следуйте инструкциям из [руководства. Использование портала для создания приложения Azure AD и субъекта-службы, которые могут получать доступ к ресурсам](../active-directory/develop/howto-create-service-principal-portal.md) для создания Azure Active Directory приложения и заметок следующих значений:

- Идентификатор каталога (клиента)
- Идентификатор приложения (клиента)
- Секрет приложения (клиента)

### <a name="prepare-token-and-headers-for-rest-api-calls"></a>Подготовка токена и заголовков для вызовов REST API 
Выполните следующие предварительные команды, чтобы получить маркер проверки подлинности для использования с REST API вызовами, а также авторизацией и другими сведениями о заголовке. 

```azurepowershell-interactive
# replace <CLIENT ID> and <CLIENT SECRET>
$body = "grant_type=client_credentials&client_id=<CLIENT ID>&client_secret=<CLIENT SECRET>&resource=https://management.core.windows.net"

# get the authentication token. Replace <TENANT ID>
$Token = Invoke-RestMethod -Method Post `
    -Uri https://login.microsoftonline.com/<TENANT ID>/oauth2/token `
    -Body $body `
    -ContentType 'application/x-www-form-urlencoded'

# set authorization and content-type headers
$Headers = @{}
$Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
$Headers.Add("Content-Type","application/json")
```

### <a name="enable-public-network-access-for-an-existing-topic"></a>Включить доступ к общедоступной сети для существующего раздела
По умолчанию доступ к общедоступной сети включен для разделов и доменов. Трафик можно ограничить, настроив правила брандмауэра для входящих IP-адресов. 

```azurepowershell-interactive
$body = @{"properties"=@{"publicNetworkAccess"="enabled"}} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Method 'Patch' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body `
    | ConvertTo-Json -Depth 5
```

### <a name="disable-public-network-access-for-an-existing-topic"></a>Отключить доступ к общедоступной сети для существующего раздела
Если для раздела или домена отключен доступ к общедоступной сети, трафик через общедоступный Интернет не разрешается. Доступ к этим ресурсам будет разрешен только для подключений к частным конечным точкам. 

```azurepowershell-interactive
$body = @{"properties"=@{"publicNetworkAccess"="disabled"}} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Method 'Patch' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body `
    | ConvertTo-Json -Depth 5
```

### <a name="create-an-event-grid-topic-with-inbound-rules-in-one-step"></a>Создание раздела сетки событий с правилами входящего трафика за один шаг

```azurepowershell-interactive

# prepare the body for the REST PUT method. Notice that inbound IP rules are included. 
$body = @{"location"="<LOCATION>"; "sku"= @{"name"="basic"}; "properties"=@{"publicNetworkAccess"="enabled"; "inboundIpRules"=@(@{"ipmask"="<IP ADDR or CIDR MASK>";"action"="allow"})}} | ConvertTo-Json -Depth 5

# create the event grid topic with inbound IP rules
Invoke-RestMethod -Method 'Put' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body

# verify that the topic was created
Invoke-RestMethod -Method 'Get' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    | ConvertTo-Json -Depth 5
```


### <a name="create-event-grid-topic-first-and-then-add-inbound-ip-rules"></a>Сначала создайте сетку событий, а затем добавьте правила IP-адресов для входящего трафика.

```azurepowershell-interactive

# prepare the body for the REST PUT method. Notice that no inbound IP rules are specified. 
$body = @{"location"="<LOCATION>"; "sku"= @{"name"="basic"}; "properties"=@{"publicNetworkAccess"="enabled";}} | ConvertTo-Json -Depth 5

# create the Event Grid topic
Invoke-RestMethod -Method 'Put' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body`

# verify that the topic was created
Invoke-RestMethod -Method 'Get' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    | ConvertTo-Json -Depth 5

# prepare the body for REST PUT method. Notice that it includes inbound IP rules now. This feature is available in both basic and premium tiers.
$body = @{"location"="<LOCATION>"; "sku"= @{"name"="basic"}; "properties"=@{"publicNetworkAccess"="enabled"; "inboundIpRules"=@(@{"ipmask"="<IP ADDR or CIDR MASK>";"action"="allow"}, @{"ipmask"="<IP ADDR or CIDR MASK>";"action"="allow"})}} | ConvertTo-Json -Depth 5

# update the topic with inbound IP rules
Invoke-RestMethod -Method 'Put' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body

# verify that the topic was updated
Invoke-RestMethod -Method 'Get' 
    -Uri "https://management.azure.com/subscriptions/<AzURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    | ConvertTo-Json -Depth 5

```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со сведениями о [мониторинге доставки сообщений в службе "Сетка событий"](monitor-event-delivery.md).
* Дополнительные сведения о ключе аутентификации см. в статье [Сетка событий: безопасность и проверка подлинности](security-authentication.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
* Сведения об устранении неполадок с сетевым подключением см. в разделе [Устранение неполадок с сетевым подключением](troubleshoot-network-connectivity.md)
