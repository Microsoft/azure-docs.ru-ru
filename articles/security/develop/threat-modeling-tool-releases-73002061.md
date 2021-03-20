---
title: Microsoft Threat Modeling Tool выпуск 02/11/2020 — Azure
description: Документирование заметок о выпуске для 7.3.00206.1 выпуска средства моделирования угроз.
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 02/25/2020
ms.openlocfilehash: 9f9b162460cd2e7a624c1ad3f992011487d1b795
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94516940"
---
# <a name="threat-modeling-tool-update-release-73002061---02112020"></a>Threat Modeling Tool обновления выпуска 7.3.00206.1 — 02/11/2020

Версия 7.3.00206.1 Microsoft Threat Modeling Tool (ТМТ) была выпущена 11 2020 февраля и содержит следующие изменения:

- Исправления ошибок

## <a name="notable-bug-fixes"></a>Исправления важных ошибок

### <a name="errors-related-to-priority-values-outside-of-the-expected-ranges"></a>Ошибки, связанные со значениями приоритета за пределами ожидаемых диапазонов

Некоторые клиенты сообщили о получении следующего сообщения об ошибке при открытии файлов, созданных в "Threat Modeling Tool 2016" или пользовательских шаблонах:

```output
System.InvalidOperationException: Invalid Priority value. Accepted values are [0..4] and 'High', 'Medium', 'Low' at ThreatModeling.Model.Threat.get_Priority()

System.ArgumentOutOfRangeException: Accepted values are 'High', 'Medium', and 'Low' Parameter name: value Actual value was 5.6. at ThreatModeling.Model.Threat.set_Priority(String value)
```

Эта проблема решена в этом выпуске.

## <a name="system-requirements"></a>Требования к системе

- Поддерживаемые операционные системы
  - [Юбилейное обновление Microsoft Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) или более поздняя версия
- Требуемая версия .NET
  - [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) или более поздней версии
- Дополнительные требования
  - Для получения обновлений средства и шаблонов требуется подключение к Интернету.

## <a name="documentation-and-feedback"></a>Документация и отзывы

- Документация по Threat Modeling Tool находится на сайте [docs.microsoft.com](./threat-modeling-tool.md) и содержит информацию [об использовании средства](./threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Дальнейшие действия

Скачайте последнюю версию [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).