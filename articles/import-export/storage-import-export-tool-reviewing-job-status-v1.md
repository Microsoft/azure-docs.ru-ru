---
title: Просмотр состояния задания инструмента импорта и экспорта Azure версии 1 | Документация Майкрософт
description: Узнайте, как использовать файлы журналов, созданные заданием импорта или экспорта, чтобы просмотреть состояние задания.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 01/19/2021
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: f634ceb60ae78d4d825c73bd2e98da2fb951b374
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98706701"
---
# <a name="reviewing-azure-importexport-job-status-with-copy-log-files"></a>Просмотр состояния задания импорта и экспорта Azure с помощью файлов журнала копирования
Когда служба импорт и экспорт Microsoft Azure обрабатывает диски, связанные с заданием импорта или экспорта, записывает файлы журнала копирования в учетную запись хранения, которая использовалась для импорта или экспорта больших двоичных объектов. Файл журнала содержит подробные сведения о состоянии каждого импортируемого или экспортируемого файла. Служба возвращает URL-адрес для каждого файла журнала копирования при запросе состояния завершенного задания. Дополнительные сведения см. в разделе [получение задания](/rest/api/storageimportexport/Jobs/Get).  

## <a name="example-urls"></a>Примеры URL-адресов

Ниже приведены примеры URL-адресов файлов журнала копирования для задания импорта с двумя дисками.  

 `http://myaccount.blob.core.windows.net/ImportExportStatesPath/waies/myjob_9WM35C2V_20130921-034307-902_error.xml`  

 `http://myaccount.blob.core.windows.net/ImportExportStatesPath/waies/myjob_9WM45A6Q_20130921-042122-021_error.xml`  

 Дополнительные сведения о формате журналов копирования и полный список кодов состояний см. в статье [Формат файла журнала службы импорта и экспорта Azure](/previous-versions/azure/storage/common/storage-import-export-file-format-log).  

## <a name="next-steps"></a>Дальнейшие действия

 * [Настройка средства импорта и экспорта Azure](storage-import-export-tool-setup-v1.md)   
 * [Подготовка жестких дисков для задания импорта](storage-import-export-data-to-blobs.md#step-1-prepare-the-drives)   
 * [Подготовка задания импорта](./storage-import-export-tool-repairing-an-import-job-v1.md)   
 * [Подготовка задания экспорта](./storage-import-export-tool-repairing-an-export-job-v1.md)