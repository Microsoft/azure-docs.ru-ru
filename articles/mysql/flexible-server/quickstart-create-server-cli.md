---
title: Краткое руководство. Создание сервера Базы данных Azure для MySQL (Гибкий сервер) с помощью Azure CLI
description: В этом кратком руководстве описывается, как создать Базу данных Azure для MySQL (Гибкий сервер) в группе ресурсов Azure с помощью Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 65cc3d2fdcbdea934e80a5f0012ca4f3da157ca3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94843440"
---
# <a name="quickstart-create-an-azure-database-for-mysql-flexible-server-using-azure-cli"></a>Краткое руководство. Создание Базы данных Azure для MySQL (Гибкий сервер) с помощью Azure CLI

В этом кратком руководстве описывается, как с помощью команд [Azure CLI](/cli/azure/get-started-with-azure-cli) в [Azure Cloud Shell](https://shell.azure.com) за 5 минут создать гибкий сервер Базы данных Azure для MySQL. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

> [!IMPORTANT] 
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью.

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Кроме того, Cloud Shell можно открыть в отдельной вкладке браузера. Для этого перейдите на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу **ВВОД**, чтобы выполнить его.

Если вы решили установить и использовать CLI локально, для выполнения инструкций, приведенных в этом кратком руководстве, вам потребуется Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Предварительные требования

Вам потребуется выполнить вход в учетную запись с помощью команды [az login](/cli/azure/reference-index#az-login). Обратите внимание на свойство **идентификатора**, которое ссылается на **идентификатор подписки** вашей учетной записи Azure.

```azurecli-interactive
az login
```

Выберите конкретную подписку вашей учетной записи, выполнив команду [az account set](/cli/azure/account#az-account-set). Запишите значение **идентификатора** из выходных данных команды **az login**, чтобы использовать его в команде в качестве значения аргумента **подписки**. Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Чтобы отобразить все ваши подписки, выполните команду [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-a-flexible-server"></a>Создание гибкого сервера

Создайте [группу ресурсов Azure](../../azure-resource-manager/management/overview.md) с помощью команды `az group create`, а затем создайте гибкий сервер MySQL в этой группе ресурсов. Необходимо указать уникальное имя. В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `eastus2`.

```azurecli-interactive
az group create --name myresourcegroup --location eastus2
```

Создайте гибкий сервер с помощью команды `az mysql flexible-server create`. Сервер может управлять несколькими базами данных. Следующая команда создает сервер, используя параметры и значения по умолчанию для [локального контекста](/cli/azure/local-context) Azure CLI: 

```azurecli
az mysql flexible-server create
```

Сервер создается со следующими атрибутами: 
- Автоматически созданное имя сервера, имя пользователя и пароль администратора, имя группы ресурсов (если не указано в локальном контексте) и расположение, которое выбрано для группы ресурсов. 
- Службы по умолчанию для остальных конфигураций сервера: уровень вычислений (с накапливаемыми ресурсами), размер (уровень) вычислительных ресурсов или номер SKU (B1MS), срок хранения резервных копий (7 дней) и версия MySQL (5.7).
- Для метода подключения по умолчанию назначается вариант "Закрытый доступ (интеграция с виртуальной сетью)" с автоматически созданными виртуальной сетью и подсетью.

> [!NOTE] 
> После создания сервера нельзя изменить метод подключения. Например, если во время создания вы выбрали *Закрытый доступ (интеграция с виртуальной сетью)* , то позднее не сможете назначить *Открытый доступ (разрешенные IP-адреса)* . Мы настоятельно рекомендуем создать сервер с закрытым доступом, чтобы безопасно обращаться к нему через интеграцию с виртуальной сетью. Дополнительные сведения о закрытом доступе см. в статье с [основными понятиями](./concepts-networking.md).

Если вы хотите изменить значения по умолчанию, воспользуйтесь [справочной документацией](/cli/azure/mysql/flexible-server) по Azure CLI, где описаны все настраиваемые параметры интерфейса командной строки. 

Ниже приведен пример выходных данных: 

```json
Command group 'mysql flexible-server' is in preview. It may be changed/removed in a future release.
Creating Resource Group 'groupXXXXXXXXXX'...
Creating new vnet "serverXXXXXXXXXVNET" in resource group "groupXXXXXXXXXX"...
Creating new subnet "serverXXXXXXXXXSubnet" in resource group "groupXXXXXXXXXX" and delegating it to "Microsoft.DBforMySQL/flexibleServers"...
Creating MySQL Server 'serverXXXXXXXXX' in group 'groupXXXXXXXXXX'...
Your server 'serverXXXXXXXXX' is using sku 'Standard_B1ms' (Paid Tier). Please refer to https://aka.ms/mysql-pricing for pricing details
Creating MySQL database 'flexibleserverdb'...
Make a note of your password. If you forget, you would have to reset your password with 'az mysql flexible-server update -n serverXXXXXXXXX -g groupXXXXXXXXXX -p <new-password>'.
{
  "connectionString": "server=serverXXXXXXXXX.mysql.database.azure.com;database=flexibleserverdb;uid=secureusername;pwd=securepasswordstring",
  "databaseName": "flexibleserverdb",
  "host": "serverXXXXXXXXX.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.DBforMySQL/flexibleServers/serverXXXXXXXXX",
  "location": "East US 2",
  "password": "securepasswordstring",
  "resourceGroup": "groupXXXXXXXXXX",
  "skuname": "Standard_B1ms",
  "subnetId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.Network/virtualNetworks/serverXXXXXXXXXVNET/subnets/serverXXXXXXXXXSubnet",
  "username": "secureusername",
  "version": "5.7"
}
```

Если вы хотите изменить значения по умолчанию, воспользуйтесь [справочной документацией](/cli/azure/mysql/flexible-server) по Azure CLI, где описаны все настраиваемые параметры интерфейса командной строки. 

> [!NOTE]
> Подключитесь к базе данных Azure для MySQL через порт 3306. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 3306 может быть запрещен. В таком случае вы не сможете подключиться к серверу. Для этого ваш ИТ-отдел должен открыть порт 3306.

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа.

```azurecli-interactive
az mysql flexible-server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите значения **fullyQualifiedDomainName** и **administratorLogin**. Ниже приведен пример выходных данных JSON: 

```json
{
  "administratorLogin": "myadminusername",
  "administratorLoginPassword": null,
  "delegatedSubnetArguments": {
    "subnetArmResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/mydemoserverVNET/subnets/mydemoserverSubnet"
  },
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/flexibleServers/mydemoserver",
  "location": "East US 2",
  "name": "mydemoserver",
  "publicNetworkAccess": "Disabled",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 0,
    "name": "Standard_B1ms",
    "tier": "Burstable"
  },
  "storageProfile": {
    "backupRetentionDays": 7,
    "fileStorageSkuName": "Premium_LRS",
    "storageAutogrow": "Disabled",
    "storageIops": 0,
    "storageMb": 10240
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/flexibleServers",
  "version": "5.7"
}
```

## <a name="connect-using-mysql-command-line-client"></a>Подключение с помощью клиента командной строки mysql

Так как вы создали гибкий сервер в режиме *Закрытый доступ (интеграция с виртуальной сетью)* , к этому серверу придется подключаться из другого ресурса в той же виртуальной сети. Для этого вы можете создать виртуальную машину и добавить ее к созданной виртуальной сети. 

После создания виртуальной машины вы сможете подключиться к ней по протоколу SSH и установить популярное клиентское средство командной строки **[mysql.exe](https://dev.mysql.com/downloads/)** .

Создайте подключение из mysql.exe, используя приведенную ниже команду. Замените в ней предложенные значения реальными значениями имени сервера и пароля. 

```bash
 mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если эти ресурсы не требуются для изучения другого руководства, вы можете их удалить. Для этого выполните следующую команду:

```azurecli-interactive
az group delete --name myresourcegroup
```

Если вы хотите удалить только что созданный сервер, выполните команду `az mysql server delete`.

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Следующие шаги

> [!div class="nextstepaction"]
>[Создание веб-приложения PHP (Laravel) с использованием MySQL](tutorial-php-database-app.md)
