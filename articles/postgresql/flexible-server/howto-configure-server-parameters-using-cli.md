---
title: Настройка параметров — база данных Azure для PostgreSQL — гибкий сервер
description: В этой статье описывается, как настроить параметры postgres в базе данных Azure для PostgreSQL-гибкого сервера с помощью Azure CLI.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 9/22/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 43b14858cc99cac41e277b03171fd4cac4d6eafa
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90940738"
---
# <a name="customize-server-parameters-for-azure-database-for-postgresql---flexible-server-using-azure-cli"></a>Настройка параметров сервера для базы данных Azure для PostgreSQL — гибкого сервера с помощью Azure CLI

С помощью интерфейса командной строки (Azure CLI) можно вывести список параметров конфигурации для сервера Azure PostgreSQL, а также отобразить и обновить их. Подмножество параметров модуля предоставляется на уровне сервера и может быть изменено. 

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- создайте базу данных и сервер базы данных Azure для PostgreSQL, выполнив инструкции из раздела [Создание базы данных Azure для PostgreSQL](quickstart-create-server-cli.md);
- установите интерфейс командной строки [Azure CLI](/cli/azure/install-azure-cli) на компьютере или используйте [Azure Cloud Shell](../../cloud-shell/overview.md) на портале Azure с помощью браузера.

## <a name="list-server-parameters-for-a-flexible-server"></a>Вывод списка параметров сервера для гибкого сервера

Чтобы получить список всех изменяемых параметров на сервере и их значения, выполните команду [AZ postgres гибок-Server parameter list](/cli/azure/postgres/flexible-server/parameter) .

Вы можете вывести список параметров сервера для сервера **mydemoserver.postgres.Database.Azure.com** в разделе **myresourcegroup** группы ресурсов.

```azurecli-interactive
az postgres flexible-server parameter list --resource-group myresourcegroup --server-name mydemoserver
```

## <a name="show-server-parameter-details"></a>Отобразить сведения о параметрах сервера

Чтобы отобразить сведения о конкретном параметре для сервера, выполните команду [AZ postgres гибкого сервера Parameter-отобразить](/cli/azure/postgres/flexible-server/parameter)  .

В этом примере отображаются сведения о параметре сервера **log \_ min \_ Messages** для сервера **mydemoserver.postgres.Database.Azure.com** в группе ресурсов **myresourcegroup.**

```azurecli-interactive
az postgres flexible-server parameter show --name log_min_messages --resource-group myresourcegroup --server-name mydemoserver
```

## <a name="modify-server-parameter-value"></a>Изменение значения параметра сервера

Можно также изменить значение определенного параметра сервера, которое обновляет базовое значение конфигурации для ядра сервера PostgreSQL. Чтобы обновить параметр, используйте команду [AZ postgres гибкие-Server Parameter Set](/cli/azure/postgres/flexible-server/parameter) . 

Чтобы обновить параметр сервера **log \_ min \_ messages** сервера **mydemoserver.postgres.Database.Azure.com** в разделе myresourcegroup группы ресурсов **.**

```azurecli-interactive
az postgres flexible-server parameter set --name log_min_messages --value INFO --resource-group myresourcegroup --server-name mydemoserver
```

Если вы хотите сбросить значение параметра, просто выберите параметр выход из необязательного `--value` параметра, и служба применяет значение по умолчанию. В приведенном выше примере это может выглядеть следующим образом.

```azurecli-interactive
az postgres flexible-server parameter set --name log_min_messages --resource-group myresourcegroup --server-name mydemoserver
```

Эта команда сбрасывает значение параметра **log \_ min \_ Messages** до значения по умолчанию **warning**. Дополнительные сведения о параметрах сервера и допустимых значениях см. в документации по PostgreSQL в разделе Параметры [параметров](https://www.postgresql.org/docs/12/config-setting.html).

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы настроить журналы сервера и получать к ним доступ, ознакомьтесь с разделом [Журналы сервера в базе данных Azure для PostgreSQL](concepts-logging.md).
