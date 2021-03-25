---
title: Заметки о выпуске для средства создания моментальных снимков с помощью приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Заметки о выпуске для средства создания моментальных снимков с помощью приложений Azure, которые можно использовать с Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/22/2021
ms.author: phjensen
ms.openlocfilehash: 01fb93dcd0a1d5c1db615c47d7811a0baa863c9b
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105111478"
---
# <a name="release-notes-for-azure-application-consistent-snapshot-tool-preview"></a>Заметки о выпуске для средства создания моментальных снимков с помощью приложений Azure (Предварительная версия)

На этой странице перечислены значительные изменения, внесенные в Азакснап для предоставления новых функций или устранения дефектов.

## <a name="march-2021"></a>Март – 2021

### <a name="azacsnap-v50-preview-build2021031830771"></a>Азакснап версии 5.0 (сборка: 20210318.30771)

Версия Азакснап версии 5.0 (сборка: 20210318.30771) была выпущена со следующими исправлениями и улучшениями:

- Удалено нужно добавить пользователя АЗАКСНАП в SAP HANA клиент баз данных, см. раздел [Включение связи с SAP HANA](azacsnap-installation.md#enable-communication-with-sap-hana) .
- Исправлено, чтобы разрешить [Восстановление](azacsnap-cmd-ref-restore.md) с томами, настроенными с помощью QoS вручную.
- Добавлен элемент управления мьютексом для регулирования подключений SSH для крупных экземпляров Azure.
- Исправьте установщик для обработки имен путей с пробелами и другими связанными проблемами.
- При подготовке к поддержке других серверов баз данных изменен необязательный параметр "--ханасид" на "--дбсид".

Скачайте [последнюю](https://aka.ms/azacsnapdownload) версию установщика и посмотрите, как [приступить к работе](azacsnap-get-started.md).

## <a name="next-steps"></a>Следующие шаги

- [Приступая к работе со средством создания моментальных снимков, совместимых с приложениями Azure](azacsnap-get-started.md)
