---
title: Запрет доступа к общедоступной сети — портал Azure — база данных Azure для MySQL.
description: Узнайте, как настроить запрет доступа к общедоступной сети с помощью портал Azure для базы данных Azure для MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 03/10/2020
ms.openlocfilehash: b6fd5b5f70eb813792be003836790752db1d071f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101732825"
---
# <a name="deny-public-network-access-in-azure-database-for-mysql-using-azure-portal"></a>Запрет доступа к общедоступной сети в базе данных Azure для MySQL с помощью портал Azure

В этой статье описывается, как настроить сервер базы данных Azure для MySQL для запрета всех общедоступных конфигураций и разрешить подключения только через частные конечные точки для дальнейшего повышения безопасности сети.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

* [База данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-portal.md) с общего назначения или ценовой категорией, оптимизированной для памяти

## <a name="set-deny-public-network-access"></a>Установка запрета доступа к общедоступной сети

Выполните следующие действия, чтобы настроить MySQL Server для запрета доступа к общедоступной сети.

1. В [портал Azure](https://portal.azure.com/)выберите существующий сервер базы данных Azure для MySQL.

1. На странице "сервер MySQL" в разделе " **Параметры**" щелкните **Безопасность подключения** , чтобы открыть страницу настройки безопасности подключения.

1. В списке **запретить доступ к общедоступной сети** выберите **Да** , чтобы включить запрет общего доступа для сервера MySQL.

    :::image type="content" source="./media/howto-deny-public-network-access/setting-deny-public-network-access.PNG" alt-text="Служба &quot;база данных Azure для MySQL&quot; запрещает доступ к сети":::

1. **Сохраните** изменения.

1. Уведомление подтверждает успешное включение параметра безопасности подключения.

    :::image type="content" source="./media/howto-deny-public-network-access/setting-deny-public-network-access-success.png" alt-text="Служба &quot;база данных Azure для MySQL&quot; запрещает доступ к сети":::

## <a name="next-steps"></a>Дальнейшие действия

Узнайте [, как создавать оповещения по метрикам](howto-alert-on-metric.md).
