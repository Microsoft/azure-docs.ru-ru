---
title: 'Руководство. Создание серверной группы: Гипермасштабирование (Citus), База данных Azure для PostgreSQL'
description: Как создать группу серверов База данных Azure для PostgreSQL с поддержкой Гипермасштабирования (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 12/16/2020
ms.openlocfilehash: 4d4c03e62077599e054dea6efd0bc3eea107aac0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97618418"
---
# <a name="tutorial-create-server-group"></a>Руководство. Создание группы серверов

С помощью этого руководства вы создадите группу серверов в Базе данных Azure для PostgreSQL с поддержкой Гипермасштабирования (Citus). Вы выполните следующие действия:

> [!div class="checklist"]
> * подготовка узлов;
> * разрешение доступа по сети;
> * подключение к узлу-координатору.

[!INCLUDE [create server group](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь группа серверов подготовлена и можно приступать к следующему руководству:

* [Работа с распределенными данными](tutorial-hyperscale-shard.md)
