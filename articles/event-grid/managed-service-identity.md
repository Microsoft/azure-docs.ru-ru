---
title: Доставка событий, управляемое удостоверение службы и закрытая ссылка
description: В этой статье описывается, как включить управляемое удостоверение службы для раздела сетки событий Azure. Используйте его для переадресации событий в поддерживаемые назначения.
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: 76f10b4627dc9578b1e616a868eab03431b59b69
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105625284"
---
# <a name="event-delivery-with-a-managed-identity"></a>Доставка событий с управляемым удостоверением
В этой статье описывается, как использовать [управляемое удостоверение службы](../active-directory/managed-identities-azure-resources/overview.md) для системного раздела службы "Сетка событий Azure", пользовательского раздела или домена. Используйте его для переадресации событий в поддерживаемые назначения, такие как очереди и разделы служебной шины, центры событий и учетные записи службы хранилища.



## <a name="prerequisites"></a>Предварительные условия
1. Назначение удостоверения, назначенного системой, в системный раздел, пользовательский раздел или домен. 
    - Сведения о пользовательских разделах и доменах см. в разделе [Включение управляемого удостоверения для пользовательских разделов и доменов](enable-identity-custom-topics-domains.md). 
    - Системные разделы см. в разделе [Включение управляемого удостоверения для системных разделов](enable-identity-system-topics.md) .
