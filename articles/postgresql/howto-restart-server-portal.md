---
title: Перезапуск сервера-портал Azure — база данных Azure для PostgreSQL — один сервер
description: В этой статье описывается, как можно перезапустить базу данных Azure для PostgreSQL-Single Server с помощью портал Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 12/20/2020
ms.openlocfilehash: faa61ff477f44347755890dc59ebf4b917afda6f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97882948"
---
# <a name="restart-azure-database-for-postgresql---single-server-using-the-azure-portal"></a>Перезапустите базу данных Azure для PostgreSQL — один сервер с помощью портал Azure
В этой статье объясняется, как перезапустить сервер в Базе данных Azure для PostgreSQL. Возможно, вам потребуется перезапустить сервер в целях обслуживания, что приводит к кратковременному отключению во время выполнения операции.

Если служба занята, перезапустить сервер не удастся. Например, служба может обрабатывать запрошенную ранее операцию, такую как масштабирование виртуальных ядер.
 
> [!NOTE] 
> Время, необходимое для завершения перезапуска, зависит от процесса восстановления PostgreSQL. Чтобы уменьшить время перезапуска, рекомендуем свести к минимуму объем действий, выполняемых на сервере перед перезапуском. Также может потребоваться увеличить частоту контрольных точек. Можно также настроить значения параметров, связанных с контрольной точкой `max_wal_size` , включая. Также рекомендуется выполнить `CHECKPOINT` команду перед перезапуском сервера.

## <a name="prerequisites"></a>Предварительные условия
Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:
- [Сервер базы данных Azure для PostgreSQL](quickstart-create-server-database-portal.md)

## <a name="perform-server-restart"></a>Перезапуск сервера

Чтобы перезапустить сервер PostgreSQL, выполните следующие действия:

1. В [портал Azure](https://portal.azure.com/)выберите сервер базы данных Azure для PostgreSQL.

2. На панели инструментов страницы **Обзор** сервера нажмите кнопку **Восстановить**.

   :::image type="content" source="./media/howto-restart-server-portal/2-server.png" alt-text="База данных Azure для PostgreSQL — &quot;Обзор&quot; — кнопка &quot;Восстановить&quot;":::

3. Нажмите кнопку **Да**, чтобы подтвердить перезапуск сервера.

   :::image type="content" source="./media/howto-restart-server-portal/3-restart-confirm.png" alt-text="База данных Azure для PostgreSQL — подтверждение восстановления":::

4. Убедитесь, что состояние сервера изменилось на "Идет перезапуск".

   :::image type="content" source="./media/howto-restart-server-portal/4-restarting-status.png" alt-text="База данных Azure для PostgreSQL — состояние перезапуска":::

5. Убедитесь, что сервер перезапущен.

   :::image type="content" source="./media/howto-restart-server-portal/5-restart-success.png" alt-text="База данных Azure для PostgreSQL — перезапуск выполнен":::

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [настройке параметров в базе данных Azure для PostgreSQL](howto-configure-server-parameters-using-portal.md)
