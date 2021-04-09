---
title: Руководство по Отправка настраиваемых событий в гибридное подключение с помощью Сетки событий
description: Руководство по Используйте службу "Сетка событий Azure" и Azure CLI, чтобы иметь возможность публиковать темы и подписываться на эти события. Гибридное подключение используется для конечной точки.
ms.date: 07/07/2020
ms.topic: tutorial
ms.custom: devx-track-azurecli
ms.openlocfilehash: c709d77827551860cc917c3c84c5a849d9fdc512
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94566832"
---
# <a name="tutorial-route-custom-events-to-azure-relay-hybrid-connections-with-azure-cli-and-event-grid"></a>Руководство по Маршрутизация пользовательских событий на гибридные подключения Azure Relay с помощью Azure CLI и службы "Сетка событий"

"Сетка событий Azure" — это служба обработки событий для облака. Гибридные подключения Azure Relay — один из поддерживаемых обработчиков событий. Гибридные подключения используются как обработчик событий, когда требуется обработать события из приложения, для которого не предусмотрена общедоступная конечная точка. Эти приложения могут находиться в корпоративной сети организации. В этой статье описано, как с помощью Azure CLI создать пользовательский раздел, подписаться на него и активировать событие для просмотра результата. События отправляются по гибридному подключению.

## <a name="prerequisites"></a>Предварительные требования

- В этой статье предполагается, что у вас уже есть гибридное подключение и приложение-прослушиватель. Чтобы приступить к работе с гибридными подключениями, см. статью [Приступая к работе с гибридными подключениями к ретранслятору](../azure-relay/relay-hybrid-connections-dotnet-get-started.md) (.NET) или [Приступая к работе с гибридными подключениями к ретранслятору](../azure-relay/relay-hybrid-connections-node-get-started.md) (Node).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.56 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Темами событий сетки являются ресурсы Azure, которые необходимо поместить в группу ресурсов Azure. Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). 

В следующем примере создается группа ресурсов с именем *gridResourceGroup* в расположении *westus2*.

```azurecli-interactive
az group create --name gridResourceGroup --location westus2
```

## <a name="create-a-custom-topic"></a>Создание пользовательской темы

Раздел сетки событий содержит определяемую пользователем конечную точку, в которой можно размещать свои события. В приведенном ниже примере создается пользовательский раздел в вашей группе ресурсов. Замените `<topic_name>` уникальным именем для вашего пользовательского раздела. Имя раздела сетки событий должно быть уникальным, так как оно представлено записью службы доменных имен (DNS).

```azurecli-interactive
az eventgrid topic create --name <topic_name> -l westus2 -g gridResourceGroup
```

## <a name="subscribe-to-a-custom-topic"></a>Создание подписки на события пользовательского раздела

Вы можете создать подписку на раздел сетки событий, чтобы определить в Сетке событий события, которые вы хотите отслеживать. В следующем примере создается подписка на созданный пользовательский раздел и передается идентификатор ресурса гибридного подключения для конечной точки. Идентификатор гибридного подключения представляется в таком формате:

`/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Relay/namespaces/<relay-namespace>/hybridConnections/<hybrid-connection-name>`

Следующий скрипт получает идентификатор ресурса пространства имен ретранслятора. Он создает идентификатор для гибридного подключения и подписку на раздел сетки событий. В нем также задается тип `hybridconnection` для конечной точки и используется идентификатор гибридного подключения для нее.

```azurecli-interactive
relayname=<namespace-name>
relayrg=<resource-group-for-relay>
hybridname=<hybrid-name>

relayid=$(az resource show --name $relayname --resource-group $relayrg --resource-type Microsoft.Relay/namespaces --query id --output tsv)
hybridid="$relayid/hybridConnections/$hybridname"
topicid=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name <event_subscription_name> \
  --endpoint-type hybridconnection \
  --endpoint $hybridid \
  --expiration-date "<yyyy-mm-dd>"
```

Обратите внимание, что задана [дата окончания срока действия](concepts.md#event-subscription-expiration) подписки.

## <a name="create-application-to-process-events"></a>Создание приложения для обработки событий

Вам нужно приложение, которое может получать события из гибридного подключения. Эту операцию выполняет [пример потребителя гибридного подключения сетки событий Microsoft Azure для C#](https://github.com/Azure-Samples/event-grid-dotnet-hybridconnection-destination). Вы уже выполнили необходимые действия.

1. Убедитесь в том, что установлена версия Visual Studio 2019 или более поздняя.

1. Клонируйте репозиторий на локальный компьютер.

1. Загрузите проект HybridConnectionConsumer в Visual Studio.

1. Откройте файл Program.cs, замените `<relayConnectionString>` и `<hybridConnectionName>` на строку релейного соединения и имя созданного гибридного подключения.

1. Скомпилируйте и запустите приложение в Visual Studio.

## <a name="send-an-event-to-your-topic"></a>Отправка события в тему

Давайте активируем событие, чтобы увидеть, как сообщение отправляется в конечную точку при помощи службы "Сетка событий". В этой статье показано, как запустить событие с помощью Azure CLI. Кроме того, можно использовать [приложение издателя "Сетка событий"](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/tree/master/EventGridPublisher).

Сначала получите URL-адрес и ключ для пользовательского раздела. Снова используйте имя пользовательского раздела для `<topic_name>`.

```azurecli-interactive
endpoint=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name <topic_name> -g gridResourceGroup --query "key1" --output tsv)
```

Для простоты используйте пример данных события для отправки в пользовательский раздел. Как правило, приложение или служба Azure отправит данные события. CURL — это служебная программа, которая отправляет HTTP-запросы. В этой статье мы используем CURL для отправки события в пользовательский раздел.

```azurecli-interactive
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Ducati", "model": "Monster"},"dataVersion": "1.0"} ]'
curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
```

Приложение-прослушиватель должно получить сообщение о событии.

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы планируете продолжить работу с этим событием, не удаляйте ресурсы, созданные при работе с этой статьей. В противном случае удалите соответствующие ресурсы с помощью следующей команды.

```azurecli-interactive
az group delete --name gridResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знаете, как создавать темы и подписки на события, ознакомьтесь с дополнительными сведениями о сетке событий, которые могут помочь вам:

- [An introduction to Azure Event Grid](overview.md) (Общие сведения о службе "Сетка событий Azure")
- [Перенаправление событий хранилища BLOB-объектов в пользовательскую конечную веб-точку (предварительная версия)](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)
- [Monitor virtual machine changes with Azure Event Grid and Logic Apps](monitor-virtual-machine-changes-event-grid-logic-app.md) (Отслеживание изменений виртуальной машины с помощью Azure Logic Apps и службы "Сетка событий Azure")
- [Потоковая передача больших данных в хранилище данных](event-grid-event-hubs-integration.md)