---
title: Устранение неполадок импорта и экспорта в службе импорта и экспорта Azure | Документация Майкрософт
description: Узнайте, как решать распространенные проблемы при использовании службы импорта и экспорта Azure.
author: v-dalc
services: storage
ms.service: storage
ms.topic: troubleshooting
ms.date: 01/19/2021
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 555529b52d586078ba7e1832373ec126ba545c11
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98706685"
---
# <a name="troubleshoot-issues-in-azure-importexport"></a>Устранение неполадок при импорте и экспорте Azure
В этой статье описывается, как устранять распространенные проблемы при импорте и экспорте данных в службе импорта и экспорта Azure.

## <a name="a-copy-session-failed-what-i-should-do"></a>Сбой сеанса копирования. Что мне нужно сделать?  

При сбое сеанса копирования у вас есть два варианта:  
* Если ошибка может быть повторена, например, если сетевая папка была автономной в течение короткого периода времени и теперь находится в сети, вы можете возобновить сеанс копирования.
* Если ошибка не может быть повторена, например, если в параметрах командной строки указан неправильный каталог исходного файла, необходимо прервать сеанс копирования.
 
<!--For information about resuming and aborting copy sessions, see [Preparing Hard Drives for an Import Job](../storage-import-export-tool-preparing-hard-drives-import-v1.md  - Article we removed from TOC. File remains.-->

## <a name="i-cant-resume-or-abort-a-copy-session"></a>Не удается возобновить или прервать сеанс копирования.

Если это первый сеанс копирования для диска, то сообщение об ошибке должно выглядеть следующим образом: "The first copy session cannot be resumed or aborted" (Первый сеанс копирования нельзя возобновить или прервать). В этом случае можно удалить старый файл журнала и выполнить команду повторно.  

Если сеанс копирования не является первым для диска, сеанс всегда можно возобновить или прервать.  

## <a name="i-lost-the-journal-file-can-i-still-create-the-job"></a>Файл журнала потерян. Можно ли по-прежнему создать задание?

Файл журнала для диска содержит полные сведения о сеансе из копии данных. При добавлении файлов на диск файл журнала используется для создания задания импорта. Если вы потеряли файл журнала, потребуется повторить все сеансы копирования для диска.

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка средства импорта и экспорта Azure](storage-import-export-tool-setup-v1.md)
* [Подготовка жестких дисков для задания импорта](storage-import-export-data-to-blobs.md#step-1-prepare-the-drives)
* [Проверка состояния задания с помощью файлов журнала копирования](storage-import-export-tool-reviewing-job-status-v1.md)
* [Исправление задания импорта](storage-import-export-tool-repairing-an-import-job-v1.md)
* [Исправление задания экспорта](storage-import-export-tool-repairing-an-export-job-v1.md)