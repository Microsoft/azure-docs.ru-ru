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
ms.openlocfilehash: 08f4e76869f124d946566f64da394c895d0af308
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102205993"
---
Вернитесь к окну терминала (в локальном расположении) и добавьте удаленное приложение Azure в локальный репозиторий Git. Замените *\<deploymentLocalGitUrl-from-create-step>* URL-адресом удаленного репозитория Git, который вы сохранили на этапе [создания веб-приложения](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Отправьте код в удаленное приложение Azure, чтобы развернуть приложение. При появлении запроса на ввод учетных данных в диспетчере учетных данных Git введите учетные данные, созданные на шаге **настройки пользователя развертывания** (а не те, которые используются для входа на портал Azure).

```bash
git push azure main
```

Выполнение этой команды может занять несколько минут. При выполнении эта команда выводит приблизительно следующие сведения:
