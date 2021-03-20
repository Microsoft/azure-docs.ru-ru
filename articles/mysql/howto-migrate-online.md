---
title: Миграция с минимальным временем простоя — база данных Azure для MySQL
description: В этой статье объясняется, как с помощью Azure Database Migration Service перенести базу данных MySQL в Базу данных Azure для MySQL с минимальным временем простоя.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/30/2020
ms.openlocfilehash: 75b548b53de4c0f4ea39959c3d70e9ddb361d64c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94540337"
---
# <a name="minimal-downtime-migration-to-azure-database-for-mysql"></a>Миграция в службу "База данных Azure для MySQL" с минимальным простоем
[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

Благодаря новой возможности **непрерывной синхронизации** в [Azure Database Migration Service](https://aka.ms/get-dms) (DMS) вы можете переносить базы данных MySQL в Базу данных Azure для MySQL с минимальным временем простоя. Эта функция позволяет сократить время простоя при работе приложения.

Дополнительные сведения и примеры использования миграции баз данных в базу данных Azure для MySQL см. в [руководстве по миграции баз данных](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide) . Это руководство содержит инструкции, которые позволят успешно спланировать и выполнить миграцию MySQL в Azure.

## <a name="overview"></a>Обзор
Azure Database Migration Service выполняет начальную загрузку локальных баз данных в Базу данных Azure для MySQL и непрерывно синхронизирует все новые транзакции с Azure во время работы приложения. Когда данные поступят в целевую службу Azure, остановите приложение на короткое время (минимальное время простоя), подождите, пока последний пакет данных (с момента остановки приложения до его фактической недоступности для принятия нового трафика) будет передан в целевую службу, а затем обновите строку подключения, чтобы она указывала на Azure. По завершении приложение станет доступно в Azure.

:::image type="content" source="./media/howto-migrate-online/ContinuousSync.png" alt-text="Непрерывная синхронизация с Azure Database Migration Service":::

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о переносе баз данных в службу "База данных Azure для MySQL" см. в [этой статье](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide).
- Просмотрите видео [Easily migrate MySQL/PostgreSQL apps to Azure managed service](https://medius.studios.ms/Embed/Video/THR2201?sid=THR2201) (Простой перенос приложений MySQL и PostgreSQL в управляемую службу Azure), в котором демонстрируется миграция приложений MySQL в Базу данных Azure для MySQL.
- Ознакомьтесь с руководством [Интерактивная миграция MySQL в Базу данных Azure для MySQL с помощью DMS](../dms/tutorial-mysql-azure-mysql-online.md).