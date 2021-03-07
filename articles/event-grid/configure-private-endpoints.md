---
title: Настройка частных конечных точек для разделов и доменов службы "Сетка событий Azure"
description: В этой статье описывается, как настроить частные конечные точки для разделов или доменов службы "Сетка событий Azure".
ms.topic: how-to
ms.date: 11/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: f8a9ac46596b1c2611ce9df387ac995e8149e7b9
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102425351"
---
# <a name="configure-private-endpoints-for-azure-event-grid-topics-or-domains"></a>Настройка частных конечных точек для разделов и доменов службы "Сетка событий Azure"
[Частные конечные точки](../private-link/private-endpoint-overview.md) можно использовать, чтобы разрешить прием событий непосредственно из виртуальной сети в разделы и домены, защищенные по [частной ссылке](../private-link/private-link-overview.md) , без использования общедоступного Интернета. Частная конечная точка использует IP-адрес из адресного пространства виртуальной сети для вашего раздела или домена. Дополнительные сведения см. в разделе [Сетевая безопасность](network-security.md).

В этой статье описывается, как настроить частные конечные точки для разделов или доменов.

## <a name="use-azure-portal"></a>Использование портала Azure 
В этом разделе показано, как с помощью портал Azure создать закрытую конечную точку для раздела или домена.

> [!NOTE]
> Действия, приведенные в этом разделе, в основном предназначены для разделов. Аналогичные действия можно использовать для создания частных конечных точек для **доменов**. 

