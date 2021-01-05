---
author: amitkumarshukla
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: amishu
ms.openlocfilehash: b8dda0347e5713ef35705425b54f29a110803488
ms.sourcegitcommit: 28c93f364c51774e8fbde9afb5aa62f1299e649e
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/30/2020
ms.locfileid: "97821525"
---
Обработка сжатого аудио-сигнала реализуется с помощью [гстреамер](https://gstreamer.freedesktop.org). По соображениям лицензирования двоичные файлы Гстреамер не компилируются и не связываются с пакетом SDK для распознавания речи. Разработчикам необходимо установить несколько зависимостей и подключаемых модулей, см. статью [Установка в Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c) или [Установка в Linux](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html?gi-language=c). Двоичные файлы Гстреамер должны находиться в системном пути, чтобы пакет SDK для распознавания речи мог загружать двоичные файлы во время выполнения. Например, в Windows, если речевой пакет SDK может найти `libgstreamer-1.0-0.dll` во время выполнения, это означает, что двоичные файлы гстреамер находятся в системном пути.

