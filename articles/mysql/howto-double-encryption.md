---
title: Двойное шифрование инфраструктуры — портал Azure — база данных Azure для MySQL
description: Узнайте, как настроить и управлять двойным шифрованием инфраструктуры для базы данных Azure для MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 06/30/2020
ms.openlocfilehash: 3f9c22a690859b459b6bb748c3b1001c4aa7660d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93241758"
---
# <a name="infrastructure-double-encryption-for-azure-database-for-mysql"></a>Двойное шифрование инфраструктуры для базы данных Azure для MySQL

Узнайте, как использовать двойное шифрование инфраструктуры для базы данных Azure для MySQL и управлять им.

## <a name="prerequisites"></a>Предварительные условия

* Подписка Azure и права администратора для нее.

## <a name="create-an-azure-database-for-mysql-server-with-infrastructure-double-encryption---portal"></a>Создание базы данных Azure для сервера MySQL с помощью двойного шифрования инфраструктуры — портал

Выполните следующие действия, чтобы создать сервер базы данных Azure для MySQL с двойным шифрованием инфраструктуры из портал Azure.

1. Выберите **Создать ресурс** (+) в левом верхнем углу страницы портала.

2. Выберите **Базы данных** > **База данных Azure для MySQL**. Чтобы найти службу, вы также можете ввести в поле поиска **MySQL**.

   :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/2_navigate-to-mysql.png" alt-text="Параметр базы данных Azure для MySQL":::

3. Укажите основные сведения о сервере. Выберите **Дополнительные параметры** и включите флажок **двойное шифрование инфраструктуры** , чтобы задать параметр.

    :::image type="content" source="./media/howto-double-encryption/infrastructure-encryption-selected.png" alt-text="Параметры базы данных Azure для MySQL":::

4. Щелкните **Просмотр и создание**, чтобы подготовить сервер.

    :::image type="content" source="./media/howto-double-encryption/infrastructure-encryption-summary.png" alt-text="Сводка по базе данных Azure для MySQL":::

5. После создания сервера можно проверить двойное шифрование инфраструктуры, проверив состояние в колонке "сервер **шифрования данных** ".

    :::image type="content" source="./media/howto-double-encryption/infrastructure-encryption-validation.png" alt-text="Проверка базы данных Azure для MySQL":::

## <a name="create-an-azure-database-for-mysql-server-with-infrastructure-double-encryption---cli"></a>Создание базы данных Azure для сервера MySQL с помощью двойного шифрования инфраструктуры — CLI

Выполните следующие действия, чтобы создать сервер базы данных Azure для MySQL с двойным шифрованием инфраструктуры из CLI:

В этом примере создается группа ресурсов с именем `myresourcegroup` в `westus` расположении.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```
В примере ниже показано создание сервера MySQL 5.7 с именем `mydemoserver` в группе ресурсов `myresourcegroup` с именем пользователя администратора сервера `myadmin`. Это сервер **4-го поколения** **общего назначения** с **двумя виртуальными ядрами**. При этом также будет включено двойное шифрование инфраструктуры для созданного сервера. Замените `<server_admin_password>` собственным значением.

```azurecli-interactive
az mysql server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 5.7 --infrastructure-encryption <Enabled/Disabled>
```

## <a name="next-steps"></a>Дальнейшие действия

 Дополнительные сведения о шифровании данных см. в статье [двойное шифрование инфраструктуры данных в базе данных Azure для MySQL](concepts-Infrastructure-double-encryption.md).
