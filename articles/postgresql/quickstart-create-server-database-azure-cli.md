---
title: Краткое руководство. Создание сервера — Azure CLI — База данных Azure для PostgreSQL — отдельный сервер
description: В этом кратком руководстве описывается создание сервера Базы данных Azure для PostgreSQL с помощью Azure CLI.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 06/25/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 1d7b6cd6d61be6df66d215613222c12a0f2c9c5e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105606744"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-by-using-the-azure-cli"></a>Краткое руководство. Создание сервера Базы данных Azure для PostgreSQL с помощью Azure CLI

В этом кратком руководстве описывается, как с помощью команд [Azure CLI](/cli/azure/get-started-with-azure-cli) в [Azure Cloud Shell](https://shell.azure.com) создать один сервер Базы данных Azure для PostgreSQL за пять минут. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

    > [!TIP]
    >  Рассмотрите возможность использования более простой команды Azure CLI [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up), которая сейчас доступна в предварительной версии. Ознакомьтесь с [кратким руководством](./quickstart-create-server-up-azure-cli.md).

- Выберите конкретный идентификатор подписки вашей учетной записи с помощью команды [az account set](/cli/azure/account).

    - Запишите значение **id** из выходных данных команды **az login**, чтобы использовать его в команде в качестве значения аргумента **subscription**. 

        ```azurecli
        az account set --subscription <subscription id>
        ```

    - Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Чтобы отобразить все свои подписки, воспользуйтесь командой [az account list](/cli/azure/account#az-account-list).

## <a name="create-an-azure-database-for-postgresql-server"></a>Создание сервера Базы данных Azure для PostgreSQL

Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью команды [az group create](/cli/azure/group#az-group-create). Затем создайте сервер PostgreSQL в этой группе ресурсов. Необходимо указать уникальное имя. В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Создайте [сервер Базы данных Azure для PostgreSQL](overview.md) с помощью команды [az postgres server create](/cli/azure/postgres/server). Сервер может управлять несколькими базами данных.

```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```
Здесь приведены подробные сведения о приведенных выше аргументах. 

**Параметр** | **Пример значения** | **Описание**
---|---|---
name | mydemoserver | Уникальное имя, позволяющее идентифицировать сервер Базы данных Azure для PostgreSQL. Имя сервера может содержать только строчные буквы, цифры и знак дефиса (-). Длина должна составлять от 3 до 63 символов. Дополнительные сведения см. в статье [Правила именования в службе "База данных Azure для PostgreSQL"](../azure-resource-manager/management/resource-name-rules.md#microsoftdbforpostgresql).
resource-group | myresourcegroup | Имя группы ресурсов Azure.
location | westus | Расположение сервера в Azure.
admin-user | myadmin | Имя пользователя для входа в качестве администратора. Не может иметь значение **azure_superuser**, **admin**, **administrator**, **root**, **guest** или **public**.
admin-password | *Надежный пароль* | Пароль администратора. Должен содержать от 8 до 128 знаков из таких трех категорий: прописные латинские буквы, строчные латинские буквы, цифры и небуквенно-цифровые знаки.
sku-name|GP_Gen5_2| Имя ценовой категории и конфигурация вычислительных ресурсов. В сокращенной записи соответствует схеме {ценовая категория} _{поколение вычислительных ресурсов}_ {число виртуальных ядер}. Дополнительные сведения см. на странице [цен на Базу данных Azure для PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/).

>[!IMPORTANT] 
>- Версия PostgreSQL на вашем сервере по умолчанию — 9.6. Чтобы просмотреть все поддерживаемые версии, см. статью [Поддерживаемые основные номера версии PostgreSQL](./concepts-supported-versions.md).
>- Все аргументы команды **az postgres server create** можно просмотреть в этом [справочном документе](/cli/azure/postgres/server#az-postgres-server-create).
>- Протокол SSL включен на сервере по умолчанию. Дополнительные сведения об SSL см. в статье [Настройка SSL-подключения](./concepts-ssl-connection-security.md).

## <a name="configure-a-server-level-firewall-rule"></a>Настройка правила брандмауэра на уровне сервера 
По умолчанию созданный сервер защищен правилами брандмауэра и не является общедоступным. Вы можете настроить правило брандмауэра на вашем сервере с помощью команды [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule), чтобы предоставить вашей локальной среде доступ для подключения к серверу. 

В приведенном ниже примере создается правило брандмауэра с именем `AllowMyIP`, которое разрешает подключения с определенного IP-адреса — 192.168.0.1. Замените IP-адрес или диапазон IP-адресов, соответствующий расположению, из которого будет устанавливаться подключение. Если вы не знаете свой IP-адрес, перейдите на сайт [WhatIsMyIPAddress.com](https://whatismyipaddress.com/), чтобы его получить.


```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Во избежание проблем с подключением убедитесь, что в брандмауэре сети разрешен порт 5432. Серверы службы "База данных Azure для PostgreSQL" используют этот порт. 

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, укажите сведения об узле и учетные данные для доступа.

```azurecli-interactive
az postgres server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите значения **administratorLogin** и **fullyQualifiedDomainName**.

```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/mydemoserver",
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
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"
}
```

## <a name="connect-to-the-azure-database-for-postgresql-server-by-using-psql"></a>Подключение к Базе данных Azure для PostgreSQL с помощью psql
Клиент [psql](https://www.postgresql.org/docs/current/static/app-psql.html) — популярное средство, используемое для подключения к серверам PostgreSQL. Вы можете подключиться к серверу, используя psql в [Azure Cloud Shell](../cloud-shell/overview.md). Вы можете также использовать psql в локальной среде, если она доступна. С новым сервером PostgreSQL автоматически создается пустая база данных **postgres**. Эту базу данных можно использовать для подключения к psql, как показано в следующем коде. 

   ```bash
 psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

> [!TIP]
> Если вы предпочитаете использовать URL-путь для подключения к Postgres, закодируйте с помощью URL-адреса знак @ в имени пользователя с использованием `%40`. Например, строка подключения для psql будет выглядеть так:
>
> ```
> psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres
> ```


## <a name="clean-up-resources"></a>Очистка ресурсов
Если эти ресурсы не требуются для изучения другого краткого руководства или учебника, вы можете их удалить. Для этого выполните следующую команду. 

```azurecli-interactive
az group delete --name myresourcegroup
```

Если вы хотите удалить созданный сервер, воспользуйтесь командой [az postgres server delete](/cli/azure/postgres/server).

```azurecli-interactive
az postgres server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Дальнейшие шаги
> [!div class="nextstepaction"]
> [Перенос базы данных с помощью экспорта и импорта](./howto-migrate-using-export-and-import.md)
