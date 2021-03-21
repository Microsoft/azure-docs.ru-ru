---
title: Запрет доступа к общедоступной сети — портал Azure — "база данных Azure для PostgreSQL — один сервер"
description: Узнайте, как настроить запрет доступа к общедоступной сети с помощью портал Azure для базы данных Azure для PostgreSQL одного сервера.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 03/10/2020
ms.openlocfilehash: e195c005676df27385e5e00736b04bdb689fafc5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101727113"
---
# <a name="deny-public-network-access-in-azure-database-for-postgresql-single-server-using-azure-portal"></a>Запрет доступа к общедоступной сети в базе данных Azure для PostgreSQL с помощью портал Azure

В этой статье описывается, как настроить односерверную базу данных Azure для PostgreSQL, чтобы запретить все общедоступные конфигурации и разрешить подключения только через частные конечные точки для дальнейшего повышения безопасности сети.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

* [База данных Azure для PostgreSQL с одним сервером](quickstart-create-server-database-portal.md) с общего назначения или ценовой категорией, оптимизированной для памяти.

## <a name="set-deny-public-network-access"></a>Установка запрета доступа к общедоступной сети

Выполните следующие действия, чтобы настроить PostgreSQL Single Server для запрета доступа к общедоступной сети.

1. В [портал Azure](https://portal.azure.com/)выберите существующую базу данных Azure для PostgreSQL Single Server.

1. На странице PostgreSQL Single Server в разделе **Параметры** щелкните **Безопасность подключения** , чтобы открыть страницу Настройка безопасности подключения.

1. В списке **запретить доступ к общедоступной сети** выберите **Да** , чтобы разрешить запретить общий доступ для PostgreSQL одного сервера.

    :::image type="content" source="./media/howto-deny-public-network-access/deny-public-network-access.PNG" alt-text="Сервер базы данных Azure для PostgreSQL запрещает доступ к сети":::

1. **Сохраните** изменения.

1. Уведомление подтверждает успешное включение параметра безопасности подключения.

    :::image type="content" source="./media/howto-deny-public-network-access/deny-public-network-access-success.png" alt-text="Служба &quot;база данных Azure для PostgreSQL&quot; запрещает доступ к сети":::

## <a name="next-steps"></a>Дальнейшие действия

Узнайте [, как создавать оповещения по метрикам](howto-alert-on-metric.md).
