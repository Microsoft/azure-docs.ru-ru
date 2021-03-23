---
title: Двойное шифрование инфраструктуры — портал Azure — база данных Azure для PostgreSQL
description: Узнайте, как настроить и управлять двойным шифрованием инфраструктуры для базы данных Azure для PostgreSQL.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: how-to
ms.date: 03/14/2021
ms.openlocfilehash: 0b1079a084072556ae79a18a12f6b35f74c92775
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104771657"
---
# <a name="infrastructure-double-encryption-for-azure-database-for-postgresql"></a>Двойное шифрование инфраструктуры для базы данных Azure для PostgreSQL

Узнайте, как использовать двойное шифрование инфраструктуры для базы данных Azure для PostgreSQL и управлять им.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure и права администратора для нее.

## <a name="create-an-azure-database-for-postgresql-server-with-infrastructure-double-encryption---portal"></a>Создание базы данных Azure для сервера PostgreSQL с помощью двойного шифрования инфраструктуры — портал

Выполните следующие действия, чтобы создать сервер базы данных Azure для PostgreSQL с двойным шифрованием инфраструктуры из портал Azure.

1. Выберите **Создать ресурс** (+) в левом верхнем углу страницы портала.

2. Выберите **Базы данных** > **База данных Azure для PostgreSQL**. Можно также ввести PostgreSQL в поле поиска, чтобы найти службу. Включен параметр развертывания с **одиночным сервером** .

   :::image type="content" source="./media/quickstart-create-database-portal/1-create-database.png" alt-text="База данных Azure для PostgreSQL в меню":::

3. Укажите основные сведения о сервере. Выберите **Дополнительные параметры** и включите флажок **двойное шифрование инфраструктуры** , чтобы задать параметр.

    :::image type="content" source="./media/howto-infrastructure-double-encryption/infrastructure-encryption-selected.png" alt-text="Параметры базы данных Azure для PostgreSQL":::

4. Щелкните **Просмотр и создание**, чтобы подготовить сервер.

    :::image type="content" source="./media/howto-infrastructure-double-encryption/infrastructure-encryption-summary.png" alt-text="Сводка по базе данных Azure для PostgreSQL":::

5. После создания сервера можно проверить двойное шифрование инфраструктуры, проверив состояние в колонке "сервер **шифрования данных** ".

    :::image type="content" source="./media/howto-infrastructure-double-encryption/infrastructure-encryption-validation.png" alt-text="Проверка базы данных Azure для MySQL":::

## <a name="create-an-azure-database-for-postgresql-server-with-infrastructure-double-encryption---cli"></a>Создание базы данных Azure для сервера PostgreSQL с помощью двойного шифрования инфраструктуры — CLI

Выполните следующие действия, чтобы создать сервер базы данных Azure для PostgreSQL с двойным шифрованием инфраструктуры из интерфейса командной строки:

В этом примере создается группа ресурсов с именем `myresourcegroup` в `westus` расположении.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```
В следующем примере создается сервер PostgreSQL 11 в западной части США под названием `mydemoserver` в группе ресурсов `myresourcegroup` с именем входа администратора сервера `myadmin` . Это сервер **4-го поколения** **общего назначения** с **двумя виртуальными ядрами**. При этом также будет включено двойное шифрование инфраструктуры для созданного сервера. Замените `<server_admin_password>` собственным значением.

```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen4_2 --version 11 --infrastructure-encryption >Enabled/Disabled>
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о шифровании данных см. в статье шифрование данных в [базе данных Azure для PostgreSQL](concepts-Infrastructure-double-encryption.md).

