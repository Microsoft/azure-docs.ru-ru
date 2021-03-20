---
title: Автоматическое увеличение хранилища Azure CLI — база данных Azure для MySQL
description: В этой статье описывается, как включить автоматическое расширение хранилища с помощью Azure CLI в базе данных Azure для MySQL.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: bee84c41d95c0220129fbf2133b57e1ad35eebdc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94542054"
---
# <a name="auto-grow-azure-database-for-mysql-storage-using-the-azure-cli"></a>Автоматическое увеличение базы данных Azure для хранилища MySQL с помощью Azure CLI
В этой статье описывается, как можно настроить расширение хранилища базы данных Azure для MySQL, не влияя на рабочую нагрузку.

Сервер достигает [предельного размера хранилища](./concepts-pricing-tiers.md#reaching-the-storage-limit)и устанавливается в значение "только для чтения". Если автоматическое увеличение размера хранилища включено, то для серверов с подготовленным хранилищем объемом менее 100 ГБ размер подготовленного хранилища увеличивается на 5 ГБ, как только объем свободного хранилища превысит 1 ГБ или 10% подготовленного хранилища. Для серверов с более чем 100 ГБ подготовленного хранилища размер подготовленного хранилища увеличивается на 5%, если объем свободного пространства превышает 5% от подготовленного объема хранилища. Максимальный объем хранилища, указанный [здесь](./concepts-pricing-tiers.md#storage) , применим.

## <a name="prerequisites"></a>Предварительные условия

Для работы с этим руководством должны быть выполнены следующие условия.

- Необходим [сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="enable-mysql-server-storage-auto-grow"></a>Включение автоматического роста хранилища MySQL Server

Включите автоматическое расширение хранилища на сервере на существующем сервере с помощью следующей команды:

```azurecli-interactive
az mysql server update --name mydemoserver --resource-group myresourcegroup --auto-grow Enabled
```

Включите автоматическое увеличение размера сервера при создании нового сервера с помощью следующей команды:

```azurecli-interactive
az mysql server create --resource-group myresourcegroup --name mydemoserver  --auto-grow Enabled --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 5.7
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте [, как создавать оповещения по метрикам](howto-alert-on-metric.md).