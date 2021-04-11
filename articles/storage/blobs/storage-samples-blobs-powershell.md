---
title: Примеры Azure PowerShell для хранилища BLOB-объектов Azure | Документы Майкрософт
description: См. таблицу со ссылками на примеры скриптов Azure PowerShell для работы с Хранилищем BLOB-объектов Azure, например для создания учетной записи хранения, переноса BLOB-объектов между учетными записями и много другого.
author: twooley
ms.author: twooley
ms.date: 11/07/2017
ms.service: storage
ms.subservice: blobs
ms.topic: sample
ms.openlocfilehash: 10e4149b7c66e507c34e486abd64c66993900fbc
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106277918"
---
# <a name="azure-powershell-samples-for-azure-blob-storage"></a>Примеры Azure PowerShell для хранилища BLOB-объектов Azure

Ниже приведена таблица с ссылками на примеры скриптов PowerShell, которые позволяют создавать хранилище Azure и управлять им.

| Скрипт | Описание |
|---|---|
|**Учетные записи хранения**||
| [Create a storage account and rotate its account access keys](../scripts/storage-common-rotate-account-keys-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) (Создание учетной записи хранения и получение или смена ее ключей доступа)| Создает учетную запись хранения Azure и получает и меняет один из ее ключей доступа. |
| [Перенос больших двоичных объектов между различными учетными записями хранения с помощью AzCopy в ОС Windows](/previous-versions/azure/storage/storage-common-transfer-between-storage-accounts?toc=%2fpowershell%2fmodule%2ftoc.json)| Перенос больших двоичных объектов между различными учетными записями хранения Azure с помощью AzCopy в ОС Windows. |
|**Хранилище BLOB-объектов**||
| [Calculate the size of a Blob storage container](../scripts/storage-blobs-container-calculate-size-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) (Вычисление размера контейнера хранилища BLOB-объектов) | Вычисление общего размера всех больших двоичных объектов в контейнере. |
| [Вычисление размера контейнера в хранилище BLOB-объектов для выставления счетов](../scripts/storage-blobs-container-calculate-billing-size-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Вычисление размера контейнера в хранилище BLOB-объектов для оценки затрат при выставлении счетов. |
| [Delete containers based on container name prefix](../scripts/storage-blobs-container-delete-by-prefix-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) (Удаление контейнеров на основе префикса имени контейнера) | Удаление контейнеров, начинающихся с указанной строки. |