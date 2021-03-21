---
title: Перезапуск сервера-портал Azure — база данных Azure для MySQL
description: В этой статье описывается, как можно перезапустить сервер базы данных Azure для MySQL с помощью портал Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 2d19c0bd4ef5c49b8be82ffa11115ff6a1c6b302
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94541799"
---
# <a name="restart-azure-database-for-mysql-server-using-azure-portal"></a>Перезапуск Базы данных Azure для MySQL с помощью портала Azure
В этой статье объясняется, как перезапустить сервер в Базе данных Azure для MySQL. Возможно, вам потребуется перезапустить сервер в целях обслуживания, что приводит к кратковременному отключению во время выполнения операции.

Если служба занята, перезапустить сервер не удастся. Например, служба может обрабатывать запрошенную ранее операцию, такую как масштабирование виртуальных ядер.

Время, необходимое для завершения перезапуска, зависит от процесса восстановления MySQL. Чтобы уменьшить время перезапуска, рекомендуем свести к минимуму объем действий, выполняемых на сервере перед перезапуском.

## <a name="prerequisites"></a>Предварительные условия
Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:
- [Сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="perform-server-restart"></a>Перезапуск сервера

Чтобы перезапустить сервер MySQL, выполните следующие действия:

1. На портале Azure выберите нужный сервер базы данных Azure для MySQL.

2. На панели инструментов страницы **Обзор** сервера нажмите кнопку **Восстановить**.

   :::image type="content" source="./media/howto-restart-server-portal/2-server.png" alt-text="База данных Azure для MySQL: кнопка &quot;Перезапуск&quot; на странице &quot;Обзор&quot;":::

3. Нажмите кнопку **Да**, чтобы подтвердить перезапуск сервера.

   :::image type="content" source="./media/howto-restart-server-portal/3-restart-confirm.png" alt-text="База данных Azure для MySQL — подтверждение перезапуска":::

4. Убедитесь, что состояние сервера изменилось на "Идет перезапуск".

   :::image type="content" source="./media/howto-restart-server-portal/4-restarting-status.png" alt-text="База данных Azure для MySQL — состояние перезапуска":::

5. Убедитесь, что сервер перезапущен.

   :::image type="content" source="./media/howto-restart-server-portal/5-restart-success.png" alt-text="База данных Azure для MySQL — перезапуск выполнен":::

## <a name="next-steps"></a>Дальнейшие действия

[Создание сервера базы данных Azure для MySQL и управление им с помощью портала Azure](./quickstart-create-mysql-server-database-using-azure-portal.md)
