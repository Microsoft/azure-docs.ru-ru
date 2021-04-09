---
title: включить файл
description: включить файл
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 03/27/2019
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: df71f0804b62eb4b17ff8d2f652b076b5c64c959
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "92743805"
---
Можно получить доступ к журналам консоли, которые были созданы в контейнере.

Сначала включите ведение журнала контейнера, выполнив следующую команду:

```azurecli-interactive
az webapp log config --name <app-name> --resource-group <resource-group-name> --docker-container-logging filesystem
```

Замените `<app-name>` и `<resource-group-name>` именами, подходящими для вашего веб-приложения.

После включения ведения журнала контейнера, выполните следующую команду, чтобы просмотреть поток данных журнала.

```azurecli-interactive
az webapp log tail --name <app-name> --resource-group <resource-group-name>
```

Если журналы консоли не отображаются, проверьте еще раз через 30 секунд.

Чтобы остановить потоковую передачу журналов, нажмите клавиши **CTRL**+**C**.

Вы также можете проверить файлы журнала в браузере на странице `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.
