---
title: Параметры для передачи данных в Azure с помощью устройства | Документация Майкрософт
description: Узнайте, как выбрать подходящее устройство для локальной пересылки данных в Azure между Data Box Edge, Синхронизация файлов Azure и сериями StorSimple 8000.
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: article
ms.date: 04/01/2019
ms.author: alkohli
ms.openlocfilehash: 4fc92146d6f076287fe37f64734bb07e8250792b
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/27/2021
ms.locfileid: "98882218"
---
# <a name="compare-storsimple-with-azure-file-sync-and-data-box-edge-data-transfer-options"></a>Сравнение StorSimple с Синхронизацией файлов Azure и параметрами передачи данных Data Box Edge 

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]
 
В этом документе представлены общие сведения о параметрах для локальной пересылки данных в Azure, а также сравнение: Data Box Edge и Синхронизация файлов Azure по сравнению с сериями StorSimple 8000.

- **[Data Box Edge](../databox-online/azure-stack-edge-overview.md)**. Data Box Edge — это локальное сетевое устройство, которое может перемещать данные на платформу Azure и с нее, а также добавлять пограничные вычисления с использованием ИИ для обработки данных во время передачи. Шлюз Data Box — виртуальная версия устройства с теми же возможностями передачи данных.
- **[Синхронизация файлов Azure](../storage/files/storage-sync-files-deployment-guide.md)**. Вы можете использовать службу синхронизации файлов Azure, чтобы централизованно хранить файловые ресурсы организации в службе файлов Azure, обеспечивая гибкость, производительность и совместимость локального файлового сервера. Это достигается путем преобразования Windows Server в быстрый кэш общего файлового ресурса Azure. Общедоступный выпуск службы Синхронизация файлов Azure был объявлен ранее в 2018 г.
- **[StorSimple](./storsimple-overview.md)**. StorSimple — это гибридное устройство, которое помогает предприятиям консолидировать их инфраструктуру хранилища для основного хранилища, защиты данных, архивации и аварийного восстановления для одного решения, тесно интегрируя с хранилищем Azure. Жизненный цикл продукта для StorSimple можно найти [здесь](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series).

## <a name="comparison-summary"></a>Сводка сравнения

|                           |StorSimple 8000   |Синхронизация файлов Azure   |Data Box Edge           |
|---------------------------|----------------------------------------|-------------------------------|-----------------------------------------|
|**Обзор**     |Многоуровневое гибридное хранение и архивирование|Хранилище общего файлового сервера с распределением по уровням в облаке и синхронизацией нескольких сайтов.  |Решение хранилища предварительно обработать данные и отправить их по сети в Azure.        |
|**Сценарии**    |Файловый сервер, архивирование, цель резервного копирования |Файловый сервер, архивирование (нескольких сайтов)   |Передача данных, данные предварительной обработки, включая выводы ML, Интернет вещей, архивирование    |
|**Пограничные вычисления** |Недоступно |Недоступно |Поддерживает выполнение контейнеров с помощью Azure IoT Edge    |
|**Форм-фактор**  |Физическое устройство   |Агент, установленный на Windows Server |Физическое устройство   |
|**Оборудование**     |Физическое устройство, предоставленное корпорацией Майкрософт как часть службы | Предоставляемый пользователем |Физическое устройство, предоставленное корпорацией Майкрософт как часть службы  |
|**Формат данных**  |Специальный формат   |Файлы         |Большие двоичные объекты или файлы    |
|**Поддержка протокола** |iSCSI          |SMB, NFS    | SMB или NFS      |
|**Цены**      |[StorSimple](https://azure.microsoft.com/pricing/details/storsimple/) |[Синхронизация файлов Azure](https://azure.microsoft.com/pricing/details/storage/files/)  |[Data Box Edge](https://azure.microsoft.com/pricing/details/storage/databox/edge/)  |

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте о [Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md) и [Шлюзе Azure Data Box](../databox-gateway/data-box-gateway-overview.md)
- Узнайте о [Синхронизации файлов Azure](../storage/files/storage-sync-files-deployment-guide.md)