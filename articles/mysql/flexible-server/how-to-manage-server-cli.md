---
title: Управление сервером-Azure CLI — гибкий сервер базы данных Azure для MySQL
description: Узнайте, как управлять базой данных Azure для гибкого сервера MySQL из Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: b58a9dd7901f85c59b09bc4ccb197d012bce2200
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92545061"
---
# <a name="manage-an-azure-database-for-mysql---flexible-server-preview-using-the-azure-cli"></a>Управление базой данных Azure для MySQL — гибким сервером (Предварительная версия) с помощью Azure CLI

> [!IMPORTANT]
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

В этой статье показано, как управлять гибким сервером (предварительной версией), развернутым в Azure. Задачи управления включают в себя вычисление и масштабирование хранилища, сброс пароля администратора и Просмотр сведений о сервере.

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

> [!Important]
> Если вы еще не создали гибкий сервер, создайте его, чтобы приступить к работе с этим руководством.

## <a name="scale-compute-and-storage"></a>Масштабирование вычислений и хранилища

Вы можете легко масштабировать свой уровень вычислений, виртуальных ядер и хранилище с помощью следующей команды. Вы можете просмотреть все операции сервера, которые можно выполнить [с помощью команды AZ MySQL гибкое обновление сервера](/cli/azure/mysql/flexible-server#az_mysql_flexible_server_update) .

```azurecli-interactive
az mysql flexible-server update --resource-group myresourcegroup --name mydemoserver --sku-name Standard_D4ds_v4 --storage-size 6144
```

Ниже приведены подробные сведения об аргументах, приведенных выше:

**Параметр** | **Пример значения** | **Описание**
---|---|---
name | mydemoserver | Введите уникальное имя для сервера службы "База данных Azure для MySQL". Имя сервера может содержать только строчные буквы, цифры и знак дефиса (-). Его длина должна составлять от 3 до 63 символов.
resource-group | myresourcegroup | Укажите имя группы ресурсов Azure.
sku-name|Standard_D4ds_v4|Введите имя уровня и размера вычислений. Соответствует соглашению Standard_ {размер виртуальной машины} в сокращенном виде. Дополнительные сведения см. на странице с [ценовыми категориями](../concepts-pricing-tiers.md).
storage-size | 6144 | Объем хранилища сервера (в мегабайтах). Минимум 5120 и увеличивается в 1024.

> [!Important]
> - Хранилище можно масштабировать (Однако уменьшить масштаб хранилища невозможно)


## <a name="manage-mysql-databases-on-a-server"></a>Управление базами данных MySQL на сервере.
Любую из этих команд можно использовать для создания, удаления, перечисления и просмотра свойств базы данных на сервере.

| Командлет | Использование| Описание |
| --- | ---| --- |
|[AZ MySQL гибкость — серверная база данных создать](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_create)|```az mysql flexible-server db create -g myresourcegroup -s mydemoserver -n mydatabasename``` |Создает базу данных.|
|[AZ MySQL гибкость — серверная база данных удаление](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_delete)|```az mysql flexible-server db delete -g myresourcegroup -s mydemoserver -n mydatabasename```|Удалите базу данных с сервера. Эта команда не удаляет сервер. |
|[AZ MySQL гибкий — серверная база данных](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_list)|```az mysql flexible-server db list -g myresourcegroup -s mydemoserver```|Список всех баз данных на сервере|
|[AZ MySQL гибкость — Серверная БД](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_show)|```az mysql flexible-server db show -g myresourcegroup -s mydemoserver -n mydatabasename```|Отображает дополнительные сведения о базе данных|

## <a name="update-admin-password"></a>Обновление пароля администратора
Вы можете изменить пароль роли администратора с помощью этой команды.
```azurecli-interactive
az mysql flexible-server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!Important]
>  Убедитесь, что пароль имеет длину не менее 8 символов и не более 128 символов.
> Пароль должен содержать символы трех из следующих категорий: прописные буквы английского алфавита, строчные буквы английского алфавита, цифры и символы, отличные от буквенно-цифровых.

## <a name="delete-a-server"></a>Удаление сервера
Если вы хотите удалить гибкий сервер MySQL, можно выполнить команду [AZ MySQL гибкий-Server DELETE Server](/cli/azure/mysql/flexible-server#az_mysql_flexible_server_delete) .

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия
- [Узнайте, как запустить или отключить сервер](how-to-stop-start-server-portal.md)
- [Узнайте, как управлять виртуальной сетью](how-to-manage-virtual-network-cli.md)
- [Устранение неполадок с подключением](how-to-troubleshoot-common-connection-issues.md)
- [Создание брандмауэра и управление им](how-to-manage-firewall-cli.md)