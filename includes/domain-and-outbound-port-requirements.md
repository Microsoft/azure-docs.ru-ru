---
title: Включить файл
description: включить файл
services: data-factory
author: nabhishek
ms.service: data-factory
ms.topic: include
ms.date: 10/09/2019
ms.author: abnarain
ms.openlocfilehash: 626074940ced57bdb1ae93f494b7a3847ef83a81
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96444997"
---
| Доменные имена                  | Исходящие порты | Описание                              |
| ----------------------------- | -------------- | ---------------------------------------- |
| `*.servicebus.windows.net`    | 443            | Требуется локальной среде выполнения интеграции для интерактивной разработки. |
| `{datafactory}.{region}.datafactory.azure.net`<br> или `*.frontend.clouddatahub.net` | 443            | Требуется локальной среде выполнения интеграции для подключения к службе фабрики данных. <br>Для нового экземпляра Фабрики данных найдите полное доменное имя из ключа локальной среды IR в формате {datafactory}.{region}.datafactory.azure.net. Для старого экземпляра Фабрики данных, если полное доменное имя не отображается в ключе локальной среды IR, используйте *.frontend.clouddatahub.net. |
| `download.microsoft.com`    | 443            | Требуется локальной среде выполнения интеграции для скачивания обновлений. Если автоматическое обновление отключено, можно пропустить настройку этого домена. |
| `*.core.windows.net`          | 443            | Используется локальной средой выполнения интеграции для подключения к учетной записи хранения Azure при помощи функции [промежуточного копирования](../articles/data-factory/copy-activity-performance.md#staged-copy). |
| `*.database.windows.net`      | 1433           | Требуется только при двунаправленном копировании с участием Базы данных SQL Azure или Azure Synapse Analytics. Используйте функцию промежуточного копирования для копирования данных в Базу данных SQL или Azure Synapse Analytics, не открывая порт 1433. |
| `*.azuredatalakestore.net`<br>`login.microsoftonline.com/<tenant>/oauth2/token`    | 443            | Требуется только при двунаправленном копировании с участием Azure Data Lake Store. |
