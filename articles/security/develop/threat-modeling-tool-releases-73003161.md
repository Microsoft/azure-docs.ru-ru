---
title: Microsoft Threat Modeling Tool выпуск 03/22/2020 — Azure
description: Документирование заметок о выпуске для 7.3.00316.1 выпуска средства моделирования угроз.
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 03/22/2020
ms.openlocfilehash: e182d40d20529b5743fa9105c68108a8ae55d943
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94516906"
---
# <a name="threat-modeling-tool-update-release-73003161---03222020"></a>Threat Modeling Tool обновления выпуска 7.3.00316.1 — 03/22/2020

Версия 7.3.00316.1 Microsoft Threat Modeling Tool (ТМТ) была выпущена 22 2020 марта и содержит следующие изменения:

- Улучшение специальных возможностей
- Исправления ошибок
- Новая функция Диаграмреадер

## <a name="notable-bug-fixes"></a>Исправления важных ошибок

### <a name="exporting-the-threat-list-to-csv"></a>Экспорт списка угроз в CSV-файл

Функция экспорта в CSV была непоследовательно выбирать поля из списка угроз, которые будут экспортированы. Теперь все поля из списка угроз будут экспортированы в CSV-файл. 

### <a name="ux-bugs"></a>Ошибки UX

- Меню справки в основном рабочем процессе (создание, открытие и анализ) и интерфейс редактора шаблонов теперь имеют единообразные параметры меню.
- На панели "наборы элементов" теперь имеется стандартный курсор и добавлены соответствующие метки.

## <a name="new-features"></a>новые функции;

### <a name="diagramreader-feature-has-been-added"></a>Добавлена функция Диаграмреадер

В главном меню была добавлена новая функция Диаграмреадер, когда модель открыта. Эта функция преобразует графическое представление модели в текстовое сообщение. 

## <a name="system-requirements"></a>Требования к системе

- Поддерживаемые операционные системы:
  - [Юбилейное обновление Microsoft Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) или более поздняя версия
- Требуется версия .NET:
  - [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) или более поздней версии
- Дополнительные требования:
  - Подключение к Интернету для получения обновлений средства, а также шаблонов

## <a name="documentation-and-feedback"></a>Документация и отзывы

- Документация по Threat Modeling Tool находится на сайте [docs.microsoft.com](./threat-modeling-tool.md) и содержит информацию [об использовании средства](./threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Дальнейшие действия

Скачайте последнюю версию [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).