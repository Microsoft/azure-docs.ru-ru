---
title: Доставка событий, управляемое удостоверение службы и закрытая ссылка
description: В этой статье описывается, как включить управляемое удостоверение службы для раздела сетки событий Azure. Используйте его для переадресации событий в поддерживаемые назначения.
ms.topic: how-to
ms.date: 01/28/2021
ms.openlocfilehash: 3e643465db7cc918499ca962c4697cb61cb4b594
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100007777"
---
# <a name="event-delivery-with-a-managed-identity"></a>Доставка событий с управляемым удостоверением
В этой статье описывается, как включить [управляемое удостоверение службы](../active-directory/managed-identities-azure-resources/overview.md) для пользовательских разделов или доменов сетки событий Azure. Используйте его для переадресации событий в поддерживаемые назначения, такие как очереди и разделы служебной шины, центры событий и учетные записи службы хранилища.

Ниже описаны действия, которые подробно описаны в этой статье:
1. Создайте пользовательский раздел или домен с удостоверением, назначенным системой, или обновите существующий пользовательский раздел или домен, чтобы включить удостоверение. 
1. Добавьте удостоверение в соответствующую роль (например, отправитель данных служебной шины) в целевом расположении (например, в очередь служебной шины).
1. При создании подписок на события включите использование удостоверения для доставки событий в назначение. 

> [!NOTE]
> В настоящее время невозможно доставить события с помощью [частных конечных точек](../private-link/private-endpoint-overview.md). Дополнительные сведения см. в разделе [частные конечные точки](#private-endpoints) в конце этой статьи. 

## <a name="create-a-custom-topic-or-domain-with-an-identity"></a>Создание пользовательского раздела или домена с удостоверением
Для начала рассмотрим, как создается раздел или домен с удостоверением, управляемым системой.

### <a name="use-the-azure-portal"></a>Использование портала Azure
Вы можете включить назначенное системой удостоверение для пользовательского раздела или домена при его создании в портал Azure. На следующем рисунке показано, как включить управляемое системой удостоверение для пользовательского раздела. Фактически можно выбрать параметр **Включить назначенное системой удостоверение** на странице **Дополнительно** мастера создания разделов. Этот параметр также отображается на странице **Дополнительно** мастера создания домена. 

![Включить удостоверение при создании пользовательского раздела](./media/managed-service-identity/create-topic-identity.png)

### <a name="use-the-azure-cli"></a>Использование Azure CLI
Можно также использовать Azure CLI для создания пользовательского раздела или домена с удостоверением, назначенным системой. Используйте команду `az eventgrid topic create` с параметром `--identity`, для которого задано значение `systemassigned`. Если не указать значение для этого параметра, используется значение по умолчанию `noidentity`. 

```azurecli-interactive
# create a custom topic with a system-assigned identity
az eventgrid topic create -g <RESOURCE GROUP NAME> --name <TOPIC NAME> -l <LOCATION>  --identity systemassigned
```

Аналогичным образом можно использовать команду `az eventgrid domain create`, чтобы создать домен с удостоверением, управляемым системой.

## <a name="enable-an-identity-for-an-existing-custom-topic-or-domain"></a>Включение удостоверения для существующего пользовательского раздела или домена
В предыдущем разделе вы узнали, как включить управляемое системой удостоверение при создании пользовательского раздела или домена. В этом разделе вы узнаете, как включить управляемое системой удостоверение для существующего пользовательского раздела или домена. 

### <a name="use-the-azure-portal"></a>Использование портала Azure
В следующей процедуре показано, как включить управляемое системой удостоверение для пользовательского раздела. Действия по включению удостоверения для домена похожи. 

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com).
2. Поиск **разделов сетки событий** на панели поиска вверху.
3. Выберите **пользовательский раздел** , для которого необходимо включить управляемое удостоверение. 
4. Перейдите на вкладку **Удостоверение**. 
5. Включите **параметр, чтобы** включить удостоверение. 
1. Нажмите кнопку **сохранить** на панели инструментов, чтобы сохранить параметр. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-topic.png" alt-text="Страница удостоверений для пользовательского раздела"::: 

Аналогичные действия можно использовать для включения удостоверения для домена сетки событий.

### <a name="use-the-azure-cli"></a>Использование Azure CLI
Используйте команду со значением, равным, `az eventgrid topic update` `--identity` `systemassigned` чтобы включить назначенное системой удостоверение для существующего пользовательского раздела. Чтобы отключить удостоверение, укажите значение `noidentity`. 

```azurecli-interactive
# Update the topic to assign a system-assigned identity. 
az eventgrid topic update -g $rg --name $topicname --identity systemassigned --sku basic 
```

Для обновления существующего домена используется аналогичная команда (`az eventgrid domain update`).

## <a name="supported-destinations-and-azure-roles"></a>Поддерживаемые назначения и роли Azure
После включения удостоверения для пользовательского раздела или домена сетки событий Azure автоматически создает удостоверение в Azure Active Directory. Добавьте это удостоверение в соответствующие роли Azure, чтобы пользовательский раздел или домен мог пересылать события в поддерживаемые места назначения. Например, добавьте удостоверение в роль **отправителя данных концентраторов событий Azure** для пространства имен концентраторов событий Azure, чтобы Пользовательская тема сетки событий могла пересылать события в концентраторы событий в этом пространстве имен. 

В настоящее время служба "Сетка событий Azure" поддерживает настраиваемые разделы или домены, для которых назначено управляемое системой удостоверение для перенаправления событий в следующие места назначения. Эта таблица также предоставляет роли, в которых должно быть удостоверение, чтобы Пользовательская тема могла переслать события.

| Назначение | Роль Azure | 
| ----------- | --------- | 
| Очереди и разделы служебной шины Azure | [Отправитель данных служебной шины Azure](../service-bus-messaging/authenticate-application.md#azure-built-in-roles-for-azure-service-bus) |
| Центры событий Azure | [Отправитель данных центров событий Azure](../event-hubs/authorize-access-azure-active-directory.md#azure-built-in-roles-for-azure-event-hubs) | 
| Хранилище BLOB-объектов Azure | [участник данных BLOB-объектов хранилища](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues); |
| Хранилище очередей Azure |[Отправитель сообщений данных в очередь хранилища](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) | 

## <a name="add-an-identity-to-azure-roles-on-destinations"></a>Добавление удостоверения в роли Azure в местах назначения
В этом разделе описывается добавление удостоверения для пользовательского раздела или домена в роль Azure. 

### <a name="use-the-azure-portal"></a>Использование портала Azure
С помощью портал Azure можно назначить для соответствующей роли пользовательский раздел или удостоверение домена, чтобы пользовательский раздел или домен мог пересылать события в назначение. 

В следующем примере показано, как добавить управляемое удостоверение для пользовательского раздела сетки событий с именем **мситесттопик** в роль **отправителя данных служебной шины Azure** для пространства имен служебной шины, содержащего ресурс очереди или раздела. При добавлении к роли на уровне пространства имен настраиваемый раздел "Сетка событий" может пересылать события во все сущности в пространстве имен. 

1. Перейдите к **пространству имен служебной шины** в [портал Azure](https://portal.azure.com). 
1. На левой панели выберите пункт **Управление доступом** . 
1. В разделе **Добавление назначения роли** выберите **Добавить**. 
1. На странице **Добавление назначения роли** выполните следующие действия.
    1. Выберите роль. В этом случае используется роль **Отправитель данных служебной шины Azure**. 
    1. Выберите **удостоверение** для пользовательского раздела или предметной области сетки событий. 
    1. Нажмите кнопку **сохранить** , чтобы сохранить конфигурацию.

Эти действие аналогично добавлению удостоверения для других ролей, упомянутых в таблице. 

### <a name="use-the-azure-cli"></a>Использование Azure CLI
В примере в этом разделе показано, как с помощью Azure CLI добавить удостоверение в роль Azure. Примеры команд предназначены для пользовательских разделов сетки событий. Для доменов сетки событий используются похожие команды. 

#### <a name="get-the-principal-id-for-the-custom-topics-system-identity"></a>Получение идентификатора субъекта для удостоверения системы пользовательского раздела 
Сначала получите идентификатор субъекта для управляемого системой удостоверения пользовательского раздела и назначьте удостоверение соответствующим ролям.

```azurecli-interactive
topic_pid=$(az ad sp list --display-name "$<TOPIC NAME>" --query [].objectId -o tsv)
```

#### <a name="create-a-role-assignment-for-event-hubs-at-various-scopes"></a>Создание назначения роли для центров событий в различных областях 
В следующем примере интерфейса командной строки показано, как добавить удостоверение пользовательского раздела в роль **отправителя данных концентраторов событий Azure** на уровне пространства имен или на уровне концентратора событий. При создании назначения роли на уровне пространства имен настраиваемый раздел может пересылать события во все концентраторы событий в этом пространстве имен. При создании назначения роли на уровне концентратора событий пользовательский раздел может пересылать события только в этот конкретный концентратор событий. 


```azurecli-interactive
role="Azure Event Hubs Data Sender" 
namespaceresourceid=$(az eventhubs namespace show -n $<EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 
eventhubresourceid=$(az eventhubs eventhub show -n <EVENT HUB NAME> --namespace-name <EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one event hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$eventhubresourceid" 
```

#### <a name="create-a-role-assignment-for-a-service-bus-topic-at-various-scopes"></a>Создание назначения роли для раздела служебной шины в различных областях 
В следующем примере интерфейса командной строки показано, как добавить удостоверение пользовательского раздела "Сетка событий" в роль " **отправитель данных" служебной шины Azure** на уровне пространства имен или на уровне разделов служебной шины. Если вы создаете назначение роли на уровне пространства имен, раздел "Сетка событий" может пересылать события во все сущности (очереди или разделы служебной шины) в пределах этого пространства имен. Если вы создаете назначение ролей на уровне очереди или раздела служебной шины, пользовательский раздел "Сетка событий" может пересылать события только в эту конкретную очередь или раздел служебной шины. 

```azurecli-interactive
role="Azure Service Bus Data Sender" 
namespaceresourceid=$(az servicebus namespace show -n $RG\SB -g "$RG" --query "{I:id}" -o tsv 
sbustopicresourceid=$(az servicebus topic show -n topic1 --namespace-name $RG\SB -g "$RG" --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$sbustopicresourceid" 
```

## <a name="create-event-subscriptions-that-use-an-identity"></a>Создание подписок на события, использующих удостоверение
После создания пользовательского раздела или домена в службе "Сетка событий" с удостоверением, управляемым системой, и добавления удостоверения к соответствующей роли в назначении, можно приступать к созданию подписок, использующих удостоверение. 

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


## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об управляемых удостоверениях службы см. в статье [Управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md). 
