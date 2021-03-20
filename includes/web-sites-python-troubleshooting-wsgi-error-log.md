---
title: включить файл
description: включить файл
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/11/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 020e59f029b09f3c7656f67039731e4141e68d31
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67185891"
---
Если при запуске приложения Python обнаруживает ошибку, будет возвращена только простая страница ошибки (например, "страница не может быть отображена из-за внутренней ошибки сервера").

Чтобы включить запись ошибок в приложении Python, сделайте следующее:

1. На портале Azure перейдите к своему веб-приложению и выберите **Параметры**.
2. На вкладке **Параметры** щелкните **Параметры приложения**.
3. В разделе **Параметры приложения** введите следующую пару "ключ — значение":
    * Ключ: WSGI_LOG.
    * Значение: D:\home\site\wwwroot\logs.txt (введите собственное имя файла).

Теперь ошибки должны отображаться в файле logs.txt в папке wwwroot.
