---
title: Краткое руководство. Создание сервера Базы данных Azure для MySQL с помощью Azure CLI
description: В этом кратком руководстве описывается создание сервера базы данных Azure для MySQL в группе ресурсов Azure с помощью Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 07/15/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3dd971ff36bde02072d14c465dbafec2823b256d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780323"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-using-azure-cli"></a>Краткое руководство. Создание сервера базы данных Azure для MySQL с помощью Azure CLI

> [!TIP]
> Рассмотрите возможность использования более простой команды Azure CLI [az postgres up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) (сейчас в предварительной версии). Ознакомьтесь с [кратким руководством](./quickstart-create-server-up-azure-cli.md).

В этом кратком руководстве описывается, как с помощью команд [Azure CLI](/cli/azure/get-started-with-azure-cli) в [Azure Cloud Shell](https://shell.azure.com) создать сервер службы "База данных Azure для MySQL" за 5 минут. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим кратким руководством требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

 - Выберите конкретную подписку вашей учетной записи, выполнив команду [az account set](/cli/azure/account). Запишите значение **идентификатора** из выходных данных команды **az login**, чтобы использовать его в команде в качестве значения аргумента **подписки**. Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Чтобы отобразить все ваши подписки, выполните команду [az account list](/cli/azure/account#az_account_list).

   ```azurecli
   az account set --subscription <subscription id>
   ```

## <a name="create-an-azure-database-for-mysql-server"></a>Создайте сервер базы данных Azure для MySQL.
Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью команды [az group create](/cli/azure/group), а затем — сервер MySQL в этой группе ресурсов. Необходимо указать уникальное имя. В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Создайте сервер базы данных Azure для MySQL, выполнив команду [az mysql server create](/cli/azure/mysql/server#az_mysql_server_create). Сервер может управлять несколькими базами данных.

```azurecli
az mysql server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```

Ниже приведены подробные сведения об аргументах, приведенных выше: 

**Параметр** | **Пример значения** | **Описание**
---|---|---
name | mydemoserver | Введите уникальное имя для сервера службы "База данных Azure для MySQL". Имя сервера может содержать только строчные буквы, цифры и знак дефиса (-). Его длина должна составлять от 3 до 63 символов.
resource-group | myresourcegroup | Укажите имя группы ресурсов Azure.
location | westus | Расположение сервера в Azure.
admin-user | myadmin | Имя для входа администратора. Не может иметь значение **azure_superuser**, **admin**, **administrator**, **root**, **guest** или **public**.
admin-password | *Надежный пароль* | Пароль администратора. Пароль должен содержать от 8 до 128 символов. Пароль должен содержать символы из таких трех категорий: прописные латинские буквы, строчные латинские буквы, цифры и небуквенно-цифровые знаки.
sku-name|GP_Gen5_2|Введите имя ценовой категории и конфигурацию вычислительных ресурсов. В сокращенной записи соответствует схеме {ценовая категория} _{поколение вычислительных ресурсов}_ {число виртуальных ядер}. Дополнительные сведения см. на странице с [ценовыми категориями](./concepts-pricing-tiers.md).

>[!IMPORTANT] 
>- Стандартная версия MySQL на вашем сервере — 5.7. В настоящее время доступны также версии 5.6 и 8.0.
>- Все аргументы команды **az mysql server create** можно просмотреть в этом [справочном документе](/cli/azure/mysql/server#az_mysql_server_create).
>- На сервере по умолчанию включен протокол SSL. Дополнительные сведения по SSL см. в статье [Настройка SSL-подключения](howto-configure-ssl.md).

## <a name="configure-a-server-level-firewall-rule"></a>Настройка правила брандмауэра на уровне сервера 
По умолчанию созданный сервер защищен правилами брандмауэра и не является общедоступным. Правило брандмауэра можно настроить на сервере, выполнив команду [az mysql server firewall-rule create](/cli/azure/mysql/server/firewall-rule). Это позволит вам подключиться к серверу локально.

В приведенном ниже примере создается правило брандмауэра с именем `AllowMyIP`, которое разрешает подключения с определенного IP-адреса — 192.168.0.1. Замените IP-адрес, с которого будете подключаться. При необходимости можно использовать диапазон IP-адресов. Если не знаете как найти свой IP-адрес, перейдите на страницу [https://whatismyipaddress.com/](https://whatismyipaddress.com/) для его получения.

```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Подключитесь к базе данных Azure для MySQL через порт 3306. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 3306 может быть запрещен. В таком случае вы не сможете подключиться к серверу. Для этого ваш ИТ-отдел должен открыть порт 3306.

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа.

```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите значения **fullyQualifiedDomainName** и **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-azure-database-for-mysql-server-using-mysql-command-line-client"></a>Подключение к серверу службы "База данных Azure для MySQL" с помощью клиента командной строки mysql
Вы можете подключиться к серверу с помощью популярного клиентского инструмента: средства командной строки **[mysql.exe](https://dev.mysql.com/downloads/)** с [Azure Cloud Shell](../cloud-shell/overview.md). Кроме того, можно использовать командную строку mysql в локальной среде.
```bash
 mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="clean-up-resources"></a>Очистка ресурсов
Если эти ресурсы не требуются для изучения другого руководства, вы можете их удалить. Для этого выполните следующую команду: 

```azurecli-interactive
az group delete --name myresourcegroup
```

Если вы хотите удалить созданный сервер, выполните команду [az mysql server delete](/cli/azure/mysql/server#az_mysql_server_delete).

```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Создание приложения PHP с подключением к MySQL (Windows)](../app-service/tutorial-php-mysql-app.md)