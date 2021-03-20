---
title: Отключение геоизбыточного резервного копирования
description: Руководство по отключению геоархивации для выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics
services: synapse-analytics
author: joannapea
manager: igorstan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 01/06/2021
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 395d5f0697138155b0bb0c629461aada9e9c18c6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98739148"
---
# <a name="disable-geo-backups-for-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Отключение геоархивации для выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics

Из этой статьи вы узнаете, как отключить геоархивацию для выделенного пула SQL (прежнее название — SQL DW) портал Azure.

## <a name="disable-geo-backups-through-azure-portal"></a>Отключение географического резервного копирования с помощью портал Azure

Выполните следующие действия, чтобы отключить геоархивацию для выделенного пула SQL (ранее — хранилища данных SQL):

> [!NOTE]
> Если вы отключаете геоархивацию, вы больше не сможете восстановить выделенный пул SQL (ранее — хранилище данных SQL) в другом регионе Azure. 
>

1. Войдите в учетную запись [портал Azure](https://portal.azure.com/) .
1. Выберите ресурс выделенного пула SQL (ранее — SQL DW), для которого вы хотите отключить геоархивацию. 
1. В разделе **Параметры** на панели навигации слева выберите **Политика географического резервного копирования**.

   ![Отключение геоархивации](./media/sql-data-warehouse-restore-from-geo-backup/disable-geo-backup-1.png)

1. Чтобы отключить геоархивацию, выберите **отключено**. 

   ![Отключенная геоархивация](./media/sql-data-warehouse-restore-from-geo-backup/disable-geo-backup-2.png)

1. Нажмите кнопку *сохранить* , чтобы убедиться, что параметры сохранены. 

   ![Сохранить параметры геоархивации](./media/sql-data-warehouse-restore-from-geo-backup/disable-geo-backup-3.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL)](sql-data-warehouse-restore-active-paused-dw.md)
- [Восстановление удаленного выделенного пула SQL (ранее — хранилища данных SQL)](sql-data-warehouse-restore-deleted-dw.md)