1. Добавьте удостоверение в соответствующую роль (например, отправитель данных служебной шины) в целевом расположении (например, в очередь служебной шины). Подробные инструкции см. [в статье Добавление удостоверения в роли Azure в местах назначения](add-identity-roles.md) .

    > [!NOTE]
    > В настоящее время невозможно доставить события с помощью [частных конечных точек](../private-link/private-endpoint-overview.md). Дополнительные сведения см. в разделе [частные конечные точки](#private-endpoints) в конце этой статьи. 

## <a name="create-event-subscriptions-that-use-an-identity"></a>Создание подписок на события, использующих удостоверение
После создания пользовательского раздела или системного раздела или домена в службе "Сетка событий" с удостоверением, управляемым системой, и добавления удостоверения к соответствующей роли в назначении, можно приступать к созданию подписок, использующих удостоверение. 

### <a name="use-the-azure-portal"></a>Использование портала Azure
При создании подписки на события вы видите параметр, позволяющий использовать назначенное системой удостоверение для конечной точки в разделе **сведения о конечной точке** . 

![Включение удостоверения при создании подписки на события для очереди служебной шины](./media/managed-service-identity/service-bus-queue-subscription-identity.png)

Можно также включить использование назначенного системой удостоверения, которое будет использоваться для недоставленных сообщений на вкладке **Дополнительные возможности** . 

![Использование назначаемого системой удостоверения для недоставленных сообщений](./media/managed-service-identity/enable-deadletter-identity.png)

### <a name="use-the-azure-cli---service-bus-queue"></a>Использование очереди служебной шины Azure CLI 
В этом разделе вы узнаете, как использовать Azure CLI, чтобы разрешить использование назначенного системой удостоверения для доставки событий в очередь служебной шины. Удостоверение должно быть членом роли **Отправитель данных служебной шины Azure**. Оно также должно быть членом роли **Участник данных BLOB-объектов хранилища** в учетной записи службы хранилища, используемой для недоставленных сообщений. 

#### <a name="define-variables"></a>Определение переменных
Сначала укажите значения для следующих переменных, которые будут использоваться в команде CLI. 

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID CUSTOM TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

# get the service bus queue resource id
queueid=$(az servicebus queue show --namespace-name <SERVICE BUS NAMESPACE NAME> --name <QUEUE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
sb_esname = "<Specify a name for the event subscription>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Создание подписки на события с помощью управляемого удостоверения для доставки 
Этот пример команды создает подписку на событие для пользовательского раздела сетки событий с типом конечной точки, установленным в **очередь служебной шины**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sb_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery-and-dead-lettering"></a>Создание подписки на события с помощью управляемого удостоверения для доставки и недоставленных сообщений
Этот пример команды создает подписку на событие для пользовательского раздела сетки событий с типом конечной точки, установленным в **очередь служебной шины**. Он также указывает, что управляемое системой удостоверение должно использоваться для недоставленных сообщений. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sb_esnameq 
```

### <a name="use-the-azure-cli---event-hubs"></a>Использование концентраторов событий Azure CLI 
В этом разделе вы узнаете, как использовать Azure CLI, чтобы разрешить использование назначенного системой удостоверения для доставки событий в концентратор событий. Удостоверение должно быть членом роли **Отправитель данных центров событий Azure**. Оно также должно быть членом роли **Участник данных BLOB-объектов хранилища** в учетной записи службы хранилища, используемой для недоставленных сообщений. 

#### <a name="define-variables"></a>Определение переменных
```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID CUSTOM TOPIC>"
topicname = "<EVENT GRID CUSTOM TOPIC NAME>"

hubid=$(az eventhubs eventhub show --name <EVENT HUB NAME> --namespace-name <NAMESPACE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
eh_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Создание подписки на события с помощью управляемого удостоверения для доставки 
Этот пример команды создает подписку на событие для пользовательского раздела сетки событий с типом конечной точки, установленным в " **концентраторы событий**". 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type eventhub 
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    -n $sbq_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Создание подписки на события с помощью управляемого удостоверения для доставки и отправки недоставленных сообщений 
Этот пример команды создает подписку на событие для пользовательского раздела сетки событий с типом конечной точки, установленным в " **концентраторы событий**". Он также указывает, что управляемое системой удостоверение должно использоваться для недоставленных сообщений. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    --deadletter-identity-endpoint $eh_deadletterendpoint
    --deadletter-identity systemassigned 
    -n $eh_esname 
```

### <a name="use-the-azure-cli---azure-storage-queue"></a>Использование Azure CLI в очереди службы хранилища Azure 
В этом разделе вы узнаете, как использовать Azure CLI, чтобы разрешить использование назначенного системой удостоверения для доставки событий в очередь службы хранилища Azure. Удостоверение должно быть членом роли **отправителя сообщений очереди хранилища** в учетной записи хранения. Оно также должно быть членом роли **Участник данных BLOB-объектов хранилища** в учетной записи службы хранилища, используемой для недоставленных сообщений.

#### <a name="define-variables"></a>Определение переменных  

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID CUSTOM TOPIC>"
topicname = "<EVENT GRID CUSTOM TOPIC NAME>"

# get the storage account resource id
storageid=$(az storage account show --name <STORAGE ACCOUNT NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)

# build the resource id for the queue
queueid="$storageid/queueservices/default/queues/<QUEUE NAME>" 

sa_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Создание подписки на события с помощью управляемого удостоверения для доставки 

```azurecli-interactive
az eventgrid event-subscription create 
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sa_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Создание подписки на события с помощью управляемого удостоверения для доставки и отправки недоставленных сообщений 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sa_esname 
```

## <a name="private-endpoints"></a>Частные конечные точки
В настоящее время невозможно доставить события с помощью [частных конечных точек](../private-link/private-endpoint-overview.md). То есть нет никакой поддержки, если есть требования к сетевой изоляции, в которых трафик доставленных событий не должен выходить за пределы частного IP-адреса. 

Однако если ваши требования вызывают безопасный способ отправки событий с помощью зашифрованного канала и известного удостоверения отправителя (в этом случае — службы "Сетка событий") с использованием пространства общедоступного IP-адреса, можно передать события в концентраторы событий, служебную шину или службу хранилища Azure с помощью пользовательского раздела службы "Сетка событий Azure" или домена с удостоверением, управляемым системой, как показано в этой Затем можно использовать закрытую ссылку, настроенную в функциях Azure, или веб-перехватчик, развернутый в виртуальной сети, для извлечения событий. См. Пример: [Подключение к частным конечным точкам с помощью функций Azure](/samples/azure-samples/azure-functions-private-endpoints/connect-to-private-endpoints-with-azure-functions/).

В этой конфигурации трафик передается через общедоступный IP-адрес или Интернет из службы "Сетка событий" в концентраторы событий, служебную шину или службу хранилища Azure, но канал может быть зашифрован и используется управляемое удостоверение службы "Сетка событий". Если вы настраиваете функции Azure или веб-перехватчик, развернутые в виртуальной сети, для использования концентраторов событий, служебной шины или хранилища Azure с помощью частной связи, этот раздел трафика будет оставаться в пределах Azure.


## <a name="next-steps"></a>Дальнейшие шаги
Дополнительные сведения об управляемых удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).
