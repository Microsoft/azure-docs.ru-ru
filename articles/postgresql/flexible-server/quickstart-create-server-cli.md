---
title: Краткое руководство. Создание сервера в Базе данных Azure для PostgreSQL (Гибкий сервер) с помощью Azure CLI
description: В этом кратком руководстве объясняется, как создать сервер для Базы данных Azure для PostgreSQL (Гибкий сервер) в группе ресурсов Azure с помощью Azure CLI.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 49c98905d99e5895dd0d2aa7e6a06e7a3605897e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105606136"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-flexible-server-using-azure-cli"></a>Краткое руководство. Создание сервера Базы данных Azure для PostgreSQL (Гибкий сервер) с помощью Azure CLI

В этом кратком руководстве объясняется, как с помощью команд [Azure CLI](/cli/azure/get-started-with-azure-cli) в [Azure Cloud Shell](https://shell.azure.com) за пять минут создать сервер Базы данных Azure для PostgreSQL (Гибкий сервер). Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

> [!IMPORTANT] 
> Сейчас доступна предварительная версия Гибкого сервера Базы данных Azure для PostgreSQL.

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

Создайте [группу ресурсов Azure](../../azure-resource-manager/management/overview.md) с помощью команды `az group create`, а затем создайте гибкий сервер PostgreSQL в этой группе ресурсов. Необходимо указать уникальное имя. В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Создайте гибкий сервер с помощью команды `az postgres flexible-server create`. Сервер может управлять несколькими базами данных. Следующая команда создает сервер, используя параметры и значения по умолчанию для [локального контекста](/cli/azure/local-context) Azure CLI: 

```azurecli
az postgres flexible-server create
```

Сервер создается со следующими атрибутами: 
- Автоматически созданное имя сервера, имя пользователя и пароль администратора, имя группы ресурсов (если оно не указано в локальном контексте) и расположение, которое выбрано для группы ресурсов. 
- Службы по умолчанию для остальных конфигураций сервера: уровень вычислений (общего назначения), размер (уровень) вычислительных ресурсов (D2s_v3 — 2 виртуальных ядра, 8 ГБ ОЗУ), срок хранения резервных копий (7 дней) и версия PostgreSQL (12).
- Для метода подключения по умолчанию назначается вариант "Закрытый доступ (интеграция с виртуальной сетью)" с автоматически созданными виртуальной сетью и подсетью.

> [!NOTE] 
> После создания сервера нельзя изменить метод подключения. Например, если во время создания вы выбрали *Закрытый доступ (интеграция с виртуальной сетью)* , то позднее не сможете назначить *Открытый доступ (разрешенные IP-адреса)* . Мы настоятельно рекомендуем создать сервер с закрытым доступом, чтобы безопасно обращаться к нему через интеграцию с виртуальной сетью. Дополнительные сведения о закрытом доступе см. в статье с [основными понятиями](./concepts-networking.md).

Если вы хотите изменить значения по умолчанию, воспользуйтесь справочной документацией по Azure CLI, <!--FIXME --> где описаны все настраиваемые параметры интерфейса командной строки. 

> [!NOTE]
> Подключения к Базе данных Azure для PostgreSQL используют порт 5432. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 5432 может быть запрещен. В таком случае вы не сможете подключиться к серверу. Для этого вашему ИТ-отделу следует открыть порт 5432.

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа.

```azurecli-interactive
az postgres flexible-server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите значения **fullyQualifiedDomainName** и **administratorLogin**.

<!--FIXME-->
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/flexibleServers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "name": "Standard_D2s_v3",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "publicAccess": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 131072
  },
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/flexibleServers",
  "userVisibleState": "Ready",
  "version": "12"
}
```

## <a name="connect-using-postgresql-command-line-client"></a>Подключение с помощью клиента командной строки PostgreSQL

Так как вы создали гибкий сервер в режиме *Закрытый доступ (интеграция с виртуальной сетью)* , к этому серверу придется подключаться из другого ресурса в той же виртуальной сети. Для этого вы можете создать виртуальную машину и добавить ее к созданной виртуальной сети. 

После создания виртуальной машины вы сможете подключиться к этому компьютеру по протоколу SSH и установить программу командной строки **[psql](https://www.postgresql.org/download/)** .

Создайте подключение с помощью psql, используя приведенную ниже команду. Замените в ней предложенные значения реальными значениями имени сервера и пароля. 

```bash
psql -h mydemoserver.postgres.database.azure.com -u mydemouser -p
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если эти ресурсы не требуются для изучения другого руководства, вы можете их удалить. Для этого выполните следующую команду:

```azurecli-interactive
az group delete --name myresourcegroup
```

Если вы хотите удалить только что созданный сервер, выполните команду `az postgres flexible-server delete`.

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Следующие шаги

> [!div class="nextstepaction"]
>[Развертывание приложения Django с помощью Службы приложений и PostgreSQL](tutorial-django-app-service-postgres.md)
