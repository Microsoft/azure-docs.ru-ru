---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: trbye
ms.openlocfilehash: 8c77efe9e3e301573b032d1dc1dd32bb4a5ab1a1
ms.sourcegitcommit: df1930c9fa3d8f6592f812c42ec611043e817b3b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/13/2021
ms.locfileid: "103417731"
---
Обработка сжатого аудио-сигнала реализуется с помощью [гстреамер](https://gstreamer.freedesktop.org). По соображениям лицензирования двоичные файлы Гстреамер не компилируются и не связываются с пакетом SDK для распознавания речи. Разработчикам необходимо установить несколько зависимостей и подключаемых модулей, см. статью [Установка в Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c) или [Установка в Linux](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html?gi-language=c). Двоичные файлы Гстреамер должны находиться в системном пути, чтобы пакет SDK для распознавания речи мог загружать двоичные файлы во время выполнения. Например, в Windows, если пакет SDK для распознавания речи способен найти `libgstreamer-1.0-0.dll` или `gstreamer-1.0-0.dll` (для последних гстреамер) во время выполнения, это означает, что двоичные файлы гстреамер находятся в системном пути.

