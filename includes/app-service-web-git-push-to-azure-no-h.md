---
title: включить файл
description: включить файл
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 8539696f4521a1b4a2f56fe7d2936b45dec26ec9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102205993"
---
Вернитесь к окну терминала (в локальном расположении) и добавьте удаленное приложение Azure в локальный репозиторий Git. Замените *\<deploymentLocalGitUrl-from-create-step>* URL-адресом удаленного репозитория Git, который вы сохранили на этапе [создания веб-приложения](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Отправьте код в удаленное приложение Azure, чтобы развернуть приложение. При появлении запроса на ввод учетных данных в диспетчере учетных данных Git введите учетные данные, созданные на шаге **настройки пользователя развертывания** (а не те, которые используются для входа на портал Azure).

```bash
git push azure master
```

Выполнение этой команды может занять несколько минут. При выполнении эта команда выводит приблизительно следующие сведения:
