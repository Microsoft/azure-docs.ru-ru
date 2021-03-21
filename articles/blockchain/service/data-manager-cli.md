---
title: Настройка Диспетчер данных Блокчейн с помощью Azure CLI Azure Блокчейн Service
description: Создание Диспетчер данных Блокчейн для службы Блокчейн Azure и управление ею с помощью Azure CLI
ms.date: 03/30/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: 867a51b60afa56005bbb297b345f8a9260160ab8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101722642"
---
# <a name="configure-blockchain-data-manager-using-azure-cli"></a>Настройка диспетчера данных блокчейна с помощью Azure CLI

Настройка Диспетчер данных Блокчейн для службы Блокчейн Azure для записи данных блокчейн отправьте их в службу "Сетка событий Azure".

Чтобы настроить экземпляр Диспетчер данных Блокчейн, сделайте следующее:

* Создание экземпляра Блокчейн Manager
* Создание входных данных для узла транзакции службы Azure Блокчейн
* Создание выходных данных в службе "Сетка событий Azure"
* Добавление приложения блокчейна
* Запуск экземпляра

## <a name="prerequisites"></a>Предварительные условия

* Установите последнюю [Azure CLI](/cli/azure/install-azure-cli) и войдите с помощью `az login` .
* Полное [руководство. использование Visual Studio Code для подключения к сети консорциума Блокчейн службы Azure](connect-vscode.md). При использовании Блокчейн Диспетчер данных рекомендуется использовать *стандартный* уровень службы Azure блокчейн.
* Создайте [раздел Сетки событий](../../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic).
* Подробные сведения см. в статье об [обработчиках событий в службе "Сетка событий Azure"](../../event-grid/event-handlers.md).

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью.

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

Если вы решили установить и использовать CLI локально, для выполнения инструкций, приведенных в этом руководстве вам потребуется Azure CLI 2.0.51 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myRG --location eastus
```

## <a name="create-instance"></a>Создание экземпляра

Экземпляр Диспетчер данных Блокчейн отслеживает узел транзакции службы Блокчейн Azure. Экземпляр регистрирует все необработанные данные о блоках и транзакциях из узла транзакций. Блокчейн Диспетчер данных публикует сообщение **равблоккандтрансактионмсг** , которое представляет собой надмножество информации, возвращенной из запросов Web3. ETH- [Block](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#getblock) и- [Transaction](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#gettransaction) .

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Blockchain Data Manager instance name> \
                   --resource-type Microsoft.blockchain/watchers \
                   --is-full-object \
                   --properties <watcher resource properties>
```

| Параметр | Описание |
|-----------|-------------|
| resource-group | Имя группы ресурсов, в которой создается экземпляр Диспетчер данных Блокчейн. |
| name | Имя экземпляра Диспетчер данных Блокчейн. |
| Тип ресурса | Тип ресурса для экземпляра Диспетчер данных Блокчейн — **Microsoft. блокчейн/наблюдатели**. |
| full-object | Указывает, что свойства содержат параметры для ресурса наблюдателя. |
| properties | Строка в формате JSON, содержащая свойства для ресурса наблюдателя. Может передаваться в виде строки или файла.  |

### <a name="create-instance-examples"></a>Примеры создания экземпляров

Пример конфигурации JSON для создания экземпляра Блокчейн Manager в регионе " **Восточная часть США** ".

``` json
{
    "location": "eastus",
    "properties": {
    }
}
```

| Элемент | Описание |
|---------|-------------|
| location | Регион для создания ресурса наблюдателя |
| properties | Свойства, устанавливаемые при создании ресурса наблюдателя |

Создайте экземпляр Диспетчер данных Блокчейн с именем *миватчер* , используя строку JSON для настройки.

``` azurecli-interactive
az resource create \
                    --resource-group myRG \
                    --name mywatcher \
                    --resource-type Microsoft.blockchain/watchers \
                     --is-full-object \
                     --properties '{"location":"eastus"}'
```

Создайте экземпляр Диспетчер данных Блокчейн с именем *миватчер* с помощью JSON-файла конфигурации.

``` azurecli
az resource create \
                    --resource-group myRG \
                    --name mywatcher \
                    --resource-type Microsoft.blockchain/watchers \
                    --is-full-object \
                    --properties @watcher.json
```

## <a name="create-input"></a>Создание входных данных

Входные данные соединяют Блокчейн Диспетчер данных с узлом транзакции службы Блокчейн Azure. Только пользователи с доступом к узлу транзакции могут создать подключение.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Input name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type inputs \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <input resource properties>
```

| Параметр | Описание |
|-----------|-------------|
| resource-group | Имя группы ресурсов, в которой создается входной ресурс. |
| name | Имя входных данных. |
| namespace | Используйте пространство имен поставщика **Microsoft. блокчейн** . |
| Тип ресурса | Тип ресурса для Блокчейн **Диспетчер данных входными данными.** |
| родитель | Путь к наблюдателю, с которым связана входные данные. Например, **наблюдатели или миватчер**. |
| full-object | Указывает, что свойства содержат параметры для входного ресурса. |
| properties | Строка в формате JSON, содержащая свойства для входного ресурса. Может передаваться в виде строки или файла. |

### <a name="input-examples"></a>Примеры входных данных

Пример конфигурации JSON для создания входного ресурса в регионе " *Восточная часть США* ", к которому подключена \<Blockchain member\> .

``` json
{
    "location": "eastus",
    "properties": {
        "inputType": "Ethereum",
        "dataSource": {
            "resourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/blockchainMembers/<Blockchain member>/transactionNodes/transaction-node"
        }
    }
}
```

| Элемент | Описание |
|---------|-------------|
| location | Регион, в котором создается входной ресурс. |
| inputType | Тип главной книги участника службы Azure Блокчейн. В настоящее время поддерживается **Ethereum** . |
| resourceId | Узел транзакции, к которому подключены входные данные. Замените \<Subscription ID\> , \<Resource group\> и \<Blockchain member\> значениями для ресурса узла транзакции. Вход подключается к узлу транзакции по умолчанию для члена службы Azure Блокчейн. |

Создайте вход с именем *myInput* для *миватчер* , используя строку JSON для конфигурации.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myInput \
                   --namespace Microsoft.Blockchain \
                   --resource-type inputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus", "properties":{"inputType":"Ethereum","dataSource":{"resourceId":"/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/BlockchainMembers/<Blockchain member>/transactionNodes/transaction-node"}}}'
```

Создайте вход с именем *myInput* для *миватчер* , используя файл конфигурации JSON.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name input \
                   --namespace Microsoft.Blockchain \ --resource-type inputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @input.json
```

## <a name="create-output"></a>Создание выходных данных

Исходящее подключение отправляет данные блокчейна в службу "Сетка событий Azure". Данные блокчейна можно отправлять в одно или несколько назначений. Диспетчер данных блокчейна для каждого отдельного экземпляра поддерживает несколько исходящих подключений к разделам Сетки событий.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Output name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <output resource properties>
```

| Параметр | Описание |
|-----------|-------------|
| resource-group | Имя группы ресурсов, в которой создается выходной ресурс. |
| name | Имя выходных данных. |
| namespace | Используйте пространство имен поставщика **Microsoft. блокчейн** . |
| Тип ресурса | Тип ресурса для Блокчейн выходных данных Диспетчер данных — **выходы**. |
| родитель | Путь к наблюдателю, с которым связаны выходные данные. Например, **наблюдатели или миватчер**. |
| full-object | Указывает, что свойства содержат параметры для выходного ресурса. |
| properties | Строка в формате JSON, содержащая свойства выходного ресурса. Может передаваться в виде строки или файла. |

### <a name="output-examples"></a>Примеры выходных данных

Пример конфигурации JSON для создания выходного ресурса в регионе " *Восточная часть США* ", подключенного к разделу сетки событий с именем \<event grid topic\> .

``` json
{
    "location": "eastus",
    "properties": {
        "outputType": "EventGrid",
        "dataSource": {
            "resourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.EventGrid/topics/<event grid topic>"
        }
    }
}
```

| Элемент | Описание |
|---------|-------------|
| location | Регион, в котором создается выходной ресурс. |
| outputType | Тип выходных данных. В настоящее время поддерживается **EventGrid** . |
| resourceId | Ресурс, к которому подключены выходные данные. Замените \<Subscription ID\> , \<Resource group\> и \<Blockchain member\> значениями для ресурса сетки событий. |

Создайте выход с именем *мйоутпут* для *миватчер* , который подключается к разделу сетки событий, используя строку конфигурации JSON.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myoutput \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus","properties":{"outputType":"EventGrid","dataSource":{"resourceId":"/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.EventGrid/topics/<event grid topic>"}}}'
```

Создайте выход с именем *мйоутпут* для *миватчер* , который подключается к разделу сетки событий с помощью JSON-файла конфигурации.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name myoutput \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @output.json
```

## <a name="add-blockchain-application"></a>Добавление приложения блокчейн

При добавлении приложения блокчейн Блокчейн Диспетчер данных декодирует события и состояние свойства для приложения. В противном случае отправляются только данные необработанных блоков и необработанных транзакций. Блокчейн Диспетчер данных также обнаруживает контрактные адреса при развертывании контракта. К экземпляру Диспетчер данных Блокчейн можно добавить несколько приложений блокчейн.


> [!IMPORTANT]
> В настоящее время блокчейн приложения, объявляющие [типы массивов](https://solidity.readthedocs.io/en/v0.5.12/types.html#arrays) или [типов сопоставлений](https://solidity.readthedocs.io/en/v0.5.12/types.html#mapping-types) , не полностью поддерживаются. Свойства, объявленные как массив или типы сопоставления, не будут декодированы в сообщениях *контрактпропертиесмсг* или *декодедконтрактевентсмсг* .

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Application name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <Application resource properties>
```

| Параметр | Описание |
|-----------|-------------|
| resource-group | Имя группы ресурсов, в которой создается ресурс приложения. |
| name | Имя приложения. |
| namespace | Используйте пространство имен поставщика **Microsoft. блокчейн** . |
| Тип ресурса | Тип ресурса для приложения Диспетчер данных Блокчейн — это **артефакты**. |
| родитель | Путь к наблюдателю, с которым связано приложение. Например, **наблюдатели или миватчер**. |
| full-object | Указывает, что свойства содержат параметры для ресурса приложения. |
| properties | Строка в формате JSON, содержащая свойства для ресурса приложения. Может передаваться в виде строки или файла. |

### <a name="blockchain-application-examples"></a>Примеры приложений блокчейн

Пример конфигурации JSON для создания ресурса приложения в регионе " *Восточная часть США* ", отслеживающего интеллектуальный контракт, определяемый интерфейсом ABI и байт-адресом контракта.

``` json
{
    "location": "eastus",
    "properties": {
        "artifactType": "EthereumSmartContract",
        "content": {
            "abiFileUrl": "<ABI URL>",
            "bytecodeFileUrl": "<Bytecode URL>",
            "queryTargetTypes": [
                "ContractProperties",
                "ContractEvents"
            ]
        }
    }
}
```

| Элемент | Описание |
|---------|-------------|
| location | Регион, в котором создается ресурс приложения. |
| artifactType | Тип приложения. В настоящее время поддерживается **есереумсмартконтракт** . |
| абифилеурл | URL-адрес для JSON-файла ABI Smart Contract. Дополнительные сведения о получении интерфейса ABI контракта и создании URL-адреса см. в разделе [получение контракта ABI и байт-кода](data-manager-portal.md#get-contract-abi-and-bytecode) и [Создание интерфейса ABI и URL для кода](data-manager-portal.md#create-contract-abi-and-bytecode-url)интерфейса. |
| битекодефилеурл | URL-адрес развернутого JSON-файла байтового кода Smart Contract. Дополнительные сведения о получении развернутого байт-кода для Smart Contract и создании URL-адреса см. в [статьях получение интерфейса ABI и байт кода](data-manager-portal.md#get-contract-abi-and-bytecode) и [Создание интерфейса ABI и URL-адреса кода контракта](data-manager-portal.md#create-contract-abi-and-bytecode-url). Примечание. для Диспетчер данных Блокчейн требуется **развернутый байт**. |
| куеритаржеттипес | Типы опубликованных сообщений. При указании **контрактпропертиес** публикуются типы сообщений *контрактпропертиесмсг* . При указании **контрактевентс** публикуются типы сообщений *декодедконтрактевентсмсг* . Примечание. типы сообщений *равблоккандтрансактионмсг* и *равтрансактионконтракткреатионмсг* всегда публикуются. |

Создайте приложение с именем *myApplication* для *миватчер* , которое отслеживает смарт-контракт, определенный строкой JSON.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myApplication \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus","properties":{"artifactType":"EthereumSmartContract","content":{"abiFileUrl":"<ABI URL>","bytecodeFileUrl":"<Bytecode URL>","queryTargetTypes":["ContractProperties","ContractEvents"]}}}'
```

Создайте приложение с именем *myApplication* для *миватчер* , которое наблюдает за интеллектуальным контрактом, определенным с помощью JSON-файла конфигурации.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name myApplication \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @artifact.json
```

## <a name="start-instance"></a>Запустить экземпляр

При запуске экземпляр Блокчейн Manager отслеживает события блокчейн из определенных входных данных и отправляет данные в определенные выходные данные.

``` azurecli
az resource invoke-action \
                          --action start \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/<Watcher name>
```

| Параметр | Описание |
|-----------|-------------|
| action | Используйте **Start** для запуска наблюдателя. |
| ids | Идентификатор ресурса наблюдателя. Замените \<Subscription ID\> , \<Resource group\> и \<Watcher name\> значениями для ресурса наблюдателя.|

### <a name="start-instance-example"></a>Пример начального экземпляра

Запустите экземпляр Диспетчер данных Блокчейн с именем *миватчер*.

``` azurecli-interactive
az resource invoke-action \
                          --action start \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/mywatcher
```

## <a name="stop-instance"></a>Останавливает экземпляр

Останавливает экземпляр Диспетчер данных Блокчейн.

``` azurecli
az resource invoke-action \
                          --action stop \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/<Watcher name>
```

| Параметр | Описание |
|-----------|-------------|
| action | Чтобы отключить наблюдатель, используйте параметр " **прерывать** ". |
| ids | Имя наблюдателя. Замените \<Subscription ID\> , \<Resource group\> и \<Watcher name\> значениями для ресурса наблюдателя. |

### <a name="stop-watcher-example"></a>Пример наблюдателя для завершения

Останавливает экземпляр с именем *миватчер*.

``` azurecli-interactive
az resource invoke-action \
                          --action stop \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/mywatcher
```

## <a name="delete-instance"></a>Удалить экземпляр

Удалите экземпляр Диспетчер данных Блокчейн.

``` azurecli
az resource delete \
                   --resource-group <Resource group> \
                   --name <Watcher name> \
                   --resource-type Microsoft.Blockchain/watchers
```

| Параметр | Описание |
|-----------|-------------|
| resource-group | Имя группы ресурсов наблюдателя, который нужно удалить. |
| name | Имя удаляемого наблюдателя. |
| Тип ресурса | Тип ресурса для наблюдателя Диспетчер данных Блокчейн — **Microsoft. блокчейн/наблюдатели**. |

### <a name="delete-instance-example"></a>Пример удаления экземпляра

Удалите экземпляр с именем *миватчер* в группе ресурсов *myRG* .

``` azurecli-interactive
az resource delete \
                   --resource-group myRG \
                   --name mywatcher \
                   --resource-type Microsoft.blockchain/watchers
```

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте следующее руководство по созданию обозревателя сообщений блокчейн TRANSACTION с помощью Блокчейн Диспетчер данных и Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Использование диспетчера данных блокчейна для отправки данных в Azure Cosmos DB](data-manager-cosmosdb.md)
