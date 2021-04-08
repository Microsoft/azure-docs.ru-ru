---
title: включить файл
description: включить файл
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 03/02/2020
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 6fc1f4152b2e16e1597c018e5af6e0245b075c3b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "78255833"
---
## <a name="robots933456-in-logs"></a>robots933456 в журналах

В журналах контейнера может отобразиться следующее сообщение:

```
2019-04-08T14:07:56.641002476Z "-" - - [08/Apr/2019:14:07:56 +0000] "GET /robots933456.txt HTTP/1.1" 404 415 "-" "-"
```

Это сообщение можно проигнорировать. `/robots933456.txt` — это фиктивный URL-путь, с помощью которого Служба приложений проверяет, может ли контейнер обслуживать запросы. Ответ 404 означает, что путь не существует. При этом он информирует Службу приложений о том, что контейнер работоспособен и готов отвечать на запросы.

