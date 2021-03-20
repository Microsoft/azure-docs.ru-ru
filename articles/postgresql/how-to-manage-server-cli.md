---
title: Управление сервером-Azure CLI — база данных Azure для PostgreSQL
description: Узнайте, как управлять сервером базы данных Azure для PostgreSQL из Azure CLI.
author: ajlam
ms.author: andrela
ms.service: postgresql
ms.topic: how-to
ms.date: 9/22/2020
ms.openlocfilehash: 2ea07e2bc12e6fc0d62abd462b8537c6a93689f9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97935807"
---
# <a name="manage-an-azure-database-for-postgresql-single-server-using-the-azure-cli"></a>Управление базой данных Azure для PostgreSQL на одном сервере с помощью Azure CLI

В этой статье показано, как управлять отдельными серверами, развернутыми в Azure. Задачи управления включают в себя вычисление и масштабирование хранилища, сброс пароля администратора и Просмотр сведений о сервере.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу. Для этой статьи требуется запустить локально Azure CLI версии 2.0 или более поздней. Чтобы узнать, какая установлена версия, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Вам потребуется выполнить вход в учетную запись с помощью команды [az login](/cli/azure/reference-index#az-login). Обратите внимание на свойство **идентификатора**, которое ссылается на **идентификатор подписки** вашей учетной записи Azure.

```azurecli-interactive
az login
```

Выберите конкретную подписку вашей учетной записи, выполнив команду [az account set](/cli/azure/account). Запишите значение **идентификатора** из выходных данных команды **az login**, чтобы использовать его в команде в качестве значения аргумента **подписки**. Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Чтобы отобразить все ваши подписки, выполните команду [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

Если вы еще не создали сервер, обратитесь к этому [краткому](quickstart-create-server-database-azure-cli.md) руководству, чтобы создать его.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

## <a name="scale-compute-and-storage"></a>Масштабирование вычислений и хранилища

Вы можете легко масштабировать ценовую категорию, вычисления и хранилище с помощью следующей команды. Вы можете просмотреть все операции сервера, которые можно выполнить с помощью команды [AZ postgres Server Overview](/cli/azure/mysql/server) .

```azurecli-interactive
az postgres server update --resource-group myresourcegroup --name mydemoserver --sku-name GP_Gen5_4 --storage-size 6144
```

Здесь приведены подробные сведения о приведенных выше аргументах.

**Параметр** | **Пример значения** | **Описание**
---|---|---
name | mydemoserver | Введите уникальное имя для сервера базы данных Azure для PostgreSQL. Имя сервера может содержать только строчные буквы, цифры и знак дефиса (-). Его длина должна составлять от 3 до 63 символов.
resource-group | myresourcegroup | Укажите имя группы ресурсов Azure.
sku-name|GP_Gen5_2|Введите имя ценовой категории и конфигурацию вычислительных ресурсов. В сокращенной записи соответствует схеме {ценовая категория} _{поколение вычислительных ресурсов}_ {число виртуальных ядер}. Дополнительные сведения см. на странице с [ценовыми категориями](./concepts-pricing-tiers.md).
storage-size | 6144 | Объем хранилища сервера (в мегабайтах). Минимум 5120 и увеличивается в 1024.

> [!Important]
> - Хранилище можно масштабировать (Однако уменьшить масштаб хранилища невозможно)
> - Масштабирование от базового до общего назначения или ценовой категории, оптимизированной для памяти, не поддерживается. Вы можете вручную выполнить масштабирование с [помощью скрипта Bash](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/upgrade-from-basic-to-general-purpose-or-memory-optimized-tiers/ba-p/830404) или [с помощью PostgreSQL Workbench](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-scale-up-azure-database-for-mysql-from-basic-tier-to/ba-p/369134) .


## <a name="manage-postgresql-databases-on-a-server"></a>Управление базами данных PostgreSQL на сервере.
Любую из этих команд можно использовать для создания, удаления, перечисления и просмотра свойств базы данных на сервере.

| Командлет | Использование| Описание |
| --- | ---| --- |
|[AZ postgres DB CREATE](/cli/azure/sql/db#az-mysql-db-create)|```az postgres db create -g myresourcegroup -s mydemoserver -n mydatabasename``` |Создает базу данных.|
|[AZ postgres DB Delete](/cli/azure/sql/db#az-mysql-db-delete)|```az postgres db delete -g myresourcegroup -s mydemoserver -n mydatabasename```|Удалите базу данных с сервера. Эта команда не удаляет сервер. |
|[AZ postgres DB List](/cli/azure/sql/db#az-mysql-db-list)|```az postgres db list -g myresourcegroup -s mydemoserver```|Список всех баз данных на сервере|
|[AZ postgres DB, демонстрация](/cli/azure/sql/db#az-mysql-db-show)|```az postgres db show -g myresourcegroup -s mydemoserver -n mydatabasename```|Отображает дополнительные сведения о базе данных|

## <a name="update-admin-password"></a>Обновление пароля администратора
Вы можете изменить пароль роли администратора с помощью этой команды.
```azurecli-interactive
az postgres server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!Important]
>  Убедитесь, что пароль имеет длину не менее 8 символов и не более 128 символов.
> Пароль должен содержать символы трех из следующих категорий: прописные буквы английского алфавита, строчные буквы английского алфавита, цифры и символы, отличные от буквенно-цифровых.

## <a name="delete-a-server"></a>Удаление сервера
Если вы хотите удалить PostgreSQL один сервер, можно выполнить команду [AZ postgres Server DELETE](/cli/azure/mysql/server#az-mysql-server-delete) .

```azurecli-interactive
az postgres server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия
- [Перезапуск сервера](howto-restart-server-cli.md)
- [Восстановление сервера в неправильном состоянии](howto-restore-server-cli.md)
- [Мониторинг и настройка сервера](concepts-monitoring.md)
