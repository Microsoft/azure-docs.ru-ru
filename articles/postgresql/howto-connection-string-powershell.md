---
title: Создание строки подключения с помощью PowerShell — база данных Azure для PostgreSQL
description: В этой статье приведен пример Azure PowerShell для создания строки подключения для подключения к базе данных Azure для PostgreSQL.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc, devx-track-azurepowershell
ms.topic: how-to
ms.date: 8/6/2020
ms.openlocfilehash: 29641bdc3696f0722ca430dde912768071fd9b16
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91708175"
---
# <a name="how-to-generate-an-azure-database-for-postgresql-connection-string-with-powershell"></a>Создание строки подключения базы данных Azure для PostgreSQL с помощью PowerShell

В этой статье показано, как создать строку подключения для сервера базы данных Azure для PostgreSQL. Строку подключения можно использовать для подключения к базе данных Azure для PostgreSQL из множества различных приложений.

## <a name="requirements"></a>Требования

В качестве отправной точки в этой статье используются ресурсы, созданные в соответствии со следующим руководством:

* [Краткое руководство. Создание сервера базы данных Azure для PostgreSQL с помощью PowerShell](quickstart-create-postgresql-server-database-using-azure-powershell.md)

## <a name="get-the-connection-string"></a>Получение строки подключения

`Get-AzPostgreSqlConnectionString`Командлет используется для создания строки подключения для подключения приложений к базе данных Azure для PostgreSQL. В примере ниже возвращается строка подключения для клиента PHP от **mydemoserver**.

```azurepowershell-interactive
Get-AzPostgreSqlConnectionString -Client PHP -Name mydemoserver -ResourceGroupName myresourcegroup
```

```Output
host=mydemoserver.postgres.database.azure.com port=5432 dbname={your_database} user=myadmin@mydemoserver password={your_password} sslmode=require
```

Допустимые значения для параметра `Client` включают следующие:

* ADO&#46;NET
* JDBC
* Node.js
* PHP
* Python
* Ruby
* WebApp

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка параметров сервера базы данных Azure для PostgreSQL с помощью PowerShell](howto-configure-server-parameters-using-powershell.md)