1. Войдите в [портал Azure](https://portal.azure.com) и перейдите к разделу или домену.
2. Перейдите на вкладку " **сеть** " на странице с разделом. Выберите **+ Частная конечная точка** на панели инструментов.

    ![Добавить закрытую конечную точку](./media/configure-private-endpoints/add-button.png)
2. На странице **Базовые** выполните следующие действия. 
    1. Выберите **подписку Azure** , в которой вы хотите создать закрытую конечную точку. 
    2. Выберите **группу ресурсов Azure** для закрытой конечной точки. 
    3. Введите **имя** конечной точки. 
    4. Выберите **регион** для конечной точки. Частная конечная точка должна находиться в том же регионе, что и виртуальная сеть, но может находиться в другом регионе, отличном от ресурса частной ссылки (в этом примере — в разделе "Сетка событий"). 
    5. Затем нажмите кнопку **Далее: ресурс >** в нижней части страницы. 

      ![Частная конечная точка — страница "Основные сведения"](./media/configure-private-endpoints/basics-page.png)
3. На странице **Ресурс** выполните следующие действия. 
    1. Если **в моем каталоге выбран вариант подключиться к ресурсу Azure**, то для метода подключения выполните следующие действия. В этом примере показано, как подключиться к ресурсу Azure в вашем каталоге. 
        1. Выберите **подписку Azure** , в которой находится **раздел или домен** . 
        1. Для типа **ресурса** выберите **Microsoft. EventGrid/Topics** или **Microsoft. EventGrid/Domains** для **типа ресурса**.
        2. В поле **ресурс** выберите раздел или домен из раскрывающегося списка. 
        3. Убедитесь, что **целевой подресурс** имеет значение **раздел** или **домен** (в зависимости от выбранного типа ресурса).    
        4. По завершении выберите **Next: Конфигурация >**  в нижней части страницы. 

            ![Снимок экрана, на котором показана страница "Создание частной конечной точки — ресурс".](./media/configure-private-endpoints/resource-page.png)
    2. При выборе **подключения к ресурсу с помощью идентификатора ресурса или псевдонима** выполните следующие действия.
        1. Введите идентификатор ресурса. Например: `/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>`.  
        2. Для **ресурса** введите **раздел** или **домен**. 
        3. используемых Добавление сообщения запроса. 
        4. По завершении выберите **Next: Конфигурация >**  в нижней части страницы. 

            ![Частная конечная точка — страница ресурсов](./media/configure-private-endpoints/connect-azure-resource-id.png)
4. На странице **Конфигурация** выберите подсеть в виртуальной сети, в которой требуется развернуть частную конечную точку. 
    1. Выберите **виртуальную сеть**. В раскрывающемся списке отображаются только виртуальные сети в выбранных сейчас подписке и расположении. 
    2. Выберите **подсеть** в выбранной виртуальной сети. 
    3. По завершении выберите **Next: Теги >** в нижней части страницы. 

    ![Частная конечная точка — страница конфигурации](./media/configure-private-endpoints/configuration-page.png)
5. На странице **Теги** создайте теги (имена и значения), которые нужно связать с ресурсом частной конечной точки. В нижней части страницы нажмите на кнопку **Просмотреть и создать**. 
6. На странице **Проверка и создание** проверьте все параметры и нажмите **Создать**, чтобы создать частную конечную точку. 

    ![Частная конечная точка — Просмотр & создание страницы](./media/configure-private-endpoints/review-create-page.png)
    

### <a name="manage-private-link-connection"></a>Управление подключением Приватного канала

При создании частной конечной точки подключение должно быть утверждено. Если ресурс, для которого создается частная конечная точка, находится в вашем каталоге, вы можете утвердить запрос на подключение, если у вас есть необходимые разрешения. При подключении к ресурсу Azure в другом каталоге необходимо дождаться, пока владелец этого ресурса утвердит запрос на подключение.

Существует четыре состояния подготовки:

| Действие в службе | Состояние частной конечной точки объекта-получателя службы | Описание |
|--|--|--|
| None | Ожидает | Соединение создается вручную и ожидает утверждения от владельца ресурса частной ссылки. |
| Утверждение | Approved | Подключение утверждено автоматически или вручную и готово к использованию. |
| Reject | Отклонено | Подключение отклонил владелец ресурса Приватного канала. |
| Удалить | Отключено | Подключение удалил владелец ресурса Приватного канала. Частная конечная точка станет информативной и подлежит удалению для очистки. |
 
###  <a name="how-to-manage-a-private-endpoint-connection"></a>Управление подключением к частной конечной точке
В следующих разделах показано, как утвердить или отклонить подключение частной конечной точки. 

1. Войдите на [портал Azure](https://portal.azure.com).
1. В строке поиска введите **разделы сетки событий** или **домены сетки событий**.
1. Выберите **раздел** или **домен** , которым требуется управлять.
1. Перейдите на вкладку **Сеть**.
1. Если ожидаются какие-либо подключения, в состоянии подготовки вы увидите подключение в списке **Ожидание** . 

### <a name="to-approve-a-private-endpoint"></a>Утверждение частной конечной точки
Вы можете утвердить частную конечную точку, которая находится в состоянии ожидания. Для утверждения выполните следующие действия. 

> [!NOTE]
> Действия, приведенные в этом разделе, в основном предназначены для разделов. Аналогичные действия можно использовать для утверждения закрытых конечных точек для **доменов**. 

1. Выберите **закрытую конечную точку** , которую вы хотите одобрить, и на панели инструментов выберите **утвердить** .

    ![Частная конечная точка — состояние ожидания](./media/configure-private-endpoints/pending.png)
1. В диалоговом окне **утверждение соединения** введите комментарий (необязательно) и нажмите **кнопку Да**. 

    ![Частная конечная точка — утверждение](./media/configure-private-endpoints/approve.png)
1. Убедитесь, что конечная точка отображается в состоянии **утверждено**. 

    ![Закрытая конечная точка — утвержденное состояние](./media/configure-private-endpoints/approved-status.png)

### <a name="to-reject-a-private-endpoint"></a>Отклонение частной конечной точки
Вы можете отклонить частную конечную точку, находящиеся в состоянии ожидания или утвержденном состоянии. Чтобы отклонить, выполните следующие действия. 

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для отклонения закрытых конечных точек для **доменов**. 

1. Выберите **закрытую конечную точку** , которую вы хотите отклонить, и на панели инструментов выберите **отклонить** .

    ![Снимок экрана, показывающий, что выбрано "отклонить подключения к частным конечным точкам".](./media/configure-private-endpoints/reject-button.png)
1. В диалоговом окне **отклонить подключение** введите комментарий (необязательно) и нажмите **кнопку Да**. 

    ![Частная конечная точка — отклонение](./media/configure-private-endpoints/reject.png)
1. Убедитесь, что конечная точка отображается в состоянии **отклонено**. 

    ![Частная конечная точка — состояние "Отклонено"](./media/configure-private-endpoints/rejected-status.png)

    > [!NOTE]
    > Вы не можете утвердить частную конечную точку в портал Azure после ее отклонения. 


## <a name="use-azure-cli"></a>Использование Azure CLI
Чтобы создать частную конечную точку, используйте метод [AZ Network Private-Endpoint Create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create) , как показано в следующем примере:

```azurecli-interactive
az network private-endpoint create \
    --resource-group <RESOURECE GROUP NAME> \
    --name <PRIVATE ENDPOINT NAME> \
    --vnet-name <VIRTUAL NETWORK NAME> \
    --subnet <SUBNET NAME> \
    --private-connection-resource-id "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME> \
    --connection-name <PRIVATE LINK SERVICE CONNECTION NAME> \
    --location <LOCATION> \
    --group-ids topic
```

Описание параметров, используемых в примере, см. в документации по команде [AZ Network Private-Endpoint Create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create). В этом примере необходимо отметить несколько моментов: 

- Для `private-connection-resource-id` Укажите идентификатор ресурса **раздела** или **домена**. В предыдущем примере используется тип: Topic.
- для `group-ids` укажите `topic` или `domain` . В предыдущем примере `topic` используется. 

Чтобы удалить закрытую конечную точку, используйте метод [AZ Network частный-Endpoint Delete](/cli/azure/network/private-endpoint?#az-network-private-endpoint-delete) , как показано в следующем примере:

```azurecli-interactive
az network private-endpoint delete --resource-group <RESOURECE GROUP NAME> --name <PRIVATE ENDPOINT NAME>
```

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания частных конечных точек для **доменов**. 



### <a name="prerequisites"></a>Предварительные требования
Обновите расширение службы "Сетка событий Azure" для интерфейса командной строки, выполнив следующую команду: 

```azurecli-interactive
az extension update -n eventgrid
```

Если расширение не установлено, выполните следующую команду, чтобы установить его: 

```azurecli-interactive
az extension add -n eventgrid
```

### <a name="create-a-private-endpoint"></a>Создание частной конечной точки
Чтобы создать частную конечную точку, используйте метод [AZ Network Private-Endpoint Create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create) , как показано в следующем примере:

```azurecli-interactive
az network private-endpoint create \
    --resource-group <RESOURECE GROUP NAME> \
    --name <PRIVATE ENDPOINT NAME> \
    --vnet-name <VIRTUAL NETWORK NAME> \
    --subnet <SUBNET NAME> \
    --private-connection-resource-id "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME> \
    --connection-name <PRIVATE LINK SERVICE CONNECTION NAME> \
    --location <LOCATION> \
    --group-ids topic
```

Описание параметров, используемых в примере, см. в документации по команде [AZ Network Private-Endpoint Create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create). В этом примере необходимо отметить несколько моментов: 

- Для `private-connection-resource-id` Укажите идентификатор ресурса **раздела** или **домена**. В предыдущем примере используется тип: Topic.
- для `group-ids` укажите `topic` или `domain` . В предыдущем примере `topic` используется. 

Чтобы удалить закрытую конечную точку, используйте метод [AZ Network частный-Endpoint Delete](/cli/azure/network/private-endpoint?#az-network-private-endpoint-delete) , как показано в следующем примере:

```azurecli-interactive
az network private-endpoint delete --resource-group <RESOURECE GROUP NAME> --name <PRIVATE ENDPOINT NAME>
```

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания частных конечных точек для **доменов**. 

#### <a name="sample-script"></a>Пример скрипта
Ниже приведен пример сценария, который создает следующие ресурсы Azure:

- Группа ресурсов
- Виртуальная сеть
- Подсеть в виртуальной сети
- Статья о службе "Сетка событий Azure"
- Частная конечная точка для раздела

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания частных конечных точек для доменов.

```azurecli-interactive
subscriptionID="<AZURE SUBSCRIPTION ID>"
resourceGroupName="<RESOURCE GROUP NAME>"
location="<LOCATION>"
vNetName="<VIRTUAL NETWORK NAME>"
subNetName="<SUBNET NAME>"
topicName = "<TOPIC NAME>"
connectionName="<ENDPOINT CONNECTION NAME>"
endpointName=<ENDPOINT NAME>

# resource ID of the topic. replace <SUBSCRIPTION ID>, <RESOURCE GROUP NAME>, and <TOPIC NAME>
topicResourceID="/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME>"

# select subscription
az account set --subscription $subscriptionID

# create resource group
az group create --name $resourceGroupName --location $location

# create vnet 
az network vnet create \
    --resource-group $resourceGroupName \
    --name $vNetName \
    --address-prefix 10.0.0.0/16

# create subnet
az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $vNetName \
    --name $subNetName \
    --address-prefixes 10.0.0.0/24

# disable private endpoint network policies for the subnet
az network vnet subnet update \
    --resource-group $resourceGroupName \
    --vnet-name $vNetName \
    --name $subNetName \
    --disable-private-endpoint-network-policies true

# create event grid topic. update <LOCATION>
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location

# verify that the topic was created.
az eventgrid topic show \
    --resource-group $resourceGroupName \
    --name $topicName

# create private endpoint for the topic you created
az network private-endpoint create 
    --resource-group $resourceGroupName \
    --name $endpointName \
    --vnet-name $vNetName \
    --subnet $subNetName \
    --private-connection-resource-id $topicResourceID \
    --connection-name $connectionName \
    --location $location \
    --group-ids topic

# get topic 
az eventgrid topic show \
    --resource-group $resourceGroupName \
    --name $topicName

```

### <a name="approve-a-private-endpoint"></a>Утверждение частной конечной точки
В следующем примере фрагмента кода CLI показано, как утвердить подключение к частной конечной точке. 

```azurecli-interactive
az eventgrid topic private-endpoint-connection approve \
    --resource-group $resourceGroupName \
    --topic-name $topicName \
    --name  $endpointName \
    --description "connection approved"
```


### <a name="reject-a-private-endpoint"></a>Отклонение частной конечной точки
В следующем примере фрагмента кода CLI показано, как отклонить подключение к частной конечной точке. 

```azurecli-interactive
az eventgrid topic private-endpoint-connection reject \
    --resource-group $resourceGroupName \
    --topic-name $topicName \
    --name $endpointName \
    --description "Connection rejected"
```

### <a name="disable-public-network-access"></a>Отключение доступа из общедоступной сети
По умолчанию доступ к общедоступной сети включен для раздела или домена сетки событий. Чтобы разрешить доступ только через частные конечные точки, отключите доступ к общедоступной сети, выполнив следующую команду:  

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access disabled
```


## <a name="use-powershell"></a>Использование PowerShell
В этом разделе показано, как создать частную конечную точку для раздела или домена с помощью PowerShell. 

### <a name="prerequisite"></a>Предварительное требование
Следуйте инструкциям из [руководства. с помощью портала Создайте приложение Azure AD и субъект-службу, которые могут получить доступ к ресурсам](../active-directory/develop/howto-create-service-principal-portal.md) для создания Azure Active Directory приложения, и запишите значения для **идентификатора каталога (клиента)**, **идентификатора приложения (клиента)** и **секрета приложения (клиента)**. 

### <a name="prepare-token-and-headers-for-rest-api-calls"></a>Подготовка токена и заголовков для вызовов REST API 
Выполните следующие обязательные команды, чтобы получить маркер проверки подлинности для использования с REST APIными вызовами и авторизацией и другими сведениями о заголовке. 

```azurepowershell-interactive
$body = "grant_type=client_credentials&client_id=<CLIENT ID>&client_secret=<CLIENT SECRET>&resource=https://management.core.windows.net"

# get authentication token
$Token = Invoke-RestMethod -Method Post `
    -Uri https://login.microsoftonline.com/<TENANT ID>/oauth2/token  `
    -Body $body  `
    -ContentType 'application/x-www-form-urlencoded' 

# set authorization and content-type headers
$Headers = @{}
$Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
```

### <a name="create-a-subnet-with-endpoint-network-policies-disabled"></a>Создание подсети с отключенными политиками сети конечных точек

```azurepowershell-interactive

# create resource group
New-AzResourceGroup -ResourceGroupName <RESOURCE GROUP NAME>  -Location <LOCATION>

# create virtual network
$virtualNetwork = New-AzVirtualNetwork `
                    -ResourceGroupName <RESOURCE GROUP NAME> `
                    -Location <LOCATION> `
                    -Name <VIRTUAL NETWORK NAME> `
                    -AddressPrefix 10.0.0.0/16

# create subnet with endpoint network policy disabled
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
                    -Name example-privatelinksubnet `
                    -AddressPrefix 10.0.0.0/24 `
                    -PrivateEndpointNetworkPoliciesFlag "Disabled" `
                    -VirtualNetwork $virtualNetwork

# update virtual network
$virtualNetwork | Set-AzVirtualNetwork
```

### <a name="create-an-event-grid-topic-with-a-private-endpoint"></a>Создание раздела сетки событий с частной конечной точкой

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для создания частных конечных точек для **доменов**. 


```azurepowershell-interactive
$body = @{"location"="<LOCATION>"; "properties"=@{"publicNetworkAccess"="disabled"}} | ConvertTo-Json

# create topic
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01"  `
    -Headers $Headers  `
    -Body $body

# verify that the topic was created
$topic=Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01"   `
    -Headers $Headers  

# create private link service connection
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
                                -Name "<PRIVATE LINK SERVICE CONNECTION NAME>" `
                                -PrivateLinkServiceId $topic.Id `
                                -GroupId "topic"

# get virtual network info 
$virtualNetwork = Get-AzVirtualNetwork  `
                    -ResourceGroupName  <RESOURCE GROUP NAME> `
                    -Name <VIRTUAL NETWORK NAME>

# get subnet info
$subnet = $virtualNetwork | Select -ExpandProperty subnets `
                             | Where-Object  {$_.Name -eq <SUBNET NAME>}  

# now, you are ready to create a private endpoint 
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName <RESOURCE GROUP NAME>  `
                                        -Name <PRIVATE ENDPOINT NAME>   `
                                        -Location <LOCATION> `
                                        -Subnet  $subnet   `
                                        -PrivateLinkServiceConnection $privateEndpointConnection

# verify that the endpoint was created
Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections?api-version=2020-06-01"   `
    -Headers $Headers   `
    | ConvertTo-Json -Depth 5

```

При проверке того, что конечная точка была создана, вы увидите результат, аналогичный приведенному ниже:

```json

{
  "value": [
    {
      "properties": {
        "privateEndpoint": {
          "id": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Network/privateEndpoints/<PRIVATE ENDPOINT NAME>"
        },
        "groupIds": [
          "topic"
        ],
        "privateLinkServiceConnectionState": {
          "status": "Approved",
          "description": "Auto-approved",
          "actionsRequired": "None"
        },
        "provisioningState": "Succeeded"
      },
      "id": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>",
      "name": "myConnection",
      "type": "Microsoft.EventGrid/topics/privateEndpointConnections"
    }
  ]
}
```

### <a name="approve-a-private-endpoint-connection"></a>Утверждение подключения частной конечной точки
В следующем примере фрагмента кода PowerShell показано, как утвердить частную конечную точку. 

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для утверждения закрытых конечных точек для **доменов**. 

```azurepowershell-interactive
$approvedBody = @{"properties"=@{"privateLinkServiceConnectionState"=@{"status"="approved";"description"="connection approved";"actionsRequired"="none"}}} | ConvertTo-Json

# approve endpoint connection
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AzuRE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01"  `
    -Headers $Headers  `
    -Body $approvedBody

# confirm that the endpoint connection was approved
Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01"  `
    -Headers $Headers

```

### <a name="reject-a-private-endpoint-connection"></a>Отклонение подключения к частной конечной точке
В следующем примере показано, как отклонить частную конечную точку с помощью PowerShell. Идентификатор GUID для закрытой конечной точки можно получить из результата предыдущей команды GET. 

> [!NOTE]
> Действия, приведенные в этом разделе, предназначены для разделов. Аналогичные действия можно использовать для отклонения закрытых конечных точек для **доменов**. 


```azurepowershell-interactive
$rejectedBody = @{"properties"=@{"privateLinkServiceConnectionState"=@{"status"="rejected";"description"="connection rejected";"actionsRequired"="none"}}} | ConvertTo-Json

# reject private endpoint
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01"  `
    -Headers $Headers  `
    -Body $rejectedBody

# confirm that endpoint was rejected
Invoke-RestMethod -Method 'Get' 
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01" ` 
    -Headers $Headers
```

Вы можете одобрить подключение даже после его отклонения через API. При использовании портал Azure нельзя утвердить конечную точку, которая была отклонена. 

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о настройке параметров брандмауэра IP см. в разделе [Настройка брандмауэра IP для разделов и доменов службы "Сетка событий Azure](configure-firewall.md)".
* Сведения об устранении неполадок с сетевым подключением см. в разделе [Устранение неполадок с сетевым подключением](troubleshoot-network-connectivity.md)
